---
layout: post
title: "BGP Basics and Best Path Selection - part1"
date: 2014-12-17 21:26:20 +0200
comments: true
categories: cisco networking
---
{% img left /images/blog_posts/bgp.png %}

This is part 1 of 4. My shortish summarized version of Cisco BGP Basics and Best Path Selection.
<!--more-->
<br>
<br>
<br>
<br>
<br>

{% youtube z8INzy9E628 %}

###BGP Rules:

- Path Vector protocol that routes AS to AS (AS-Path).
- "You CAN'T tell someone else what to do whith their traffic!"
- Use TCP port 179 to establish neighbor relationships.
- Private AS range = 64512 - 65535
- By default ONLY advertise best path.
- Triggered updates ONLY (default 5 seconds INTERNAL and 30 seconds EXTERNAL).
- Neighbor relationships form/converge in approximately 30 - 60 seconds (SLOW)
- Default Hold timers: 180 seconds.
- Loopback routes get default wight of 32768.
- `network x.x.x.x` must be EXACT match as in routing table otherwise BGP will not advertise it.
- iBGP does NOT modify any attributes like AS-Path or Next-hop, therefore has rules like split-horizon to prevent loops. (`neighbor x.x.x.x next-hop-self`).
- iBGP, use **loopbacks** to form neighbor relationships.
- By default sends BGP messages to eBGP neighbors with a TTL of 1
- Supports up to 6 paths load-balanced.

###Best Path Algorithm:

1. MOST specific route
2. Administrative distance

|“We Love Oranges AS Oranges Mean Pure Refreshment”|
:-------|:----------------------------------------:
W 	    |Weight (Highest)
L 	    |LOCAL_PREF (Highest)
O 	    |Originate (local)
AS 	    |AS_PATH (shortest)
O 	    |ORIGIN Code (IGP > EGP > Incomplete)
M 	    |MED (lowest)
P 	    |Paths (External > Internal)
R 	    |RID (lowest)

###Basic Config:
{% codeblock %}
neighbor x.x.x.x remote-as <AS_NUM>
neighbor x.x.x.x update-source loopback0
neighbor x.x.x.x description <WHATEVER>
no synchronization
no auto-summary
{% endcodeblock %}

###Basic Show Commands:
{% codeblock %}
show ip bgp summary
show ip bgp
{% endcodeblock %}

**NOTES:**

If eBGP neighbor is NOT directly connected (iBGP does NOT use same rules):
`neighbor x.x.x.x ebgp-multihop <HOP_COUNT_1-255>`

Removing **private AS** number outbound:
`neighbor x.x.x.x remove-private-as`

Authentication: (MUST match both sides of neighbor relationship)

- `neighbor x.x.x.x password 0 cisco` (clear text)
- `neighbor x.x.x.x password 7 DFEAF10390E560AEA745CCBA53E044ED` (MD5 hashed)

Advertise default route outbound:
`neighbor x.x.x.x default-originate`