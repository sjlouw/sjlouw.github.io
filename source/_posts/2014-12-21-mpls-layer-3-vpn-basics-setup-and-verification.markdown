---
layout: post
title: "MPLS Layer 3 VPN Basics, Setup and Verification"
date: 2014-12-21 14:16:06 +0200
comments: true
categories: cisco networking
---
{% img left /images/blog_posts/mpls.png %}

My shortish summarized version of MPLS Layer 3 VPN Basics, Setup and Verification.
<!--more-->
<br>
<br>
<br>
<br>
<br>

{% youtube MEWIdO40U54 %}

###MPLS Layer 3 VPNs:

* Private routing (site-2-site) per customer (VRF)
* Virtual Network (private) VPN - Separates data-plane, securely, per customer.
* Terminology:
  * Label Switched Path (**LSP**)
  * Label Switch Router (**LSR**)
  * Customer Edge (**CE**)
  * Provider Edge (**PE**)
  * Provider (**P**)
* General Label flow procedure:
  * CE ------------> PE = **PUSH** label (entering core)
  * PE ------------> PE = **SWOP** labels (inside core)
  * PE ------------> CE = **POP** label (exiting core)
* MPLSs' control-plane protocol is Label Distribution Protocol **(LDP)**
* Experimental bits inside packet = Used for QoS
* Bottom-of-stack bits (S-bit) inside packet = If = 1 indicates bottom of stack has been reached
* MPLS header injected between Data-link (Layer2) and Network layer (Layer3)

###Basic Setup Procedure:

1. Before setting up MPLS, first verify basic IP routing/reachability (NLRI)
2. Setup MPLS only on PE to PE routers.
3. `mpls label range 100 199` (Setup label range for router to use)
4. `mpls ip` (Enable MPLS in Global Config mode)
5. `mpls ip` (Enable MPLS under interfaces to be used)

When MPLS is enabled on interfaces, **LDP** forms neighbors and exchange labels.

###Verifying Labels:

1. `show mpls ldp bindings x.x.x.x <SLASH_NOTATION_MASK>`
2. `show mpls forwarding-table x.x.x.x` (Forwarding table is build from **RIB**)

**NOTES:**

* Label, implicit Null (**imp-null**) (3) - Directly attached networks
* MPLS reserved labels = 0-15
* By default LAST **P** router POPs label before sending to LAST **PE** router. (Penultimate Hop Popping - PHP)
* If MPLS enabled : `traceroute` will include MPLS label
