---
layout: post
title: "Cisco IOS DNS server"
date: 2010-04-30
comments: true
categories: cisco dns
---
{% img left /images/blog_posts/cisco3.png %}

Setting up and running a full-fletched DNS server on Cisco IOS, most users will probably only need to do the following to enable the Cisco IOS DNS server for name lookups: For a more advanced DNS server configuration please read on...
<!--more-->
<br>
<br>
<br>

{% codeblock %}
Router01(config)#ip dns server
Router01(config)#ip host Router02 192.168.1.2
Router01(config)#ip host Router03 192.168.1.3
Router01(config)#ip host Router04 192.168.1.4
{% endcodeblock %}

And then on the client routers:

{% codeblock %}
Router02(config)#ip name-server 192.168.1.1 (where 192.168.1.1 is the IP address of the router acting as DNS server)
{% endcodeblock %}

For a more advanced DNS server configuration, I will create two primary domains, mywebsite1.com and mywebsite2.com. Under each I will add 3 ISP nameservers, 2 MX mail records and 1 host record to make the www work.

* **86400** = 24 hours Refresh time
* **3600** = 1 hour Refresh retry time
* **1209600** = 14 days Authority expire time
* **86400** = 24 hours Minimum TTL for zone info

{% codeblock %}
Router01(config)#ip dns server
{% endcodeblock %}

For the www.mywebsite1.com domain:

{% codeblock %}
Router01(config)#ip dns primary mywebsite1.com soa isp.ns1.mywebsite1.com admin@mywebsite1.com 86400 3600 1209600 86400
Router01(config)#ip host mywebsite1.com ns isp.ns1.mywebsite1.com
Router01(config)#ip host mywebsite1.com ns isp.ns2.mywebsite1.com
Router01(config)#ip host mywebsite1.com ns isp.ns3.mywebsite1.com
Router01(config)#ip host mywebsite1.com mx 1 mail.mywebsite1.com
Router01(config)#ip host mywebsite1.com mx 2 mail2.mywebsite1.com
Router01(config)#ip host mail.mywebsite1.com 192.168.1.40
Router01(config)#ip host mail2.mywebsite1.com 192.168.1.50
Router01(config)#ip host www.mywebsite1.com 192.168.1.30
{% endcodeblock %}

For the www.mywebsite2.com domain:

{% codeblock %}
Router01(config)#ip dns primary mywebsite2.com soa isp.ns1.mywebsite2.com admin@mywebsite2.com 86400 3600 1209600 86400
Router01(config)#ip host mywebsite2.com ns isp.ns1.mywebsite2.com
Router01(config)#ip host mywebsite2.com ns isp.ns2.mywebsite2.com
Router01(config)#ip host mywebsite2.com ns isp.ns3.mywebsite2.com
Router01(config)#ip host mywebsite2.com mx 1 mail.mywebsite2.com
Router01(config)#ip host mywebsite2.com mx 2 mail2.mywebsite2.com
Router01(config)#ip host mail.mywebsite2.com 192.168.1.70
Router01(config)#ip host mail2.mywebsite2.com 192.168.1.80
Router01(config)#ip host www.mywebsite2.com 192.168.1.60
{% endcodeblock %}

Verify the DNS setup:

{% codeblock %}
Router01(config)#show hosts
{% endcodeblock %}
