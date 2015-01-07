---
layout: post
title: "BGP Optimization, Peer Groups, Peer Templates, Route Dampening and Route Refresh - part4"
date: 2014-12-17 21:24:08 +0200
comments: true
categories: cisco networking
---
{% img left /images/blog_posts/bgp.png %}

This is part 4 of 4. My shortish summarized version of Cisco BGP Optimization, Peer Groups, Peer Templates, Route Dampening and Route Refresh.
<!--more-->
<br>
<br>
<br>

###Optimizations:

{% codeblock %}
ip tcp path-mtu-discovery (default is 536 bytes)
{% endcodeblock %}

{% codeblock %}
interface Gi0/0
hold-queue 1000 in (default is 75 packets)
{% endcodeblock %}

The BGP process scans the BGP table for changes for triggered updates.
{% codeblock %}
router bgp 100
  bgp scan-time <SECONDS> (default is 60 seconds)
  neighbor x.x.x.x advertisement-interval <SECONDS> (default is 30 seconds eBGP and 5 seconds iBGP)
{% endcodeblock %}

Setting a maximum prefix limit:
{% codeblock %}
neighbor x.x.x.x maximum-prefix <NUM>

show ip bgp summary (State/PfxRcd)
{% endcodeblock %}

###Peer Groups:

- Can be used to make neighbor configurations more effecient by using templates.
- CANNOT combine iBGP and eBGP peer-groups

Example:
{% codeblock %}
router bgp 100
  neighbor MY_PEERGROUP peer-group
  neighbor MY_PEERGROUP update-source loopback0
  neighbor ...............
  neighbor x.x.x.x peer-group MY_PEERGROUP

show ip bgp peer-group
{% endcodeblock %}

###Peer Templates:

- Generates single **outbound** update for all peers.
- Individual configurations supported for **inbound** updates.

Example: (peer-session)
{% codeblock %}
router bgp 100
  template peer-session MYTEMPLATE1
    timers 30 300
  exit-peer-session
  
  template peer-session MYTEMPLATE2
    remote-as 200
    inherit peer-session MYTEMPLATE1
  exit-peer-session

  neighbor x.x.x.x inherit peer-session MYTEMPLATE1

show ip bgp template peer-session
{% endcodeblock %}

Example: (peer-policy)
{% codeblock %}
router bgp 100
  template peer-policy MYTEMPLATE1
    prefix-list MYPREFIXLIST in
  exit-peer-policy

  neighbor x.x.x.x inherit peer-policy MYTEMPLATE1

show ip bgp template peer-policy
{% endcodeblock %}

###Route Dampening:

* Controls flapping routes
* Default 5 seconds **WAIT** for iBGP and 30 seconds **WAIT** for eBGP
* Flapping route will be dampened after 3 successive flaps and for 30 minutes default.
* Every route will receive the following by default:
  * Penalty = 1000
  * Suppress Limit = 750
  * Reuse limit = 750
* Decay algorithm = 15 minutes (Half-life)
* Maximum penalty = 4x decay algorithm = 60 minutes

Example:
{% codeblock %}
router bgp 100
  bgp dampening 10 1000 4000 255

clear ip bgp dampening x.x.x.x x.x.x.x (resets dampening)
clear ip bgp x.x.x.x flap-statistics (resets dampening)
{% endcodeblock %}

###Refreshing neighbor changes without tearing down the BGP session:

- Outbound soft reconfigure = `clear ip bgp x.x.x.x soft-out` (NO extra configuration needed)
- Inbound soft reconfigure = `clear ip bgp x.x.x.x soft-in` (MUST be pre-configured: `neighbor x.x.x.x soft-reconfiguration inbound`)
- Route refresh ONLY = `clear ip bgp x.x.x.x soft-in` (WITHOUT neighbor config as above)