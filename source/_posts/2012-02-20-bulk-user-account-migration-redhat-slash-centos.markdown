---
layout: post
title: "Bulk user account migration - RedHat/CentOS"
date: 2012-02-20
comments: true
categories: linux
---
{% img left /images/blog_posts/useraccounts.png %}

Here are two scripts to transfer user accounts from one RedHat/CentOS server to another. All home directory files, mail, group settings, passwords stays in tact. The first script needs to be executed on the source server and the second script needs to be executed on the destination server:
<!--more-->
<br>
<br>
{% codeblock lang:bash %}
#!/bin/bash
#Run on source server

DESTSERVER=<destination_server_ip>
export UGIDLIMIT=500
mkdir /root/usersmigrate

awk -v LIMIT=$UGIDLIMIT -F: '($3>=LIMIT) && ($3!=65534)' /etc/passwd > /root/usersmigrate/passwd.mig
awk -v LIMIT=$UGIDLIMIT -F: '($3>=LIMIT) && ($3!=65534)' /etc/group > /root/usersmigrate/group.mig
awk -v LIMIT=$UGIDLIMIT -F: '($3>=LIMIT) && ($3!=65534) {print $1}' /etc/passwd | tee - |egrep -f - /etc/shadow > /root/usersmigrate/shadow.mig
cp /etc/gshadow /root/usersmigrate/gshadow.mig

scp -rp /root/usersmigrate root@$DESTSERVER:/root/

tar zcvf - /home/ | ssh root@$DESTSERVER 'cd /; tar zxvf - '
tar zcvf - /var/spool/mail/ | ssh root@$DESTSERVER 'cd /; tar zxvf - '
{% endcodeblock %}

{% codeblock lang:bash %}
#!/bin/bash
#Run on destination server

export UGIDLIMIT=500

awk -v LIMIT=$UGIDLIMIT -F: '($3<=LIMIT) && ($3!=500)' /etc/passwd > /etc/passwdnew
awk -v LIMIT=$UGIDLIMIT -F: '($3<=LIMIT) && ($3!=500)' /etc/group > /etc/groupnew
awk -v LIMIT=$UGIDLIMIT -F: '($3<=LIMIT) && ($3!=500) {print $1}' /etc/passwd | tee - |egrep -f - /etc/shadow > /etc/shadownew

cat /root/usersmigrate/passwd.mig >> /etc/passwdnew
mv -f /etc/passwdnew /etc/passwd
cat /root/usersmigrate/group.mig >> /etc/groupnew
mv -f /etc/groupnew /etc/group
cat /root/usersmigrate/shadow.mig >> /etc/shadownew
mv -f /etc/shadownew /etc/shadow
cp /root/usersmigrate/gshadow.mig /etc/gshadow
{% endcodeblock %}
