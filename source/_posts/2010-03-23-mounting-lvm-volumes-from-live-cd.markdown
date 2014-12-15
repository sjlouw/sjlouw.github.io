---
layout: post
title: "Mounting LVM volumes from live CD"
date: 2010-03-23
comments: true
categories: linux
---
{% img left /images/blog_posts/lvm.png %}

Instructions for mounting Linux LVM volumes with a live CD. I've used a Gentoo minimal install/Live CD.
<!--more-->
<br>
<br>
<br>
<br>

...boot the Live CD and open a terminal.

**Mounting the LVM Volume:**

{% codeblock lang:bash %}
vgchange -a y
lvscan
mount -t ext3 /dev/VolGroup00/LogVol00 /mnt/
{% endcodeblock %}

**Unmounting the LVM Volume:**

{% codeblock lang:bash %}
umount /mnt/
vgchange -a y
{% endcodeblock %}
