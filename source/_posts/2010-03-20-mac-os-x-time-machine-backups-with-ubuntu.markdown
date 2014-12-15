---
layout: post
title: "Mac OS X Time Machine backups with Ubuntu"
date: 2010-03-20
comments: true
categories: macosx linux other
---
{% img left /images/blog_posts/timemachine.png %}

Setting up a Ubuntu Time Machine server for Mac OS X Snow Leopard backups. [Netatalk](http://netatalk.sourceforge.net/) 2.0.5-3 and Ubuntu 9.10 minimal was used for this configuration.
<!--more-->
<br>
**UPDATE 2014-01-13: The installation procedure has changed with later versions of Ubuntu and [netatalk](http://netatalk.sourceforge.net/).
<br>

{% codeblock lang:bash %}
wget http://ftp.us.debian.org/debian/pool/main/d/db/libdb4.8_4.8.26-1_i386.deb
dpkg -i libdb4.8_4.8.26-1_i386.deb
wget http://ftp.us.debian.org/debian/pool/main/libg/libgcrypt11/libgcrypt11_1.4.5-2_i386.deb
dpkg -i libgcrypt11_1.4.5-2_i386.deb

apt-get install libcrack2

wget http://ftp.us.debian.org/debian/pool/main/n/netatalk/netatalk_2.0.5-3_i386.deb
dpkg -i netatalk_2.0.5-3_i386.deb

mkdir /storage/TimeMachine
chmod a+rw /storage/TimeMachine
{% endcodeblock %}

{% codeblock lang:bash %}
vi /etc/netatalk/AppleVolumes.default
{% endcodeblock %}

change the last part to look like this:

{% codeblock lang:bash %}
# By default all users have access to their home directories.
#~/                     "Home Directory"
/storage/TimeMachine "TimeMachine" options:tm
{% endcodeblock %}

{% codeblock lang:bash %}
/etc/init.d/netatalk restart

apt-get install avahi-daemon
apt-get install libnss-mdns
{% endcodeblock %}

{% codeblock lang:bash %}
vi /etc/nsswitch.conf
{% endcodeblock %}

add "mdns to end of line to look like this:

{% codeblock lang:bash %}
hosts:          files mdns4_minimal [NOTFOUND=return] dns mdns4 mdns
{% endcodeblock %}

{% codeblock lang:bash %}
vi /etc/avahi/services/afpd.service
{% endcodeblock %}

Paste the following code and save:

{% codeblock lang:xml %}
<!--*-nxml-*-->
<service-group>
<name replace-wildcards="yes">%h</name>
<service>
<type>_afpovertcp._tcp</type>
<port>548</port>
</service>
<service>
<type>_device-info._tcp</type>
<port>0</port>
<txt-record>model=Xserve</txt-record>
</service>
</service-group>
{% endcodeblock %}

Reboot the server:

{% codeblock lang:bash %}
init 6
{% endcodeblock %}

**NOTES:**<br>
After the machine returns from reboot, you can check to see if everything is running:

`ps -ef | grep afpd`<br>
`ps -ef | grep avahi`<br>

**Should you need to manually connect to server via finder, you can:**

Go --> Connect to Server and type `afp://your-server-ip`

If you were not able to log in, you can check in `/var/log/daemon.log` on your linux server
for any debug messages.
