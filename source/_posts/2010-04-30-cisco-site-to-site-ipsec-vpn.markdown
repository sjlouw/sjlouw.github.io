---
layout: post
title: "Cisco site-to-site IPSEC VPN"
date: 2010-04-30
comments: true
categories: cisco vpn
---
{% img left /images/blog_posts/cisco2.png %}

Setting up a "site-to-site" IPSEC VPN between two Cisco routers.

<!--more-->
<br>
<br>
<br>
<br>

###The following will be assumed:

* Router01 inside IP address = **192.168.0.1 255.255.255.0**
* Router01 outside IP address = **192.168.255.1 255.255.255.252**
* Router02 inside IP address = **192.168.1.1 255.255.255.0**
* Router02 outside IP address = **192.168.255.2 255.255.255.252**
* **mysecretkey** = can be anything, must be the same on both routers.
* **mytransform** = can be anything, must be the same on both routers.
* **mycryptomap** = can be anything, must be the same on both routers.
* **12345** = any number between 1-65535, sequence to insert into crypto map entry

###Router01 setup:

{% codeblock %}
Router01(config)#crypto isakmp policy 1
Router01(config-isakmp)#encryption 3des
Router01(config-isakmp)#authentication pre-share
Router01(config-isakmp)#group 2
Router01(config-isakmp)#exit
Router01(config)#crypto isakmp key mysecretkey address 192.168.255.2 no-xauth
Router01(config)#crypto ipsec transform-set mytransform esp-3des esp-sha-hmac
Router01(cfg-crypto-trans)#exit
Router01(config)#crypto map mycryptomap 12345 ipsec-isakmp
Router01(config-crypto-map)#set peer 192.168.255.2
Router01(config-crypto-map)#set transform-set mytransform
Router01(config-crypto-map)#match address ACL2Router02
Router01(config-crypto-map)#exit
Router01(config)#int s1/0
Router01(config-if)#crypto map mycryptomap
Router01(config-if)#no shut
Router01(config-if)#exit
Router01(config)#ip access-list extended ACL2Router02
Router01(config-ext-nacl)#20 permit ip 192.168.0.0 0.0.0.255 192.168.1.0 0.0.0.255
Router01(config-ext-nacl)#exit
Router01(config)#exit
{% endcodeblock %}

###Router02 setup:

{% codeblock %}
Router02(config)#crypto isakmp policy 1
Router02(config-isakmp)#encryption 3des
Router02(config-isakmp)#authentication pre-share
Router02(config-isakmp)#group 2
Router02(config-isakmp)#exit
Router02(config)#crypto isakmp key mysecretkey address 192.168.255.1 no-xauth
Router02(config)#crypto ipsec transform-set mytransform esp-3des esp-sha-hmac
Router02(cfg-crypto-trans)#exit
Router02(config)#crypto map mycryptomap 12345 ipsec-isakmp
Router02(config-crypto-map)#set peer 192.168.255.1
Router02(config-crypto-map)#set transform-set mytransform
Router02(config-crypto-map)#match address ACL2Router01
Router02(config-crypto-map)#exit
Router02(config)#int s1/0
Router02(config-if)#crypto map mycryptomap
Router02(config-if)#no shut
Router02(config-if)#exit
Router02(config)#ip access-list extended ACL2Router01
Router02(config-ext-nacl)#20 permit ip 192.168.1.0 0.0.0.255 192.168.0.0 0.0.0.255
Router02(config-ext-nacl)#exit
Router02(config)#exit
{% endcodeblock %}

###Verfify setup on both routers:

{% codeblock %}
Router#show crypto isakmp policy
Router#show crypto map
Router#show crypto ipsec transform-set
{% endcodeblock %}
