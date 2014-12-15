---
layout: post
title: "Cisco PPP with authentication"
date: 2010-04-29
comments: true
categories: cisco
---
{% img left /images/blog_posts/cisco4.png %}

Cisco use HDLC encapsulation by default for point-to-point serial links. This is how to setup PPP encapsulation with authentication, but without encryption.

Encapsulation must be the same on both routers. <!--more-->Note how the hostname of the reverse router was used as the username. Passwords must be the same on both routers. PAP or CHAP authentication can be enabled.

###Router 01 setup:

{% codeblock %}
Router(config)#hostname Router01
Router01(config)#username Router02 password mypassword
Router01(config)#int s1/0
Router01(config-if)#ip address 192.168.1.1 255.255.255.0
Router01(config-if)#encapsulation ppp
Router01(config-if)#ppp authentication chap
Router01(config-if)#no shut
{% endcodeblock %}

###Router 02 setup:

{% codeblock %}
Router(config)#hostname Router02
Router02(config)#username Router01 password mypassword
Router02(config)#int s1/0
Router02(config-if)#ip address 192.168.1.2 255.255.255.0
Router02(config-if)#encapsulation ppp
Router02(config-if)#ppp authentication chap
Router02(config-if)#no shut
{% endcodeblock %}

###Verify:

{% codeblock %}
Router01#sh int s1/0
Router02#sh int s1/0
{% endcodeblock %}
