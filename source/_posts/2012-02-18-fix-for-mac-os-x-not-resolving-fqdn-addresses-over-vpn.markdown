---
layout: post
title: "Fix for Mac OS X not resolving FQDN addresses over VPN"
date: 2012-02-18
comments: true
categories: macosx dns vpn
---
{% img left /images/blog_posts/dns2.png %}

On Mac OS X Lion (This also apply to all later versions of OSX), while connected to VPN, I was not able to resolve hostnames on a remote site. The solution was to create domain resolver files in `/etc/resolver` named for the different domains the I wanted resolved<!--more-->, for example:
<br>
<br>
{% codeblock lang:bash %}
sudo mkdir /etc/resolver

vi /etc/resolver/yourdomain.com
{% endcodeblock %}

Add the following content and save the file:

{% codeblock %}
nameserver x.x.x.x <- the DNS server to resolve hosts for yourdomain.com
domain yourdomain.com
port 53
{% endcodeblock %}

You can create as much as needed custom domain resolver files, one for each domain.
