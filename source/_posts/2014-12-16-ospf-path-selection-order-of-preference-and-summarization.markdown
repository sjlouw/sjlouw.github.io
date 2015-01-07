---
layout: post
title: "OSPF Path Selection Order of Preference and Summarization - part1"
date: 2014-12-16 22:19:58 +0200
comments: true
categories: cisco networking
---
{% img left /images/blog_posts/ospf.png %}

This is part 1 of 2. My shortish summarized version of OSPF Path Selection Order of Preference and Summarization.
<!--more-->
<br>
<br>
<br>

###Path Selection:

These rules apply in THIS order even if the OSPF link metric (Cost value) is changed.

1. Intra Area Routes (O)
2. Inter Area Routes (O IA)
3. External Type 1 (E1)
4. External Type 2 (E2)
5. NSSA Type 1 (N1)
6. NSSA Type 2 (N2)

**Interface cost is derived from the bandwidth. The formula is:**

Cost = Reference / Bandwidth. (Rounded down to the closest integer)

By default, Reference = 100000 (Kb/s)

**NOTES:**
- Changing cost values, you need to use the `ip ospf cost <COST_VALUE>` command on the interface and NOT the `bandwidth` statement. The `bandwidth` command is also used for other traffic minipulation techniques like QoS and will break those.

On FastE, GigE, TenGigE and 100GigE the default OSPF cost will be "1", so you WILL want to change this!

###Summarization:

All routers inside the area MUST have exactly the same LSDB to be able to summarize!

**Summarization can ONLY be done:**

- between areas (ABR) with `area <SOURCE_AREA> range <ADDRESS> <MASK>`
- during redistribution (ASBR) from another protocol with `summary-address <ADDRESS> <MASK>`

It is possible to 'black hole' the traffic or sub-optimal route within the domain with OSPF summarization. OSPF will AUTO create a discard route with the **Null0** interface as the next-hop to prevent the router from using a shorter match (0.0.0.0), if the more specific destination network is not present in the routing table. This behavior can be disabled with `no discard-route <INTERNAL|EXTERNAL>`.
