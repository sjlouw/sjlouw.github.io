---
layout: post
title: "Router on a stick Cisco router and a 3COM switch"
date: 2010-04-06
comments: true
categories: cisco
---
{% img left /images/blog_posts/cisco10.png %}

Using a Cisco router to do the routing and a 3COM 4500 Superstack III switch for the VLANs, 802.1Q trunk setup. I'm not going to use port 1 on the switch, as this belongs to the native VLAN1. Also remember that all the other ports on the switch, not assigned to their own VLANs, will also by default belong to VLAN1. In other words, all devices plugged into any of the VLAN1 ports, will be able to see each other by default!
<!--more-->

{% img center /images/blog_posts/routeronastick.png %}

##Setting up the Cisco router:

{% codeblock %}
interface fa0/1
description Trunk to 3COM Switch
no ip address
no shut

interface FastEthernet0/1.13
description Finance to switch port 13
encapsulation dot1Q 13
ip address 192.168.1.1 255.255.255.0

interface FastEthernet0/1.14
description Legal to switch port 14
encapsulation dot1Q 14
ip address 192.168.2.1 255.255.255.0

interface FastEthernet0/1.15
description HR to switch port 15
encapsulation dot1Q 15
ip address 192.168.3.1 255.255.255.0

interface FastEthernet0/1.16
description QC to switch port 16
encapsulation dot1Q 16
ip address 192.168.4.1 255.255.255.0

interface FastEthernet0/1.17
description Management to switch port 17
encapsulation dot1Q 17
ip address 192.168.5.1 255.255.255.0
{% endcodeblock %}

Save the changes!

##Setting up the 3COM 4500 Superstack III switch:

{% codeblock %}
Select menu option (bridge/vlan): create
Select VLAN ID (2-4094)[3]: 2
Enter VLAN Name [VLAN 2]: Trunk

Select menu option (bridge/vlan): create
Select VLAN ID (2-4094)[3]: 13
Enter VLAN Name [VLAN 13]: Finance

Select menu option (bridge/vlan): create
Select VLAN ID (2-4094)[3]: 14
Enter VLAN Name [VLAN 14]: Legal

Select menu option (bridge/vlan): create
Select VLAN ID (2-4094)[3]: 15
Enter VLAN Name [VLAN 15]: HR

Select menu option (bridge/vlan): create
Select VLAN ID (2-4094)[3]: 16
Enter VLAN Name [VLAN 16]: QC

Select menu option (bridge/vlan): create
Select VLAN ID (2-4094)[3]: 17
Enter VLAN Name [VLAN 17]: Management

Select menu option (bridge/vlan/modify): add
Select VLAN ID (1-2,116,120)[1]: 13
Select bridge ports (AL1-AL4,unit:port...,?): 1:13
Enter tag type (untagged, tagged): untagged

Select menu option (bridge/vlan/modify): add
Select VLAN ID (1-2,116,120)[1]: 14
Select bridge ports (AL1-AL4,unit:port...,?): 1:14
Enter tag type (untagged, tagged): untagged

Select menu option (bridge/vlan/modify): add
Select VLAN ID (1-2,116,120)[1]: 15
Select bridge ports (AL1-AL4,unit:port...,?): 1:15
Enter tag type (untagged, tagged): untagged

Select menu option (bridge/vlan/modify): add
Select VLAN ID (1-2,116,120)[1]: 16
Select bridge ports (AL1-AL4,unit:port...,?): 1:16
Enter tag type (untagged, tagged): untagged

Select menu option (bridge/vlan/modify): add
Select VLAN ID (1-2,116,120)[1]: 17
Select bridge ports (AL1-AL4,unit:port...,?): 1:17
Enter tag type (untagged, tagged): untagged

Select menu option (bridge/vlan/modify): add
Select VLAN ID (1-2,116,120)[1]: 13-17
Select bridge ports (AL1-AL4,unit:port...,?): 1:2
Enter tag type (untagged, tagged): tagged
{% endcodeblock %}

Save the changes!
