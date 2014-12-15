---
layout: post
title: "ADSL setup and configuration on a Cisco 877 router"
date: 2014-01-20 17:16:40 +0200
comments: true
categories: cisco
---
{% img left /images/blog_posts/cisco13.png %}

Setup and configuration guide to configure a Cisco 877 router for ADSL connection.
<!--more-->
<br>
<br>
<br>
<br>

###Variables used in configuration:
* **DSL Username:** ISP_DSL_USERNAME
* **DSL Password:** ISP_DSL_PASSWORD

**Setup ATM interface:**
{% codeblock %}
interface ATM0
 no ip address
 no ip redirects
 no ip unreachables
 no ip proxy-arp
 ip flow ingress
 no atm ilmi-keepalive
 dsl bitswap both
{% endcodeblock %}

**Setup the virtual ATM interface:**
{% codeblock %}
interface ATM0.1 point-to-point
 no ip redirects
 no ip unreachables
 no ip proxy-arp
 ip flow ingress
 pvc 8/35
  pppoe-client dial-pool-number 1
{% endcodeblock %}

**Setup the local inside interface. I will be using the default, Vlan1, interface:**
{% codeblock %}
 interface Vlan1
 description FW_INSIDE
 ip address 172.16.1.1 255.255.255.0
 no ip redirects
 no ip unreachables
 no ip proxy-arp
 ip nat inside
 ip virtual-reassembly max-reassemblies 64
 ip tcp adjust-mss 1452
 no autostate
{% endcodeblock %}

**Setup the DSL dialer interface:**
{% codeblock %}
interface Dialer0
 description FW_OUTSIDE
 ip address negotiated
 no ip redirects
 no ip unreachables
 no ip proxy-arp
 ip mtu 1492
 ip nat outside
 ip virtual-reassembly max-reassemblies 64
 encapsulation ppp
 ip tcp adjust-mss 1452
 dialer pool 1
 dialer idle-timeout 0
 dialer persistent delay 10
 dialer-group 1
 no cdp enable
 ppp authentication pap chap callin
 ppp chap hostname ISP_DSL_USERNAME
 ppp chap password ISP_DSL_PASSWORD
 ppp pap sent-username ISP_DSL_USERNAME password ISP_DSL_PASSWORD
{% endcodeblock %}

**Create a default route for all traffic, outbound, to the Dialer0 interface:**
{% codeblock %}
ip route 0.0.0.0 0.0.0.0 Dialer0
{% endcodeblock %}

**Create an access-list to allow all traffic outbound from the inside subnet to outside any:**
{% codeblock %}
access-list 111 remark *****PERMIT local network Internet Access*****
access-list 111 permit ip 172.16.1.0 0.0.0.255 any
access-list 111 permit ip any any
{% endcodeblock %}

**Create a route-map for the inside to outside NAT and tell it to use access-list 111:**
{% codeblock %}
route-map NAT permit 10
 match ip address 111
{% endcodeblock %}

**Create the inside to outside NAT rule and apply it on the outside interface:**
{% codeblock %}
ip nat inside source route-map NAT interface Dialer0 overload
{% endcodeblock %}

###Display DSL connection statistics:
{% codeblock %}
show dsl interface atm 0
{% endcodeblock %}
