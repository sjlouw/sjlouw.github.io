---
layout: post
title: "SSH passwordless public private key authentication"
date: 2010-04-10
comments: true
categories: linux
---
{% img left /images/blog_posts/ssh.png %}

Creating and setting up SSH key based authentication for passwordless remote server logins on Linux or Mac OS X.
<!--more-->
<br>
<br>
<br>
<br>

**On the local server or workstation, logged in as, eg. myuser:**

{% codeblock lang:bash %}
mkdir ~/.ssh
cd ~/.ssh
ssh-keygen -t rsa
{% endcodeblock %}

**NOTE:** When asked for file in wich to save the key, make sure the path reads something like /home/myuser/.ssh/. We want to save the new generated key-pair in the full path to above created `.ssh` folder. Also make sure not to specify a passphrase, otherwise, you will still have to input a password at logon. You can also choose to generate a DSA key-pair in place of the RSA key-pair.

{% codeblock lang:bash %}
scp ~/.ssh/id_rsa.pub remoteuser@remote-server:/home/remoteuser/id_rsa.local-server.pub
{% endcodeblock %}

**On the remote server, you wish to log into:**

{% codeblock lang:bash %}
mkdir ~/.ssh
chmod 700 ~/.ssh
cat /home/remoteuser/id_rsa.local-server.pub >> /home/remoteuser/.ssh/authorized_keys
chmod 644 /home/remoteuser/.ssh/authorized_keys
{% endcodeblock %}

Now you can test from the local server or workstation if it works:
{% codeblock lang:bash %}
ssh remoteuser@remote-server
{% endcodeblock %}
