---
layout: post
title: "Cisco IOS and DNS views"
date: 2014-01-20 22:01:30 +0200
comments: true
categories: cisco dns
---
{% img left /images/blog_posts/cisco14.png %}

Using DNS views on a Cisco router to query specific DNS servers for specified domains. In my case I'm using it to direct all DNS queries to a service such as [Unotelly](http://www.unotelly.com/unodns/) for accessing region locked web content ([Netflix](https://signup.netflix.com), [Hulu](http://www.hulu.com) and [Pandora](http://www.pandora.com) Internet Radio).
<!--more-->
<br>
<br>

###Variables used:
* **UnoDNS Server 1:** UnoDNS_server1
* **UnoDNS Server 2:** UnoDNS_server2
* **Vlan1** is the local subnet
* **8.8.8.8** and **4.4.4.4** is the default DNS servers

###Create the alternate DNS view:
These DNS servers will be used for the specified domains.

{% codeblock %}
ip dns view GEOLOCKBYPASS
dns forwarder UnoDNS_server1
dns forwarder UnoDNS_server2
dns forwarding source-interface Vlan1
{% endcodeblock %}

###Create the default DNS view:
These DNS servers will be the default for all other queries.

{% codeblock %}
ip dns view default
 domain timeout 2
 dns forwarder 8.8.8.8
 dns forwarder 4.4.4.4
{% endcodeblock %}

###Create a view-list and add the above created view:
{% codeblock %}
ip dns view-list DNS
 view GEOLOCKBYPASS 10
  restrict name-group 1
 view default 1000
{% endcodeblock %}

###Specify what domains will use the alternate DNS view:
{% codeblock %}
ip dns name-list 1 permit \.HULU\.COM
ip dns name-list 1 permit \.PANDORA\.COM
ip dns name-list 1 permit \.NETFLIX\.COM
ip dns server view-group DNS
ip dns server
{% endcodeblock %}
