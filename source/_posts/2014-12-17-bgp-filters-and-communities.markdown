---
layout: post
title: "BGP Filters and Communities"
date: 2014-12-17 21:25:55 +0200
comments: true
categories: cisco networking
---
{% img left /images/blog_posts/bgp.png %}

My shortish summarized version of Cisco BGP Filters and Communities.
<!--more-->
<br>
<br>
<br>
<br>
<br>

MATCH USING...     |STUFF TO SET...       |DIRECTION
:-------------------|:---------------------|:--------
prefix-list         |weight               |inbound
access-list         |local preference     |outbound
as-path access-list |origin               |
neighbor            |as-path (prepending) |
route-map           |med/metric           |
community-list      |next-hop (self)      |
distribute-list     |community            |
filter-list         |                     |
...and lots more    |...and lots more     |

###REGEX Filters:

Example: Filter AS 100 outbound with REGEX

{% codeblock %}
ip as-path access-list 1 deny <REGEX> (`_100_`) (Match AS 100 exactly)
ip as-path access-list 1 permit <REGEX> (`.*`) (Match all/permit any)

route-map FILTER_100 permit 10
  match as-path 1

router bgp <AS_NUM> (100)
  neighbor x.x.x.x route-map FILTER_100 out

clear ip bgp x.x.x.x
{% endcodeblock %}

###Prefix-lists:

Prefix-lists is for matching routes!

- Uses tree-structure
- Better CPU utilization
- Better subnet matching
- Two-stage matching system: **network + mask**
- Created in **Global Config** mode
- Matches x.x.x.x/xx EXACTLY
- **le** = less than or equel
- **ge** = greater than or equel

**Examples:**

1. `ip prefix-list NAME permit 0.0.0.0/0` matches ONLY 0.0.0.0/0 (default route)
2. `ip prefix-list NAME permit 0.0.0.0/0 ge 32` matches all HOST routes
3. `ip prefix-list NAME permit 0.0.0.0/0 le 32` matches ANY route
4. `ip prefix-list NAME permit 0.0.0.0/1 ge 24 le 24` matches CLASS A networks or it's subnets
5. `ip prefix-list NAME permit 128.0.0.0/2 ge 16` matches CLASS B networks or it's subnets

{% codeblock %}
ip prefix-list NAME permit 0.0.0.0/0 ge 32

router bgp 100
  neighbor x.x.x.x prefix-list NAME in

clear ip bgp x.x.x.x
{% endcodeblock %}

Prefix-lists can also be attached to route-maps.

###Outbound Route Filtering (ORF):

- Transmits INBOUND filters to apply OUTBOUND
- Neighbors must support **ORF Type** both sides
- `neighbor x.x.x.x capability orf prefix-list <SEND_RECEIVE_BOTH>`

###BGP Communities:

- The BGP community value is an optional, transitive attribute.
- ISP's normally create community values for customers, to use, to influence routes.
- RFC guideline format: 100:12345 where **100 = AS_NUM** and **12345 = community value**
- community-list **(1-99)** matches on normal community values
- community-list **(100-199)** matches on REGEX

**By default 4 well known communities that can be used to mark prefixes:**

- **Internet:** advertise these routes to all neighbors.
- **Local-as:** prevent sending routes outside the local As within the confederation.
- **No-Advertise:** do not advertise this route to any peer, internal or external.
- **No-Export:** do not advertise this route to external BGP peers.

**Examples:**

{% codeblock %}
ip bgp-community new-format
  neighbor x.x.x.x send-community

ip access-list standard CUSTOMER_ROUTES
  permit x.x.x.x x.x.x.x

route-map SET-COMMUNITY
  match ip address CUSTOMER_ROUTES
  set community 100:12345 100:84957 100:3892 (Multiple can be added on one line)
{% endcodeblock %}

{% codeblock %}
ip community-list 1 permit 100:2345

route-map MYMAP
  match community 1
  set local_preference 20 
{% endcodeblock %}