---
layout: post
title: "DHCP server on Cisco IOS"
date: 2010-04-08
comments: true
categories: cisco
---
{% img left /images/blog_posts/cisco8.png %}

Setting up a DHCP server to hand out IP addresses to clients, on a Cisco router or switch.
<!--more-->
<br>
<br>
<br>
<br>
<br>

{% codeblock %}
Router(config)#ip dhcp pool <any-dhcp-pool-name>
Router(dhcp-config)#network <my-network-ip> <my-network-subnetmask>
Router(dhcp-config)#dns-server <my-dns-server-ip>
Router(dhcp-config)#default-router <my-gateway-ip>
Router(dhcp-config)#domain-name <my-domain-name>
Router(dhcp-config)#lease DAYS HOURS MINUTES
Router(config)#ip dhcp excluded-address <ip-address-to-exclude-from-pool>
Router(config)#service dhcp
{% endcodeblock %}

Showing current leases:
{% codeblock %}
Router#show ip dhcp binding
{% endcodeblock %}

Show some DHCP server statistics:
{% codeblock %}
Router#show ip dhcp server statistics
{% endcodeblock %}

Should you choose, you can exclude a range of addresses from being assigned:
{% codeblock %}
Router(config)#ip dhcp excluded-address <start-ip-to-exclude> <end-ip-to-exclude>
{% endcodeblock %}

You can also make your ip address leases never expire:
{% codeblock %}
Router(dhcp-config)#lease infinite
{% endcodeblock %}
