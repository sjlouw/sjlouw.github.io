---
layout: post
title: "Apache LDAP authentication"
date: 2012-02-21
comments: true
categories: linux
---
{% img left /images/blog_posts/apache.png %}

This guide will show how to configure the Apache web server to authenticate users against Microsoft Active Directory. The most difficult part was to get the correct LDAP path for the `AuthLDAPUrl` and `AuthLDAPBindDN` parameters. Without the exact correct path it will NOT work, because LDAP does not traverse the Active Directoy for specified users, but relies on the exact full path specified.
<!--more-->

### CentOS - Apache, authenticating Microsoft Active Directory users:

`vi /etc/httpd/conf/httpd.conf`

Make sure the following 3 lines are NOT hashed out: 

{% codeblock %}
LoadModule authnz_ldap_module modules/mod_authnz_ldap.so
LoadModule ldap_module modules/mod_ldap.so
LoadModule auth_basic_module modules/mod_auth_basic.so
{% endcodeblock %}

Wherever your web directory is, still in `/etc/httpd/conf/httpd.conf`:

{% codeblock %}
<Directory "/var/www/html">

Options Indexes FollowSymLinks
Order deny,allow
Deny from All
AuthName "AD Username Password please"
AuthType Basic
AuthBasicProvider ldap
AuthzLDAPAuthoritative off
AuthLDAPUrl "ldap://your_dc_fqdn:389/OU=SOME_OU,DC=yourdomain,DC=com?sAMAccountName?sub?(objectClass=*)" NONE
AuthLDAPBindDN "CN=your_AD_user,CN=Users,DC=yourdomain,DC=com"
AuthLDAPBindPassword your_AD_user_password
Require valid-user
Satisfy any

</Directory>
{% endcodeblock %}

`vi /etc/openldap/ldap.conf`

Hash everything out and add the following line: 
{% codeblock %}
REFERRALS off 
{% endcodeblock %}

Restart Apache
{% codeblock lang:bash %}
/etc/init.d/httpd restart
{% endcodeblock %}

Now if you go to your web server's root with your browser, you will be prompted for a username and password. If you do have a valid Active Directory user account, you will be authenticated against AD.
