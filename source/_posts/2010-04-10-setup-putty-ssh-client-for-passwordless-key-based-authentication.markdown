---
layout: post
title: "Setup Putty SSH client for passwordless key based authentication"
date: 2010-04-10
comments: true
categories: other
---
{% img left /images/blog_posts/putty.png %}

Setup Putty SSH client, for passwordless public private key based authentication on Windows.
<!--more-->
<br>
<br>
<br>
<br>

**Go to Start --> All Programs --> Putty --> PuTTYgen**

Set type of key to generate SSH-2 RSA and number of bits in a generated key to 1024. Click the "Generate" button and move the mouse to generate a unique key.

{% img center /images/blog_posts/putty1.png %}

When that's finished, Click "Save public key" to save your public key to a file, e.g. mykey.pub. Click "Save private key" and save it as e.g. mykey.pkk. SSH to your server with e.g. myuser, user account.

{% codeblock lang:bash %}
ssh-keygen
mkdir /home/myuser/.ssh       
echo "ssh-rsa AAAAAWNzaC1KEcNZLmBl1poSrNCQ9o5rv5ts+txlu8eFvwk== rsa-key-20100410" >> /home/myuser/.ssh/authorized_keys
{% endcodeblock %}

**NOTE:** For the purposes of this tutorial, I've shorten the random characters of the key. Also know that the whole part that you echo needs to be on one line, in other words, the part that is in quotes.

**Go to Start --> All Programs --> Putty --> PuTTY**

Set up your Session:

{% img center /images/blog_posts/putty2.png %}

**Under Connection --> Data**

{% img center /images/blog_posts/putty3.png %}

**Under Connection --> SSH --> Auth**

Click "Browse" and browse for the mykey.pkk file that you saved earlier.

{% img center /images/blog_posts/putty4.png %}

Go back to Session and click "Save". You will now have a new session entry, REMOTEHOST, that you can double-click and PuTTY will automatically login as the myuser username to your REMOTEHOST box without a password.
