---
layout: post
title: "MPLS Layer 3 VPN VRFs and Redistribution - part2"
date: 2014-12-21 14:16:06 +0200
comments: true
categories: cisco networking
---
{% img left /images/blog_posts/mpls.png %}

This is part 2 of 2. My shortish summarized version of MPLS Layer 3 VPN VRFs and Redistribution.
<!--more-->
<br>
<br>
<br>
<br>

###Virtual Routing and Forwarding (VRF) Instances:

- **VRF Name:** AS:THECOMPANY
- **Route Distinguisher:** AS:NUMBER or IP_ADDRESS:NUMBER (Does NOT have to match PE-PE)
- **Route Target (extended community):** IP_ADDRESS:NUMBER or AS:NUMBER (Export and Import MUST match PE-PE)
- **Interface** towards CE to be allocated to VRF

NOTES:

In production, disable TTL propagation (on **ALL** MPLS routers) to hide label info inside traceroute! (`no mpls ip propagate-ttl`)

**Configuration (On PE routers):**

{% codeblock %}
vrf definition 100:THECOMPANY
  rd 100:1
  address-family ipv4
    route-target export x.x.x.x:100 (reversed on other PE router)
    route-target import y.y.y.y:100 (reversed on other PE router)
    exit
  exit

interface Gi0/1
  description THECOMPANY
  vrf forwarding 100:THECOMPANY (matches VRF definition)
  ip address x.x.x.x x.x.x.x (lives inside VRF)
  no shutdown
{% endcodeblock %}

**Verify:**

{% codeblock %}
show ip interface brief
show vrf
show ip route vrf 100:THECOMPANY
{% endcodeblock %}

###Routing Between CE and PE:

{% img center /images/blog_posts/mpls_vrf_forwarding_procedure.png %}

- Routing needs to be configured and associated to VRF (PE routers ONLY)
- CE routing needs to be configured (CE routers ONLY)
- Mutual redistribution IGP<-->mBGP needs to be configured (PE routers ONLY)

**OSPF (PE):**

{% codeblock %}
router ospf 100 vrf 100:YOURCOMPANY
  network x.x.x.x x.x.x.x area 0

show ip route vrf 100:YOURCOMPANY
{% endcodeblock %}

**OSPF (CE):**

{% codeblock %}
router ospf 1
  network x.x.x.x x.x.x.x area 0
{% endcodeblock %}

**EIGRP (PE):**

{% codeblock %}
router eigrp 1
  address-family ipv4 vrf 100:YOURCOMPANY autonomous-system 100
  network x.x.x.x

show ip route vrf 100:YOURCOMPANY
{% endcodeblock %}

**EIGRP (CE):**

{% codeblock %}
router eigrp 1
  network x.x.x.x
  no auto-summary
{% endcodeblock %}

**RIP (PE):**

{% codeblock %}
router rip
  version 2
  address-family ipv4 vrf 100:YOURCOMPANY
    network x.x.x.x
    no auto-summary

show ip route vrf 100:YOURCOMPANY
{% endcodeblock %}

**RIP (CE):**

{% codeblock %}
router rip
  version 2
  network x.x.x.x
  no auto-summary
{% endcodeblock %}

**BGP (PE):**

{% codeblock %}
router bgp 100
  address-family ipv4 vrf 100:YOURCOMPANY
    neighbor x.x.x.x remote-as 101 (eBGP)
    exit
  address-family ipv4
    neighbor x.x.x.x next-hop-self

clear ip bgp x.x.x.x
show ip route vrf 100:YOURCOMPANY
{% endcodeblock %}

**BGP (CE):**

{% codeblock %}
router bgp 101
  neighbor x.x.x.x remote-as 100 (eBGP)
  network x.x.x.x
{% endcodeblock %}

###Redistribution on PE routers:

- CE routes to be **imported** into VRF and **exported** to mBGP
- **VPNv4 labels** will be added to **Label Switched Path (LSP)** (Bottom Label).
- **Route distinguiser (RD)** and Route **Target (RT)** will be attached to routes.

**OSPF mutual redistribution:**

{% codeblock %}
router bgp 100
  address-family ipv4 vrf 100:YOURCOMPANY
    redistribute ospf 100 vrf 100:YOURCOMPANY
    exit
  exit

router ospf 100 vrf 100:YOURCOMPANY
  redistribute bgp 100 subnets
{% endcodeblock %}

**EIGRP mutual redistribution:**

{% codeblock %}
router bgp 100
  address-family ipv4 vrf 100:YOURCOMPANY
    redistribute eigrp 1
    exit
  exit

router eigrp 1
  address-family ipv4 vrf 100:YOURCOMPANY autonomous-system 100
    redistribute bgp 100 metric y y y y y
{% endcodeblock %}

**RIP mutual redistribution:**

{% codeblock %}
router bgp 100
  address-family ipv4 vrf 100:YOURCOMPANY
    redistribute rip
    exit
  exit

router rip
  address-family ipv4 vrf 100:YOURCOMPANY
    redistribute bgp 100 metric y (hop count)
{% endcodeblock %}

**Verify:**

{% codeblock %}
show bgp vpnv4 unicast rd 100:1 labels
show bgp vpnv4 unicast vrf 100:YOURCOMPANY label
show mpls forwarding-table vrf 100:YOURCOMPANY
show ip cef vrf 100:YOURCOMPANY x.x.x.x/xx
show bgp vpnv4 unicast vrf 100:YOURCOMPANY x.x.x.x/xx
show ip protocols
traceroute x.x.x.x
traceroute mpls ipv4 x.x.x.x x.x.x.x
{% endcodeblock %}

**Using "VRF-aware" applications for troubleshooting:**

{% codeblock %}
show ip route vrf 100:YOURCOMPANY
ip route vrf 100:YOURCOMPANY x.x.x.x y.y.y.y z.z.z.z
ping vrf 100:YOURCOMPANY x.x.x.x
traceroute vrf 100:YOURCOMPANY x.x.x.x
telnet x.x.x.x /vrf 100:YOURCOMPANY
{% endcodeblock %}
