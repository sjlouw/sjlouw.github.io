---
layout: post
title: "CentOS 5.7 - Build and install BIND DNS RPMS"
date: 2012-03-02
comments: true
categories: linux dns
---
{% img left /images/blog_posts/dns.png %}

This guide is just a quick howto compile and build the Linux DNS server, BIND, version 9.9.0-1 RPMS from the source RPM.
<!--more-->
<br>
<br>
<br>
### Building BIND 9.9.0-1 DNS server RPMS on CentOS 5.7:

{% codeblock lang:bash %}
yum -y install make gcc rpm-build libtool autoconf openssl-devel libcap-devel libidn-devel libxml2-devel openldap-devel postgresql-devel sqlite-devel mysql-devel krb5-devel xmlto openscap-devel

cd /usr/src/redhat/SRPMS
wget http://centos.alt.ru/pub/repository/centos/5/SRPMS/bind-9.9.0-1.el5.src.rpm
rpm -ivh --nomd5 bind-9.9.0-1.el5.src.rpm

cd /usr/src/redhat/SPECS
rpmbuild -ba ./bind9_9.spec

cd /usr/src/redhat/RPMS/x86_64/
rpm -Uvh bind-9.9.0-1.x86_64.rpm bind-chroot-9.9.0-1.x86_64.rpm bind-utils-9.9.0-1.x86_64.rpm bind-libs-9.9.0-1.x86_64.rpm bind-devel-9.9.0-1.x86_64.rpm
{% endcodeblock %}
