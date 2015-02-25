---
layout: post
title:  "Ooh look! Shiny new Wi-Fi!"
date:   2015-02-25 19:04
tags: hardware networking
---
The hotspots powering the guest and internal Wi-Fi networks at work were getting a bit old and senile, so around 6 months ago we decided to get some money to upgrade. At the time we only thought only one needed replacing, so we went out and bought a [Cisco WAP321][wap321] to replace it, having prior knowledge of their reputation and not wanting to be burdened with another unreliable hotspot.

Overall the setup was painless and the web interface working in any browser, unlike the Cisco SGE2010 switches we have which require Internet Explorer in Compatibility Mode. The only problem I had, was unfortunately quite a major one; by default the hotspot has "Spanning Tree Mode" enabled, an option buried in a deep dark sub-menu. As we use use STP on our 3 SGE2010 switches you would think that this wouldn't pose a problem. Sadly it put the switches into an STP loop and more or less brought down the entire LAN. Not very helpful and it persisted regardless of changes to switch port configuration changes until I turned off the option on the access point.

Since then the WAP321 has worked faultlessly and provided great performance. Although I wish it could operate in 2.4GHz and 5GHz mode simultaniously like its bigger brother the WAP371, which is perhaps more comparable to the Unifi hotspot.

(Why we got the unifi).

The Ubiquiti UniFi AP-AC impressed straight out of the box; unlike the Cisco it ships with a Gigabit POE adapter (instead of being £50 extra) it also comes with some very sturdy looking wall mounting brackets and all the screws you need. I even had comments when I was configuring it at my desk, just how smart it looked with the subtle blue light that illuminates when it is configured. Unlike the Cisco it doesn't have a web interface (although it does have an SSH interface), instead you need to install their very smart UniFi Controller application on a server. The download page only lists Mac and Windows versions so I put it on a Windows Server 2012 R2 VM I had, but it turns out there is a Linux version too which you can download from the [UniFi Updates Blog][unifi-blog], just make sure you've got a Java JRE installed and ports 8080 and 8443 aren't used.

(Problem with port initially).

(Further autodiscovery problems).

(Shiny web interface and great performance).

[wap321]: http://www.cisco.com/c/en/us/products/wireless/wap321-wireless-n-selectable-band-access-point-single-point-setup/index.html
[ap-ac]: https://www.ubnt.com/unifi/unifi-ap-ac/
[unifi-blog]: http://community.ubnt.com/t5/UniFi-Updates-Blog/bg-p/Blog_UniFi
