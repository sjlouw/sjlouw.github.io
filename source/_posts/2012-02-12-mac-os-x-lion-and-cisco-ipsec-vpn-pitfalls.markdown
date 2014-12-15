---
layout: post
title: "Mac OS X Lion and Cisco IPSEC VPN pitfalls"
date: 2012-02-12
comments: true
categories: cisco macosx vpn
---
{% img left /images/blog_posts/cisco_vpn.png %}

The Mac OS X Lion, Native VPN client, with Cisco IPSEC EasyVPN Server was NOT working properly for myself. The problem I faced was that traffic was NOT passed to the remote LAN when connected to VPN. Split-tunnel and normal EasyVPN setups did NOT work.
<!--more-->
<br>
<br>
1.  **When presented with a split-tunnel ACL the Apple client will create a proxy pair for each line.**
      * i.e. VPN IP address of A with a split ACL of:
    
        * permit B
        * permit C
        * permit D

     You would see an ipsec sa from A to B, A to C, and A to D.

2.  **When presented with a split-tunnel ACL the Cisco client will create a single ipsec sa:**
      * i.e. A to any

    However the client will only route traffic to B, C, D over the tunnel.

This is fine and has no problems when using a crypto map style setup for Cisco EasyVPN.

However when you configure the use of dVTI this becomes difficult.  This is because the VTI can only support 1 ipsec sa built to it.  As a results when the Apple client tries to propose the proxy pair for the A to C entry it is rejected.

This leaves you with two options here:

1.  Switch to a tunnel-all configuration
2.  Switch back to the crypto map configuration rather than the virtual-template configuration.

[Reference](https://supportforums.cisco.com/thread/2095921)

###I chose to take the "old" crypto map style setup. Here's how I made it work on a Cisco 877 DSL router:

{% codeblock %}
ip nat inside source route-map NAT interface Dialer0 overload

route-map NAT permit 10
match ip address 111
exit

access-list 101  remark ----------------------------------------------
access-list 101  remark *****VPN Access-list*****
access-list 101  permit ip 172.16.20.0 0.0.0.255 172.16.40.0 0.0.0.15
!
access-list 111  remark ----------------------------------------------
access-list 111  remark *****DENY Local LAN to VPN Traffic*****
access-list 111  deny ip 172.16.20.0 0.0.0.255 172.16.40.0 0.0.0.15 
access-list 111  remark ----------------------------------------------
access-list 111  remark *****PERMIT Networks Internet Access*****
access-list 111  permit ip 172.16.20.0 0.0.0.255 any
access-list 111  permit ip any any

aaa new-model
aaa authentication login userauth local
aaa authorization network groupauth local

username myusername password 0 mypassword

crypto isakmp policy 3
encryption 3des
authentication pre-share
group 2
lifetime 86400
exit

crypto isakmp client configuration group my_vpn
key mysecretgroupkey
dns 172.16.20.1 8.8.8.8
domain my.domain
pool my_vpn_pool
acl 101
max-logins 10
max users 10
save-password
split-dns my.domain
include-local-lan
exit

crypto ipsec transform-set my_set esp-3des esp-md5-hmac
exit

crypto dynamic-map dynmap 10
set transform-set my_set
set security-association idle-time 900
reverse-route
exit

crypto map clientmap client authentication list userauth
crypto map clientmap isakmp authorization list groupauth
crypto map clientmap client configuration address respond
crypto map clientmap 10 ipsec-isakmp dynamic dynmap

ip local pool my_vpn_pool 172.16.40.2 172.16.40.8

interface Dialer0
ip nat outside
crypto map clientmap

interface vlan1
no autostate
ip nat inside
{% endcodeblock %}

I have tested this setup with Mac OS X Lion VPN client and with iPhone IOS 5.0.1. All is working well now. Yeeaay!
