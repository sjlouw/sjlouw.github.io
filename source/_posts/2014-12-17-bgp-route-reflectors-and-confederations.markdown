---
layout: post
title: "BGP Route Reflectors and Confederations - part3"
date: 2014-12-17 21:25:21 +0200
comments: true
categories: cisco networking
---
{% img left /images/blog_posts/bgp.png %}

This is part 3 of 4. My shortish summarized version of Cisco BGP Route Reflectors and Confederations.
<!--more-->
<br>
<br>
<br>
<br>

###Route Reflectors:

iBGP MUST have full mesh to prevent loops, thus to scale, you can use Route Reflectors.

1. Clients must have full mesh connection to Route-Reflectors.
2. Route Reflector groups add a cluster-id attribute to routes.

**Route Reflectors process updates as follows:**

- Routes learned from **eBGP** peers: Sent to all **iBGP**, **eBGP**, **Clients** and **Non-Client** peers.
- Routes learned from **iBGP (non-client)** peers: Sent to all **eBGP** and **Client** peers.
- Routes learned from **iBGP (client)** peers: Sent to **all** peers **except sender**.

Example Route Reflector Client Setup (No client-side setup needed):

{% codeblock %}
neighbor x.x.x.x route-reflector-client
{% endcodeblock %}

Example Route Reflector Cluster Setup:

{% codeblock %}
router bgp 100
  bgp cluster-id 10
{% endcodeblock %}

###Confederations:

Another way to solve the iBGP full mesh problem is to use Confederations.

- An AS inside an AS
- Uses **INTRA-AS** numbers which are stripped before sending updates via eBGP.
- Inter-Confederation peers are treated as eBGP to establish, but as iBGP relating to attributes.
- Public AS split into smaller Sub Autonomous Systems (Sub-ASs).
- Breaks iBGP AS into smaller Autonomous Systems.
- Typically use private AS numbers (64512 - 65535)
- Full iBGP mesh required **within confederation AS / Sub-AS** (RR also an option within confederation)

Example:

{% codeblock %}
router bgp 64512 (Private AS)
  bgp confederation identifier 500 (Public AS)
  bgp confederation peers 64513 64514 (eBGP peer ASs)
  neighbor x.x.x.x remote-as 64512 (will be iBGP)
  neighbor y.y.y.y remote-as 64513 (will be eBGP)
  neighbor y.y.y.y ebgp-multihop <HOP_COUNT>
  neighbor z.z.z.z remote-as 64514 (will be eBGP)
  neighbor z.z.z.z ebgp-multihop <HOP_COUNT>

show ip bgp summary
show ip bgp neighbors
{% endcodeblock %}