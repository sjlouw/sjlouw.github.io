---
layout: post
title: "SNMP and Cisco routers or switches"
date: 2010-04-08
comments: true
categories: cisco
---
{% img left /images/blog_posts/cisco7.png %}

Setting up read-only SNMP community and sending SNMP traps to a central SNMP trap receiver, on Cisco devices. Some options like Spanning-tree, under the trap configuration, will not neccessary be available on all devices.
<!--more-->
<br>
<br>
<br>

**Configuring the SNMP community string and allow only the "Trap server" to connect "read-only" mode to it:**

{% codeblock %}
snmp-server community omsa_checker ro 70
access-list 70 permit <trap_server_ip> <trap_server_subnet>
snmp-server host <trap_server_ip> version 2c <some_comunity_string>
{% endcodeblock %}

**Specifying specific events that will trigger a message to be send to the "Trap server":**

{% codeblock %}
snmp-server enable traps envmon fan shutdown supply temperature status
snmp-server enable traps bridge newroot topologychange
snmp-server enable traps stpx inconsistency root-inconsistency loop-inconsistency
snmp-server enable traps vlan-membership
snmp-server enable traps errdisable
snmp-server host <trap_server_ip> version 2c <some_comunity_string> envmon bridge stpx vlan-membership errdisable
{% endcodeblock %}

**Verifying the SNMP configuration:**

{% codeblock %}
switch#show snmp host
{% endcodeblock %}

Having a "Trap server" with the above configuration, makes life much easier to monitor the network or be notified of events that will normally not be picked up with general monitoring software. I am specifically pointing here at Spanning-tree related events.
