---
layout: post
title: "Decrypting Cisco type-7 passwords with IOS"
date: 2010-03-23
comments: true
categories: cisco
---
{% img left /images/blog_posts/cisco11.png %}

Decrypting Cisco type-7 passwords on a IOS router. I saw this once on the net and thought it could come in handy... This does not work for type-5 passwords!
<!--more-->
<br>
<br>
<br>

##Turn on type-7 encryption for local passwords and create a temp username:

{% codeblock %}
Router1(config)#service password-encryption
Router1(config)#username tempuser password !@&*^&*^$#
{% endcodeblock %}

##Show the created username:

{% codeblock %}
Router1(config)#do show run | include username username tempuser password 7 -encrypted string-
{% endcodeblock %}

##Create a key chain and enter the type-7 encrypted password as the key string:

{% codeblock %}
Router1(config)#key chain decrypt
Router1(config-keychain)#key 1
Router1(config-keychain-key)#key-string 7 -encrypted string-
{% endcodeblock %}

**The show command will now do the decryption:**

{% codeblock %}
Router1(config-keychain-key)#do show key chain decrypt
{% endcodeblock %}

**Key-chain decrypt:**

*key 1 -- text "testuser:decyptedpassword"*<br>
*accept lifetime (always valid) - (always valid) [valid now]*<br>
*send lifetime (always valid) - (always valid) [valid now]*
