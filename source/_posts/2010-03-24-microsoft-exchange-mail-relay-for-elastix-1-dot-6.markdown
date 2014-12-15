---
layout: post
title: "Microsoft Exchange mail relay for Elastix 1.6"
date: 2010-03-24
comments: true
categories: linux
---
{% img left /images/blog_posts/elastix.png %}

Relaying e-mail via Microsoft Exchange server for Elastix 1.6 or CentOS 5.x. You will need to have a valid Active Directory domain user with an Exchange mailbox to send mail as.
<!--more-->
<br>
<br>
<br>

##Configuring Postfix:
SSH to your Elastix/CentOS box and run the following commands:

{% codeblock lang:bash %}
postconf -e 'relayhost = exchange-ip-address'
postconf -e 'smtp_sasl_auth_enable = no'
postconf -e 'smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd'
postconf -e 'smtp_sasl_security_options ='
postconf -e 'mydomain = your-domain-name.com'
postconf -e 'myhostname = elastix-hostname.your-domain-name.com'

echo "exchange-ip-address   domain-username:domain-password" > /etc/postfix/sasl_passwd

chown root:root /etc/postfix/sasl_passwd
chmod 600 /etc/postfix/sasl_passwd

postmap /etc/postfix/sasl_passwd
{% endcodeblock %}

##Restarting Postfix to apply changes:

{% codeblock lang:bash %}
/etc/init.d/postfix restart
{% endcodeblock %}

###Testing if mail works:

{% codeblock lang:bash %}
mail -s test valid-email@whatever.com < /etc/hosts
{% endcodeblock %}

If everything is configured correctly, you should get an email from your Elastix box with subject "test".
