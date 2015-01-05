---
layout: post
title: "QoS (Quality of Service) - part1"
date: 2015-01-05 09:29:42 +0200
comments: true
categories: cisco networking
---
{% img left /images/blog_posts/qos.png %}

This is part 1 of 2. My shortish summarized version of Quality of Service (QoS), Basic concepts, Marking and Congestion Management.
<!--more-->
<br>
<br>
<br>

###Quality Of Service (QoS) Basics:

**Main Traffic-type Characteristics:**

* **Voice** traffic flow is smooth in nature, subject to packet drops, delay and has predictable bandwidth usage.
* **Video** traffic is bursty and has greedy traffic usage.
* **Data** traffic varies per application.

**Main Traffic Challenges:**

* Lack of bandwith
* Packet loss
* Delay
* Jitter (Variation in delay)

**Latency/Delay Types:**

* **Propagation delay:** time taken for head of signal to travel from sender to receiver.
* **Serialization delay:** time needed to place data on the wire/medium.
* **Processing delay:** time spend to take from input interface and move to output interface.
* **Packetization delay:** time taken to turn data into packets.
* **Queueing delay:** time spend in the queues of output interface.

###QoS Methods:

* **Classification** - identifying and groupiong different traffic types.
* **Marking** - tags/colors packets so it can be identified elsewhare in a network.
* **Policing** - drops or marks packets when specified limit is reached.
* **Shaping** - queues packets when a specified limit is reached.
* **Congestion Avoidence**
  * FIFO (First-in First-out) - Tail-drop
  * RED (Random Early Detection) - Drops packets randomly
  * WRED (Weighted Random Early Detection) - Drops packets with specified markings
* **Congestion Management** - Ordering packets in most efficient way
* **Link Efficiency**
  * Compression
  * Link Fragmentation and Interleaving (LFI)

###Modular QoS CLI (MQC):

1. Configure `class-map`
2. Configure `policy-map`
3. Apply `service-policy` (ONLY 1 per interface per direction)

<br>
{% img center /images/blog_posts/qos_mqc.png %}

###Classification:

- **Inspecting** one or more aspects of a packet to see what it's carrying.

**Network Based Application Recognition (NBAR):**

- Deep packet inspection to identify traffic
- OSI Layers 5 - 7
- Identification "signatures" (PDLM files / PDLM packs) can be extended (`ip nbar pdlm flash://whatever.pdlm`)

**Example:**

{% codeblock %}
class-map MYMAP
  match protocol <SOME_PROTOCOL_NAME>
{% endcodeblock %}

**NBAR Stats Collection:**

{% codeblock %}
interface Gi0/0
  ip nbar protocol-discovery
  load-interval 60

show ip nbar protocol-discovery stats bit-rate top-n 10
show ip nbar unclassified-port-stats (Unknown traffic)
{% endcodeblock %}

###Marking:

- **Writing** information to a packet to identify the classification decision.
- Mark packets as close to source (trust boundry) as possible.

**Types Of Marking:**

* **Layer 2** - Typically stripped at Layer 3 hops
  * example: COS, MPLS (Experimental bits), Frame Relay (DE bit), ATM (CLD bit)
* **Layer 3** - Passes through Layer 3 hops, excluding NAT
  * example: IP Precedence, DSCP

**Example:**

{% codeblock %}
class-map MATCH_HTTP
  match protocol http

policy-map MARKING_HTTP
  class MATCH_HTTP
    set ip precedence <PRECEDENCE_VALUE> (0 - 7)

interface Gi0/0
  service-policy <INPUT_OR_OUTPUT> MARKING_HTTP
{% endcodeblock %}

####IP Precedence and DSCP:

<br>
{% img center /images/blog_posts/ipp_dscp_compat.png %}

* **7** - Network - Reserved
* **6** - Internet - Reserved
* **5** - Critical - Voice bearer (RTP)
* **4** - Flash Override - Video
* **3** - Flash - Voice signaling (RTCP) or Video
* **2** - Immediate - High priority data
* **1** - Priority - Medium priority data
* **0** - Routine - Best effort data

{% img center /images/blog_posts/ipp_dscp_compat_chart.png %}

###Congestion Management:

* **FIFO (First-in First-out):**
  * Best effort
  * ONLY 1 queue (1 send and 1 receive)
  * method: FIFO
  * NO delay guarentee
  * NO bandwidth guarentee
* **Priority Queueing:**
  * Can cause bandwith starvation for "NOT high" trafffic.
  * 4 queues (High, Medium, Normal, Low)
  * method: strict priority
  * ONLY High priority delay guarentee
  * NO bandwidth guarentee
* **Custom Queueing:**
  * Assigns bytes per protocol per queue.
  * 16 queues
  * method: Round-robin
  * NO delay guarentee
  * Bandwidth GUARENTEE
* **Weighted Fair Queueing (WFQ):**
  * Cisco default for links less than 2048 bit/s
  * Number of queues per flow
  * method: Weighted Fair (least top talker gets priority)
  * NO delay guarentee
  * NO bandwidth guarentee
* **Class-based Weighted Fair Queueing (CBWFQ):**
  * Up to 256 classes (class-maps)
  * Use % of bandwidth per class-map
  * method: N/A
  * NO delay guarentee
  * Bandwidth GUARENTEE
* **Low Latency Queueing (LLQ):**
  * 1 Priority queue plus CBWFQ
  * method: Policed priority (Uses combination of other queueing methods)
  * Delay GUARENTEE for priority queue traffic
  * Bandwidth GUARENTEE

**Marking and LLQ Implementation:**

**NOTE:** Will ONLY see effect when there is congestion!

<br>
{% img center /images/blog_posts/congestion_management.png %}

**Router 2 Configuration:**

{% codeblock %}
class-map MATCH_HTTP
  match protocol http

class-map MATCH_FTP
  match protocol ftp

policy-map MARK_TRAFFIC
  class MATCH_HTTP
    set dscp af21
  class MATCH_FTP
    set dscp af11

interface Gi0/1
  service-policy output MARK_TRAFFIC

show class-map
show policy-map
show policy-map interface Gi0/1
{% endcodeblock %}

**Router 1 Configuration:**

{% codeblock %}
class-map MATCH_AF11
  match dscp af11

class-map MATCH_AF21
  match dscp af21

policy-map LLQ
  class MATCH_AF11
    bandwidth percent 15 (WFQ)
  class MATCH_AF21
    priority percent 15 (LLQ)
  class class-default
    fair-queue (CBWFQ)

interface Gi0/1
  service-policy output LLQ

show class-map  
show policy-map
show policy-map interface Gi0/1
{% endcodeblock %}
