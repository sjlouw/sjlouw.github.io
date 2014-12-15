---
layout: post
title: "Cisco IOS IPSEC VPN server and client config"
date: 2010-04-21
comments: true
categories: cisco vpn
---
{% img left /images/blog_posts/cisco9.png %}

Configuring a Cisco IOS VPN server and allowing VPN clients to connect encrypted IPSEC. This setup still allows Internet access using a Split Tunneling configuration.
<!--more-->
<br>
<br>

##This configuration assumes the following:

* VPN Client Username: myusername
* VPN Client Password: mypassword
* VPN Group authentication name: mygroupname
* VPN Group authentication password: mygroupkey
* Your internal DNS: 192.168.1.100
* Your domain: mydomain
* IPs to be assigned to VPN clients: 192.168.1.5 to 192.168.1.10/24
* FastEthernet0/0 is the router's outside interface

##Router configuration:

{% codeblock %}
Router(config)#aaa new-model
Router(config)#aaa authentication login userauth local-case
Router(config)#aaa authorization network groupauth local
Router(config)#username myusername password 0 mypassword
Router(config)#crypto isakmp policy 3
Router(config-isakmp)#encryption 3des
Router(config-isakmp)#authentication pre-share
Router(config-isakmp)#group 2
Router(config-isakmp)#exit
Router(config)#crypto isakmp client configuration group mygroupname
Router(config-isakmp-group)#key mygroupkey
Router(config-isakmp-group)#dns 192.168.1.100
Router(config-isakmp-group)#domain mydomain
Router(config-isakmp-group)#pool myvpnpool
Router(config-isakmp-group)#acl 101
Router(config-isakmp-group)#exit
Router(config)#crypto ipsec transform-set myset esp-3des esp-md5-hmac
Router(cfg-crypto-trans)#exit
Router(config)#crypto dynamic-map dynmap 10
Router(config-crypto-map)#set transform-set myset
Router(config-crypto-map)#reverse-route
Router(config-crypto-map)#exit
Router(config)#crypto map clientmap client authentication list userauth
Router(config)#crypto map clientmap isakmp authorization list groupauth
Router(config)#crypto map clientmap client configuration address respond
Router(config)#crypto map clientmap 10 ipsec-isakmp dynamic dynmap
Router(config)#int fa0/0
Router(config-if)#ip address <outside_IP_address> <subnet_mask>
Router(config-if)#no shut
Router(config-if)#ip nat outside
Router(config-if)#crypto map clientmap
Router(config-if)#exit
Router(config)#ip local pool myvpnpool 192.168.1.5 192.168.1.10
Router(config)#ip nat inside source list 111 interface FastEthernet0/0 overload
Router(config)#access-list 111 deny ip <local_network_IP> <inverted mask> 192.168.1.0 0.0.0.255
Router(config)#access-list 111 permit ip any any
Router(config)#access-list 101 permit ip <local_network_IP> <inverted mask> 192.168.1.0 0.0.0.255
{% endcodeblock %}

Remember to save your config!

Install the Cisco VPN client. Restart your computer. Open the VPN client and click "New". Fill out the details you just configured on your router:

**Still getting Error 2738 with a Windows 7 install?**

Bring up an administrative terminal:

**Start --> Run --> Type "cmd" hold ctrl+shift and press ENTER

Re-register VBScript engine:

{% codeblock %}
reg delete "HKCU\SOFTWARE\Classes\CLSID\{B54F3741-5B07-11CF-A4B0-00AA004A55E8}" /f
%systemroot%\system32\regsvr32 vbscript.dll
{% endcodeblock %}

{% img center /images/blog_posts/vpnclient.png %}

Click "Save". Double click your new connection entry and supply your configured Username and Password.

##Verifying that the VPN server is working:

**Show all current IKE Security Associations (SAs) at a peer.**

{% codeblock %}
Router#show crypto isakmp sa
{% endcodeblock %}

**Show the settings used by current SAs:**

{% codeblock %}
Router#show crypto ipsec sa
{% endcodeblock %}
