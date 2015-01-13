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

###Open Shortest Path First (OSPF) Basics:

* Use protocol number **89**
* Use bandwidth-based cost metric
* Use highest IP on interfaces in UP/UP state as **Router-ID** by default
* **Broadcast Networks:**
  * **224.0.0.5** - ALL OSPF Routers - Used for **Hello** packets. Used by DR/BDR for LS-Updates and LS-Acks
  * **224.0.0.6** - ALL DR/BDR Routers - Used by ALL routers except DR/BDR to send LS-Updates and LS-Acks to DR/BDR
* **Point-to-Point Networks:**
  * **224.0.0.5** - Used by ALL for ALL OSPF "messages"
* **Non-Broadcast Networks:**
  * **NO** multicast - Destination IP of Hello / Link State packets is unicast IP of a specific neighbor.
  * **Neighbor IP** is a required part of OSPF configuration for Non-Broadcast links.
* OSPFv2 Supports **cleartext** and **MD5** authentication
* Supports MPLS-TE via "Opaque" LSA's (9-link-local, 10-area-local, 11-AS)
* If equal cost routes exist, uses CEF load balancing.

**Neighbors MUST agree on following to become adjacent:**
  
* Area Number
* Authentication
* Hello and Dead intervals
* Stub area flag
* MTU
* Neighbors MUST use compatible network types
* MUST be on same subnet. ONLY on Point-to-Point links rule does not apply. (`ip unnumbered`)

###Path Selection:

These rules apply in THIS order even if the OSPF link metric (Cost value) is changed.

1. Intra Area Routes (O)
2. Inter Area Routes (O IA)
3. External Type 1 (E1)
4. External Type 2 (E2)
5. NSSA Type 1 (N1)
6. NSSA Type 2 (N2)

**Interface cost is derived from the bandwidth. The formula is:**

By default, Reference = 100000 (Kb/s) (100 Mbps)

Cost = Reference bandwidth / Interface bandwidth. (Rounded down to the closest integer)

Can be modified: `ospf auto-cost reference-bandwidth`

**NOTES:**
- Changing cost values, you need to use the `ip ospf cost <COST_VALUE>` command on the interface and NOT the `bandwidth` statement. The `bandwidth` command is also used for other traffic minipulation techniques like QoS and will break those.

On FastE, GigE, TenGigE and 100GigE the default OSPF cost will be "1", so you WILL want to change this!

###Summarization:

All routers inside the area MUST have exactly the same LSDB to be able to summarize!

**Summarization can ONLY be done:**

- between areas (ABR) with `area <SOURCE_AREA> range <ADDRESS> <MASK>`
- during redistribution (ASBR) from another protocol with `summary-address <ADDRESS> <MASK>`

It is possible to 'black hole' the traffic or sub-optimal route within the domain with OSPF summarization. OSPF will AUTO create a discard route with the **Null0** interface as the next-hop to prevent the router from using a shorter match (0.0.0.0), if the more specific destination network is not present in the routing table. This behavior can be disabled with `no discard-route <INTERNAL|EXTERNAL>`.
