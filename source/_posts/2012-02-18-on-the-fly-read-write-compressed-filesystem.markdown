---
layout: post
title: "On-the-Fly Read-Write compressed filesystem"
date: 2012-02-18
comments: true
categories: linux other
---
{% img left /images/blog_posts/zip.png %}

I recently had a problem where ["SARG"](http://sourceforge.net/projects/sarg/) completely chew up all root filesystem space as reports was generated daily and stored under /var/www/html/sarg. Quick solution... I thought this can also come in handy for future reference... "On-the-Fly read-write compressed filesystem"
<!--more-->
<br>
<br>
I did the following on CentOS 5.5:

{% codeblock lang:bash %}
rpm -Uhv http://apt.sw.be/redhat/el5/en/x86_64/rpmforge/RPMS//rpmforge-release-0.3.6-1.el5.rf.x86_64.rpm

yum -y install squashfs-tools fuse-unionfs

mv /var/www/html/sarg /root/sarg-old
mkdir /var/www/html/sarg

mksquashfs /root/sarg-old /.sarg-compressed.sqfs -check_data

mkdir -p /var/squashed/{ro,rw}
{% endcodeblock %}

Add the following to /etc/fstab:
{% codeblock %}
/.sarg-compressed.sqfs  /var/squashed/ro  squashfs  loop,ro  0 0
unionfs#/var/squashed/rw=rw:/var/squashed/ro=ro /var/www/html/sarg fuse default_permissions,allow_other,use_ino,nonempty,suid,cow 0 0
{% endcodeblock %}

{% codeblock lang:bash %}
mount -all

touch /var/www/html/sarg/test
rm -rf /var/www/html/sarg-old
{% endcodeblock %}

Check that your new fuse filesystem is mounted:

{% codeblock lang:bash %}
df -h
{% endcodeblock %}

By doing this, all files written to /var/www/html/sarg is actually being written "inside" /.sarg-compressed.sqfs (The compressed filesystem) Files like text, or html in this instance, are compressed at a massive ratio.
