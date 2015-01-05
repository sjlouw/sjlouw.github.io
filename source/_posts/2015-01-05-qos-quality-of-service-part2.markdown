---
layout: post
title: "QoS (Quality of Service) - part2"
date: 2015-01-05 09:14:26 +0200
comments: true
categories: cisco networking
---
{% img left /images/blog_posts/qos.png %}

This is part 2 of 2. My shortish summarized version of Quality of Service (QoS), Congestion Avoidence, Policing and Shaping.
<!--more-->
<br>
<br>
<br>

###Congestion Avoidence with WRED:

- ONLY TCP traffic
- **TCP Global Synchronization** happens to TCP flows during periods of congestion. Each sender will reduce their TX rate (at same time) - (Window-size to half) when packet loss occurs.
- WRED designed to avoid numerous and sudden packet drops that can cause TCP Global Synchronization.
- Randomly drops packets from TCP flows to minimize synchronization.
- Dropping becomes more aggressive as queues fill up.

{% img center /images/blog_posts/wred.png %}

**Example:**

{% codeblock %}
class-map MATCH_HTTP
  match protocol http

policy-map SET_HTTP
  class MATCH_HTTP
    random-detect dscp af11 <PACKETS_MIN> <PACKETS_MAX> <MPD_1-65535>
               or
    random-detect dscp-based (auto)

interface Gi0/0
  service-policy <INPUT_OR_OUTPUT> SET_HTTP

show class-map
show policy-map
show policy-map interface Gi0/0
{% endcodeblock %}

###ECN (Explicit Congestion Notification):

* WRED enhancement
* Sender and receiver MUST support ECN
* Instead of dropping packets, tells sender to slow down
* **2x ECN bits:**
  * **00** = NOT ECN capable
  * **01** = ECN capable
  * **10** = ECN capable
  * **11** = Congestion experienced
* When upstream router gets packet with **11** bit set, it returns **ECN-echo** packet to sender (application / host) and tells it to slow down.
* Can be enabled under policy-map with `random-detect ecn`.

###Compression:

- Payload Compression
- TCP Header Compression (suppress redundent header information)

{% codeblock %}
class-map MATCH_HTTP
  match protocol http

policy-map COMPRESS_HTTP
  class MATCH_HTTP
    compression header ip ?

interface Gi0/0
  service-policy <INPUT_OR_OUTPUT> COMPRESS_HTTP

show class-map
show policy-map
show policy-map interface Gi0/0
{% endcodeblock %}

###LFI (Link Fragmentation and Interleaving):

- Cause higher bandwidth usage due to packets being fragmented and every fragment needs it's own header.
- Used to reduce serialization delay
- NOT used on links faster than 768 Kbps
- ONLY used on Frame-relay and PPP

###Traffic Policing:

- **Drops** or **marks** traffic exceeding specified thresholds
- Use "Token Bucket" concept

**Single Token Bucket:**

{% img center /images/blog_posts/single_token_bucket.png %}

**Example:**

{% codeblock %}
class-map match-any MATCH_PROTOCOLS
  match protocol edonkey
  match protocol napster

policy-map POLICE_PROTOCOLS
  class MATCH_PROTOCOLS
    police <BPS_RATE> conform-action transmit exceed-action drop

interface Gi0/0
  service-policy input POLICE_PROTOCOLS

show class-map
show policy-map
show policy-map interface Gi0/0
{% endcodeblock %}

**Dual Token Bucket:**

{% img center /images/blog_posts/dual_token_buckets.png %}

**Example:**

{% codeblock %}
class-map match-any MATCH_PROTOCOLS
  match protocol edonkey
  match protocol napster

policy-map POLICE_PROTOCOLS
  class MATCH_PROTOCOLS
    police <BPS_RATE> <BURST_RATE> <PEAK_BURST> conform-action transmit exceed-action set-dscp-transmit <AF23> violate-action drop

interface Gi0/0
  service-policy input POLICE_PROTOCOLS

show class-map
show policy-map
show policy-map interface Gi0/0
{% endcodeblock %}

###Traffic Shaping:

- **Shapes** per class and NOT per interface
- **Queues** excess traffic and send at desired rate (class-based)
- Cannot mark traffic with shaping
- Can shape per average or peak rate
- BECN (Backwards Explicit Congestion Notification)
- FECN (Forward Explicit Congestion Notification)

{% img center /images/blog_posts/shaping.png %}

**Example Shaping:**

{% codeblock %}
class-map match-any MATCH_PROTOCOLS
  match protocol edonkey
  match protocol napster

policy-map SHAPE_PROTOCOLS
  class MATCH_PROTOCOLS
    shape peak <BPS> (CIR)
             or
    shape average <BPS> (CIR)

interface Gi0/0
  service-policy input SHAPE_PROTOCOLS

show class-map
show policy-map
show policy-map interface Gi0/0
{% endcodeblock %}

**Example Queueing (Nested policy and class-maps):**

{% codeblock %}
policy-map PRIORITY
  class DATA
    bandwidth 50
  class class-default
    fair-queue

class-map ALL_TRAFFIC
  match class DATA
  match class class-default

policy-map SHAPE_IT
  class ALL_TRAFFIC
    shape average 500000
    service-policy PRIORITY

interface Gi0/0
  service-policy input SHAPE_IT

show class-map
show policy-map
show policy-map interface Gi0/0
{% endcodeblock %}

###Catalyst QoS (Class Of Service (COS)):

* Layer 2
* Classification, marking, policing, etc. done in hardware ASICs (CEF)
* Switch QoS capabilities defined in terms of:
  * Queues (priority / standard)
  * Thresholds (ammount of packets possibly held in queue)
* **Queues can be configured as:**
  * Priority Queues
  * Weighted Round-robin (WRR) - Custom queue
  * Weighted Round-robin (WRR) - Priority queue
* **Thresholds can be configured with:**
  * Tail drop
  * WRED
* Also uses class-maps and policy-maps
* IP phones normaly tags voice packets - can be "caught" by switch and manipulated.
* Supports TX queues (RX queues ONLY on high end switches)

**Example switch QoS support tags:**

* **2Q2T** = 2x Queue, 2x Thresholds per Queue
* **1P2Q2T** = 1x Priority Queue, 2x Standard Queues, 2x Thresholds per Queue

**Example:**

{% codeblock %}
mls qos (turn it on)

interface Gi0/0
  mls qos trust (trust markings received on port)
  wrr-queue cos-map 1 0 1 2  ---|
  wrr-queue cos-map 2 3 4       |> first row, 1 - 4 = queue number
  wrr-queue cos-map 3 6 7       |> the rest, IP Precedence values 0 - 7
  wrr-queue cos-map 4 5      ---|
  wrr-queue bandwidth <WEIGHT_1-65536> 5 6 10 1
  priority-queue out
{% endcodeblock %}

- 5 = queue1, 6 = queue2, 10 = queue3, 1 = queue 4 (Last queue becomes priority queue)
- 5 + 6 + 10 + 1 = 22
- Queue1 = 5 / 22 * 100 = 22.73% of total bandwidth
- Queue2 = 6 / 22 * 100 = 27.27% of total bandwidth
- Queue3 = 10 / 22 * 100 = 45.45% of total bandwidth
- Queue4 = 1 / 22 * 100 = 4.55% of total bandwidth

###Auto-QoS:

- **cef** MUST be anabled (`ip cef`)
- **bandwidth** MUST be configured on interface (`bandwidth <BPS>`)
- **IP address** MUST be assigned

**Example:**

{% codeblock %}
interface Gi0/0
  bandwidth <BPS>
  auto qos <   > ?

show class-map
show policy-map
show ip access-list
show running-config
{% endcodeblock %}

###PAK Priority:

- Router auto-tags all "mission-critical" (control-plane) traffic with IPP6/CS6
- Is an un-changeable internal mechanism

###TX-Ring:

- Hardware queue of a router
- Automatically tuned by later Cisco IOS versions
- Typically holds 32 - 64 packets by default
- Reduce to 3 on slow, smaller then 1.5Mbit/s links
- `show controllers <INTERFACE> | incl tx_limited`
