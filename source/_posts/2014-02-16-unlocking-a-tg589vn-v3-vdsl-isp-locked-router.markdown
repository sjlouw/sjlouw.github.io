---
layout: post
title: "Unlocking a TG589vn v3 VDSL ISP locked router"
date: 2014-02-16 11:55:09 +0200
comments: true
categories: other
---
{% img left /images/blog_posts/tg589vnv3_pwned.png %}

Unlocking a locked ISP, TG589vn v3 VDSL router by installing generic Technicolor firmware. The following procedure was what worked for me.
<!--more-->
<br>
<br>
<br>
<br>

First give your PC a static IP address 192.168.1.2 / 255.255.255.0

Download and install [Tftpd32](http://www.jounin.net/tftpd32_download.html)

Download the generic binary firmware file from [here](http://www.mediafire.com/download/ku9yo2wsl577npf/MST_TG589vnv3_R10.5.2.F_BN.bin), and put it under "C:\Program Files\Tftpd32\" or wherever your TFTP root is.

###Configure the TFTP software as below:
{% img center /images/blog_posts/tftpd32_1.png %}
{% img center /images/blog_posts/tftpd32_2.png %}

Plug the router in the ethernet port as shown below:
{% img center /images/blog_posts/tg589vnv3_ports.png %}

###Make sure you can ping the router IP address from your PC.
{% codeblock %}
ping 192.168.1.1

PING 192.168.1.1 (192.168.1.1): 56 data bytes
64 bytes from 192.168.1.1: icmp_seq=0 ttl=255 time=1.075 ms
64 bytes from 192.168.1.1: icmp_seq=1 ttl=255 time=1.802 ms
{% endcodeblock %}

###Resetting the router to the default settings and start flashing it:
Hold the small button in the hole on the back of the router for more than 16 seconds, while it's powered on. (Blue "Upgrade" light will come on) The router will restart and start the upgrade process with BOOTP / TFTP. You will see a small box pop up showing the binary file transfer to the router. After that is done, the router will unpack the new file and boot it. DO NOT RESTART your router at any point until you can see all lights in the front of the router showing the normal green status.

The firmware flash will be done at this point and you should be able to ping your router at it's new default IP address, 192.168.1.254.

Open your browser and and go to http://192.168.1.254, go to advanced settings and change the Administrator password.

You should be able to SSH to your router now with full root access. Username: Administrator and password will be the one you set above. You will also see in the web interface that the branded ISP logo is gone and the generic Technicolor logo will show.
{% img center /images/blog_posts/tg589vnv3_ssh.png %}
{% img center /images/blog_posts/tg589vnv3_web.png %}

###Here is some advanced CLI documentation for this router:
* [TG589vn-v3_EN_r10.4.B.B_mh.pdf](http://www.mediafire.com/view/8hiaraeqx31d8hj/TG589vn-v3_EN_r10.4.B.B_mh.pdf)
* [command_list_v2.txt](http://www.mediafire.com/view/3j6s1mmloottjbi/command_list_v2.txt)
* [command_mlp_v2.txt](http://www.mediafire.com/view/vtdihfhs73ce5gb/command_mlp_v2.txt)

You can also install some PINS on the router's mainboard and get serial console access, by following the below guide posted on youtube by someone else...
{% youtube FwiosVp4Ra0 %}
