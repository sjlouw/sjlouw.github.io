---
layout: post
title: "Cisco IOS updating multiple no-ip dynamic DNS host names"
date: 2014-01-20 00:27:31 +0200
comments: true
categories: cisco
---
{% img left /images/blog_posts/noip.png %}

Setting up a Cisco IOS router to update multiple [no-ip](http://www.noip.com) dynamic DNS host names.
<!--more-->
<br>
<br>
<br>
<br>
<br>

###The following variables will be used:
* **no-ip Username:** noipusername
* **no-ip Password:** noippassword
* **no-ip Hostname1:** someddnshostname1.sytes.net
* **no-ip Hostname1:** someddnshostname2.sytes.net

###First create the [no-ip](http://www.noip.com) connection configuration.

{% codeblock %}
ip ddns update method no-ip
 HTTP
  add http://noipusername:noippassword@dynupdate.no-ip.com/nic/update?hostname=someddnshostname1.sytes.net,someddnshostname1.sytes.net&myip=<a>
  remove http://noipusername:noippassword@dynupdate.no-ip.com/nic/update?hostname=someddnshostname1.sytes.net,someddnshostname1.sytes.net&myip=<a>
 interval maximum 0 0 5 0
{% endcodeblock %}

###Then on the router's, internet facing, outside interface, apply the config.

{% codeblock %}
interface Dialer0
 ip ddns update hostname someddnshostname1.sytes.net
 ip ddns update no-ip
{% endcodeblock %}
