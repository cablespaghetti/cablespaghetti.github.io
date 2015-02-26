---
layout: post
title:  "Ooh look! Shiny new Wi-Fi!"
date:   2015-02-25 19:04
tags: hardware networking
---
The hotspots powering the guest and internal Wi-Fi networks at work were getting a bit old and senile, so around 6 months ago we decided to get some money to upgrade. At the time we only thought only one needed replacing, so we went out and bought a [Cisco WAP321][wap321] to replace it, having prior knowledge of their reputation and not wanting to be burdened with another unreliable hotspot.

Overall the setup was painless and the web interface working in any browser, unlike the Cisco SGE2010 switches we have which require Internet Explorer in Compatibility Mode. The only problem I had, was unfortunately quite a major one; by default the hotspot has "Spanning Tree Mode" enabled, an option buried in a deep dark sub-menu. As we use use STP on our 3 SGE2010 switches you would think that this wouldn't pose a problem. Sadly it put the switches into an STP loop and more or less brought down the entire LAN. Not very helpful and it persisted regardless of changes to switch port configuration changes until I turned off the option on the access point. Wireless > WDS Bridge

Since then the WAP321 has worked faultlessly and provided great performance. Although I wish it could operate in 2.4GHz and 5GHz mode simultaniously like its bigger brother the WAP371, which is perhaps more comparable to the Unifi hotspot.

Packet capture, captive portal, syslog, rogue ap detection, qos, snmp

So, a few months later we've got problems with the Wi-Fi at the other end of the office and since buying the Cisco I've taken to reading [/r/sysadmin] on my OnePlus One before I drag myself out of bed in the morning. On this most excellent community I've seen numerous mentions of Ubiquiti's products and I don't think any of them have been negative, so after some further research I decided to recommend we got a [UniFi AP-AC][ap-ac] instead of another Cisco. Although the good reviews helped, the decision was mostly based on 2 things; firstly that £200 would get us another WAP321 and PoE injector from Cisco which is only single band and 802.11n, whereas the AP-AC is dual band and supports the new 802.11ac standard. The second major factor was that the WAP321 will only support 32 clients at a time [[source]][ciscodatasheet] (although I'm not sure what exactly happens when you hit that) which doesn't leave us much room for expansion, whereas the Unifi supports 200+ [[source]][unifidatasheet]. That's a lot of extra bang for our precious IT budget buck!

The Ubiquiti UniFi AP-AC impressed straight out of the box; unlike the Cisco it ships with a Gigabit POE adapter (instead of being £50 extra) it also comes with some very sturdy looking wall mounting brackets and all the screws you need. I even had comments when I was configuring it at my desk, just how smart it looked with the subtle blue light that illuminates when it is configured. Unlike the Cisco it doesn't have a web interface (although it does have an SSH interface), instead you need to install their very smart UniFi Controller application on a server. The download page only lists Mac and Windows versions so I put it on a Windows Server 2012 R2 VM I had, but it turns out there is a Linux version too which you can download from the [UniFi Updates Blog][unifi-blog], just make sure you've got a Java JRE installed and ports 8080 and 8443 aren't used.

The first machine I installed the Windows software on had something else listening on port 8080, but instead of bringing up an error the software just listened on localhost and of course the access point couldn't connect to it. So after working this out, I put the Controller software on a different machine and waited patiently for a few minutes but the access point still didn't pop up in the web interface. After a bit of Googling I found these instructions [on the Spiceworks Community][spiceworks] which solved the problem and got me to the point where I could start playing with the many configuration options. Yay!
Flash based
Captive portal settings, guest control settings syslog, snmp, bandwidth limiting for groups of users, rogue ap under insight tab

First impressions of the Unifi web interface were very positive; after logging in you are presented with an example office floorplan, invited to upload your own floorplan and drag your access points to their location, to plan out your coverage. However much this makes me want more access points to play with all the functionality properly, I've only got the one so I completely skipped this tab (for now at least) and headed straight for the Settings menu to set up 

[wap321]: http://www.cisco.com/c/en/us/products/wireless/wap321-wireless-n-selectable-band-access-point-single-point-setup/index.html
[ap-ac]: https://www.ubnt.com/unifi/unifi-ap-ac/
[unifi-blog]: http://community.ubnt.com/t5/UniFi-Updates-Blog/bg-p/Blog_UniFi
[/r/sysadmin]: http://www.reddit.com/r/sysadmin
[ciscodatasheet]: http://www.cisco.com/c/dam/en/us/products/collateral/wireless/wap321-wireless-n-selectable-band-access-point-single-point-setup/aag_c45-717569.pdf
[unifidatasheet]: http://dl.ubnt.com/datasheets/unifi/UniFi_AP_DS.pdf
[spiceworks]: http://community.spiceworks.com/how_to/9692-manually-setting-the-controller-address-for-a-unifi-ap
