---
layout: post
title: "MPLS Layer 3 VPN Basics, Setup and Verification - part1"
date: 2014-12-22 22:59:29 +0200
comments: true
categories: cisco networking
---
{% img left /images/blog_posts/mpls.png %}

This is part 1 of 2. My shortish summarized version of MPLS Layer 3 VPN Basics, Setup and Verification.
<!--more-->
<br>
<br>
<br>
<br>
<br>

{% youtube MEWIdO40U54 %}

###MPLS Layer 3 VPNs:

* Private routing (site-2-site) per customer (VRF)
* Virtual Network (private) VPN - Separates data-plane, securely (NO encryption), per customer.
* Terminology:
  * Label Switched Path (**LSP**)
  * Label Switch Router (**LSR**)
  * Customer Edge (**CE**)
  * Provider Edge (**PE**)
  * Provider (**P**)
  * Routing Information Base (**RIB**)
  * Forwarding Information Base (**FIB**)
  * Label Information Base (**LIB**)
  * Label Forwarding Information Base (**LFIB**)
* General label flow procedure:
  * CE ------------> PE = **PUSH** label (entering core)
  * PE ------------> PE = **SWOP** labels (inside core)
  * PE ------------> CE = **POP** label (exiting core)
* MPLSs' control-plane protocol is Label Distribution Protocol **(LDP)**
* Experimental bits inside packet = Used for QoS
* Bottom-of-stack bits (S-bit) inside packet - If = 1 indicates bottom of stack has been reached
* MPLS header injected between Data-link (Layer2) and Network layer (Layer3)

###Basic Setup Procedure:

1. Before setting up MPLS, first verify basic IP routing/reachability.
2. Setup MPLS only on PE to PE routers.
3. `mpls label range 100 199` (Setup label range for router to use)
4. `mpls ip` (Enable MPLS in Global Config mode)
5. `mpls ip` (Enable MPLS under interfaces to be used)

When MPLS is enabled globally, it creates local labels for each route in it's routing table. (default does **NOT** stay static, can change) Then when MPLS is enabled on interfaces, **LDP** forms neighbors and exchange labels.

###Verifying Labels:

1. `show mpls ldp bindings x.x.x.x <SLASH_NOTATION_MASK>`
2. `show mpls forwarding-table x.x.x.x` (Forwarding table is build from **RIB**)

**NOTES:**

* Label, implicit Null (**imp-null**) (3) - Directly attached networks
* MPLS reserved labels = 0-15
* By default LAST **P** router POPs label before sending to LAST **PE** router. (Penultimate Hop Popping - PHP)
* If MPLS enabled : `traceroute` will include MPLS label

###Label Distribution Protocol (LDP):

- Labels does **NOT** stay statically numbered by default. It **WILL** change when router reboots, config changes, etc.
- Best path will ALWAYS be LSP over IPv4
- IPv4 **transport address** (can be changed) used to form neighbors, NOT necessarily = Router-ID.
- Router MUST be able to reach neighbor's **transport address** to establish neighbor relationship.
- Use multicast **224.0.0.2**, destination **UDP port 646** in Hello packets.
- Use unicast, **TCP port 646** to establish neighbor relationship.
- Highest Router-ID will be **Active / initiating** router and lower Router-ID will be **Passive / receiving** router in TCP LDP neighbor negotiation.

Show labels assigned to routes:

`show mpls ldp binings`

Show local labels ONLY:

`show mpls ldp bindings local`

Show label best path for a route:

`show mpls forwarding-table x.x.x.x`

Debug LDP bindings:

`debug mpls ldp bindings`

**LDP Router-ID selection, order of preference:**

1. Administratively configured address (`mpls ldp router-id <INTERFACE/NUM> force (global config mode)`)
2. Highest IP address on loopback
3. Highest IP address on interface

**Transport address selection, order of preference:**

1. Administratively configured address (`mpls ldp discovery transport-address interface (under interface)`)
2. Router-ID

**Verifying both Router-ID and transport address:**

`show mpls ldp discovery detail`

###Label Switched Path (LSP):

**Label Retention:**

- **Liberal Label Retention (LLR) (default):** Label mapping received from peer is retained regardless of whether the LSR is the next hop for advertised mapping.
- **Conservative Label retention (CLR):** Label mappings will ONLY be retained if sending LSR is next hop downstream router for this specific FEC. ONLY labels used for MPLS packet forwarding are kept in LIB.

**Verification:**

{% codeblock %}
show mpls ldp neighbor
show mpls ldp bindings x.x.x.x
show mpls forwarding-table x.x.x.x
traceroute x.x.x.x
{% endcodeblock %}

###Multi-protocol BGP (mBGP):

- Configured on PE routers
- VPNv4 routes include VRF/Route distinguishing (RD) info
- **Route targets** included inside VPNv4 routes as extended community values, to be imported into VRFs on PE routers.

**Configuration:**

**1. Verify** PE to PE connectivity (`ping` / `traceroute`)

**2. Setup:** 
{% codeblock %}
router bgp 100
  neighbor x.x.x.x remote-as 100 (iBGP)
  neighbor x.x.x.x update-source loopback 0
  address-family vpnv4
    neighbor x.x.x.x activate
    neighbor x.x.x.x send-community extended
{% endcodeblock %}

**3. Verify:**
{% codeblock %}
show ip bgp neighbors | section capabilities
show control-plane host open-ports
{% endcodeblock %}