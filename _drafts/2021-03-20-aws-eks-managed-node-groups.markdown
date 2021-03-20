---
layout: post
title:  "Amazon EKS Managed Node Groups, the good, the bad and the config"
description: "The pros and cons, and how to migrate to Amazon EKS Managed Node Groups."
date: 2021-03-20 17:00
tags: kubernetes aws eks
---

Amazon EKS launched in 2018 to the relief of many who had been managing their own Kubernetes clusters on AWS. However it wasn't as fully featured as some had hoped out of the gate. One of the big improvments Amazon made was to release (Managed Node Groups)[https://aws.amazon.com/blogs/containers/eks-managed-node-groups/) in 2019. This removed the need for people to manage their own Auto Scaling Groups for their Kubernetes nodes on EKS and tasks like replacing nodes to upgrade to a new AMI release no longer required a long drawn out manual process or [home grown automation](https://github.com/hellofresh/eks-rolling-update).

Unfortunately the initial release of Managed Node Groups had some limitations which meant it wasn't suitable for everyone (including us). Most importantly for us, it only supported On Demand instances, but users could also not customise the Launch Template used for the nodes so were restricted to using the official EKS Optimized AMI and could only customise node labels rather than having full control over the [bootstrap.sh script](https://aws.amazon.com/blogs/opensource/improvements-eks-worker-node-provisioning/). 

The features which ultimately made them viable for us were [launch template support](https://aws.amazon.com/blogs/containers/introducing-launch-template-and-custom-ami-support-in-amazon-eks-managed-node-groups/) and [spot instance support](https://aws.amazon.com/blogs/containers/amazon-eks-now-supports-provisioning-and-managing-ec2-spot-instances-in-managed-node-groups/) which came out late last year, [although I only found out a couple of weeks ago](https://twitter.com/rothgar/status/1368457026175602693).

You can read a little more about how we've got things set up in my [previous post about spot instances](https://cablespaghetti.dev/2021/03/05/aws-spot-instances-in-production/). However I'll go into more detail on the Managed Node Group specifics in this post. The short version is that we obviously needed Spot support and to customise the taints on our [ARM nodes](https://cablespaghetti.dev/2021/02/20/managing-multi-arch-kubernetes-clusters/), so had to wait until now.

## The Good

The primary reason we desperately wanted to move to Managed Node Groups is the amount of time and effort it currently takes us to upgrade to a new AMI, either to get a security fix or upgrade to a new Kubernetes version. We've been using [hellofresh/eks-rolling-update](https://github.com/hellofresh/eks-rolling-update) for this which is a great tool, but needs to be run manually. This isn't too much of a problem with one or two clusters but for fifteen it gets *very* time consuming.

There are other advantages, such as the reduction in complexity in adding nodes with a standard configuration to an EKS cluster which has been a bit of a barrier to entry until this point. However this wasn't a big factor for us, as we've customised things to much anyway in our Terraform config.

## The Bad

For our specific use case where we have multiple groups of nodes with different priorities; so our groups with Spot Instances scale up most of the time and On Demand instances only get launched if those groups are unable to launch capacity, the naming of the Auto Scaling Groups which Managed Node Groups spin up under the hood was a problem. This is because the Kubernetes Cluster Autoscaler doesn't see the nice descriptive name you give your Node Group, it only sees the UUID style hash given to the ASG. I had to [build a tool](https://github.com/cablespaghetti/priority-expander-eks-managed-nodegroup-configurer) to generate Priority Expander configuration for the autoscaler to work around this. However if you're not using the Priority Expander it's not going to be an issue.

## The Config

We found launching managed node groups with our own custom Launch Template to be a little *nuanced*. For example we had some cryptic error messages when trying to use a custom user-data script without explicitly setting the AMI. Here's the Terraform we used in case it helps others who need to tweak things in more complex ways:

`user-data.sh`:
```sh
#!/bin/bash -xe
# Inject imageGCHighThresholdPercent value unless it has already been set.
if ! grep -q imageGCHighThresholdPercent /etc/kubernetes/kubelet/kubelet-config.json;
then
    sed -i '/"apiVersion*/a \ \ "imageGCHighThresholdPercent": 70,' /etc/kubernetes/kubelet/kubelet-config.json
fi

# Inject imageGCLowThresholdPercent value unless it has already been set.
if ! grep -q imageGCLowThresholdPercent /etc/kubernetes/kubelet/kubelet-config.json;
then
    sed -i '/"imageGCHigh*/a \ \ "imageGCLowThresholdPercent": 50,' /etc/kubernetes/kubelet/kubelet-config.json
fi

/etc/eks/bootstrap.sh --b64-cluster-ca ${cluster_auth_base64} \
--apiserver-endpoint ${endpoint} \
--kubelet-extra-args '--node-labels=lifecycle="${node_lifecycle}" --register-with-taints="${node_taint}"' \
${cluster_name}
```

We then configure the `template_file` for each set of Node Groups:
```terraform
data "template_file" "userdata_spot_arm64" {
  template = file("${path.module}/user-data.sh")
  vars = {
    cluster_name        = var.cluster-name
    endpoint            = aws_eks_cluster.eks-cluster.endpoint
    cluster_auth_base64 = aws_eks_cluster.eks-cluster.certificate_authority[0].data
    node_lifecycle      = "spot"
    node_taint          = "arch=arm64:NoSchedule"
  }
}
```

We also need to get the AMI from Amazon's public Systems Manager paths:
```
data "aws_ssm_parameter" "eks-worker-ami" {
  name = "/aws/service/eks/optimized-ami/1.19/amazon-linux-2/recommended/image_id"
}

data "aws_ssm_parameter" "eks-worker-ami-arm64" {
  name = "/aws/service/eks/optimized-ami/1.19/amazon-linux-2-arm64/recommended/image_id"
}
```

The there's the `launch_template`:
```terraform
resource "aws_launch_template" "eks-cluster-node-group-worker-nodes-spot-arm64" {
  image_id               = data.aws_ssm_parameter.eks-worker-ami-arm64.value
  name                   = "${var.cluster-name}-eks-cluster-node-group-worker-nodes-spot-arm64"
  vpc_security_group_ids = [aws_security_group.eks-cluster-worker-nodes.id]
  user_data              = base64encode(data.template_file.userdata_spot_arm64.rendered)
  ebs_optimized          = true

  block_device_mappings {
    device_name = "/dev/xvda"
    ebs {
      volume_size = 50
      volume_type = "gp3"
      iops        = 3000
    }
  }
  lifecycle {
    create_before_destroy = true
  }
  depends_on = [aws_eks_cluster.eks-cluster]
}
```

Finally the `node_group`:
```terraform
resource "aws_eks_node_group" "eks-worker-nodes-spot-arm64" {
  count           = length(local.subnet-ids)
  cluster_name    = aws_eks_cluster.eks-cluster.name
  capacity_type   = "SPOT"
  node_group_name = "${var.cluster-name}-${count.index}-spot-arm64"
  node_role_arn   = aws_iam_role.eks-cluster-worker-nodes.arn
  subnet_ids      = [element(local.subnet-ids, count.index)]
  instance_types  = ["m6g.2xlarge"]

  scaling_config {
    desired_size = var.spot-arm64-min-hosts-per-az
    max_size     = var.spot-arm64-max-hosts-per-az
    min_size     = var.spot-arm64-min-hosts-per-az
  }

  lifecycle {
    ignore_changes = [scaling_config.0.desired_size]
  }

  launch_template {
    id      = aws_launch_template.eks-cluster-node-group-worker-nodes-spot-arm64.id
    version = aws_launch_template.eks-cluster-node-group-worker-nodes-spot-arm64.latest_version
  }

  tags = {
    Name        = "${var.cluster-name}-${count.index}-spot-arm64"
    Environment = var.cluster-name
  }
}
```

## The Migration Steps

* Upgrade the EKS Cluster
* Upgrade kube-proxy, cluster autoscaler etc
* Drain the nodes - kubectl drain --selector '!eks.amazonaws.com/nodegroup' --delete-local-data --ignore-daemonsets
* Delete the old ASGs


'm always happy to chat to people working on similar challenges, so get in touch on [Twitter](https://twitter.com/cablespaghetti) if you have any questions or just fancy a chat!
