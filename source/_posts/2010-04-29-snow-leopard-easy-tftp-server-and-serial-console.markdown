---
layout: post
title: "Snow Leopard easy TFTP server and serial console"
date: 2010-04-29
comments: true
categories: macosx
---
{% img left /images/blog_posts/tftp.png %}

I couldn't find an easy working TFTP server and serial Terminal for Mac OS X Snow Leopard, so I've build two simple AppleScript programs to make my life easier for updating and configuring Cisco equipment and serial connectivity.
<!--more-->
<br>
<br>
<br>
Download Links:

[TFTP Controller](http://www.mediafire.com/?eyndtecwgzy)<br>
[Serial Connection](http://www.mediafire.com/?q0g1tmng04g)

####Changing the default TFTP server directory in Snow Leopard:

**NOTE:** The directory must have full 777 permissions, for all users, to be able to "get" and "put" data from and to TFTP server:

{% codeblock lang:bash %}
mkdir /Users/yourusername/TFTProot
chmod -R 777 /Users/yourusername/TFTProot
{% endcodeblock %}

{% codeblock lang:bash %}
sudo vi /System/Library/LaunchDaemons/tftp.plist
{% endcodeblock %}

Change<br>
`<string>/private/tftpboot</string>`<br>
to your new path<br>
`<string>/Users/yourusername/TFTProot</string>`

####You can also start or stop the build-in TFTP server of Snow Leopard with the following commands:

Starting the TFTP server:

{% codeblock lang:bash %}
sudo launchctl load -F -w /System/Library/LaunchDaemons/tftp.plist
{% endcodeblock %}

Stopping the TFTP server:

{% codeblock lang:bash %}
sudo launchctl unload -F -w /System/Library/LaunchDaemons/tftp.plist
{% endcodeblock %}
