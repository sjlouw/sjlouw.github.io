---
layout: post
title: "Asterisk - registered Useragent audit"
date: 2012-02-19
comments: true
categories: linux asterisk
---
{% img left /images/blog_posts/asterisk.png %}

Here is a quick script I put together to get a list of all phone types currently registered to a Asterisk box:
<!--more-->
<br>
<br>
<br>
<br>
{% codeblock lang:bash %}
#!/bin/bash

for i in `asterisk -rx "sip show peers" | grep -av Unspecified | grep -a "/" | grep -a "^[0-9]" | cut -f 1 -d '/'`
do
user=`asterisk -rx "sip show peer $i" | grep -a "Useragent"`
echo $i = $user |awk '{ print $1","$5 }'
done
{% endcodeblock %}
