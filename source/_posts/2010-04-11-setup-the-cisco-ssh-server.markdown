---
layout: post
title: "Setup the Cisco SSH server"
date: 2010-04-11
comments: true
categories: cisco
---
{% img left /images/blog_posts/cisco6.png %}

This is with normal "Username/Password" authentication. Apparently IOS 15 and up supports public/private key authentication...
<!--more-->
<br>
<br>
<br>
<br>

{% codeblock %}
Router(config)#hostname myhostname
myhostname(config)#ip domain-name mydomain
{% endcodeblock %}

This must be a modulus of 1024 and higher else only SSH version 1 can be enabled:
{% codeblock %}
myhostname(config)#crypto key generate rsa modulus 1024
{% endcodeblock %}

{% codeblock %}
myhostname(config)#ip ssh time-out 120
myhostname(config)# ip ssh authetication-retries 3    
myhostname(config)# ip ssh port 1234    
myhostname(config)#ip ssh version 2    
myhostname(config)#username myusername secret mypassword    
myhostname(config)#line vty 0 4    
myhostname(config)# transport input ssh    
myhostname(config-line)#login local
{% endcodeblock %}

Verify the configuration:
{% codeblock %}
myhostname(config)#show ip ssh
{% endcodeblock %}

...and remember to save the changes!
