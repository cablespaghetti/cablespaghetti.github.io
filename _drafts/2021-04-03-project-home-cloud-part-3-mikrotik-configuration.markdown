---
layout: post
title:  "Project Home Cloud Part 3: MikroTik hAP ac² Initial Configuration"
description: "Setting up MikroTik hAP ac² with the basic functionality to replace my ISP provided router."
date: 2021-04-04 12:00
tags: networking hardware mikrotik life
image: assets/thumbs/homecloudpart1-mikrotik.webp
---

In my last couple of posts I went over my plans to overhaul my home network and separate off the machines I have hosting web-facing services from the private network used by my family. In this post I'll outline the configuration of my new MikroTik router, as someone with a bit of networking knowledge but no previous experience with the brand.

## Initial setup

The hAP ac² (if you don't buy a used one) comes set up with an IP address of 192.168.88.1, DHCP enabled for this range and no password on the "admin" user. These devices have a lot of options, but to help you get started when you first access the "Webfig" web UI you will first be prompted to set a new password and then presented with the simplified "Quick set" interface.

For our purposes the "WISP AP" mode (selected in the top right corner) is the one we want. Here you can fill in the basic(ish) details for our Wi-Fi, Internet connection and LAN IP address. 

[![Quick Connect UI](/assets/thumbs/homecloudpart3-quickset.webp)](/assets/homecloudpart3-quickset.webp)

Unfortunately when I filled all this in, I hit a bug which made Webfig unavailable. [Other people seem to have hit it previously](https://forum.mikrotik.com/viewtopic.php?t=147568) but there didn't seem to be any misconfigured firewall rules in my case. In the end I logged in via SSH and ran `system reset-configuration` to take it back to factory defaults again.

The next time around I didn't hit the problem.  the "Local Network" portion in the Quick Set UI and left the rest to do the *hard way* later on.

## PPPoE configuration
 
https://support.aa.net.uk/Router_-_RouterOS_and_Routerboard#Native_IPv6_over_PPPoE


Do initial setup for IP and PPPoE. Disable DHCP.
Manually configure wifi
Manually configure port forwarding

