---
layout: post
title: "Linksys WRT54GL, DD-WRT persistant PPPOE"
date: 2010-05-01
comments: true
categories: other linux
---
{% img left /images/blog_posts/ddwrt.png %}

A script I’ve put together to make my DSL PPPOE dialup persistant. First open the DD-WRT web interface, set WAN to PPPOE and configure a dummy username and password. Then go to Administration, Commands and paste the following script. Change your DSL username and password and save it, Startup Script.
<!--more-->
<br>
<br>
{% codeblock lang:bash %}
#!/bin/sh
PATH=/usr/sbin:/sbin:/usr/bin:$PATH

#ISP
USER=myispusername
PASS=myisppassword
#OTHER SETTINGS
INTRFACE=nic-vlan1
TIMEOUT=120

setdefaultroute () {
echo …applying default route
route del default
route del default
route del default
route add default ppp0
}

connect () {
gpio disable 3; sleep 1
pppd plugin /usr/lib/rp-pppoe.so $INTRFACE noipdefault noauth nodefaultroute noaccomp noccp nobsdcomp nodeflate nopcomp novj novjccomp nomppe nomppc usepeerdns user $1 password $2 default-asyncmap mtu 1492 mru 1492 persist lcp-echo-interval 60 lcp-echo-failure 10 maxfail 0 unit $3
gpio enable 3; sleep 1
}

connlinkppp () {
while true
 do
 if [ ip link show dev ppp0 |grep ppp0 |awk '{ print $2 }' == “ppp0:” ]
 then
 echo …ppp link is up
 break
 else
 echo …waiting for ppp to connect
 gpio disable 3; sleep 1
 gpio enable 3; sleep 1
 fi
done
}

echo Starting link checking procedure… Please wait…
sleep 40

while true
 do
 if [ ip link show dev ppp0 |grep ppp0 |awk '{ print $2 }' == “ppp0:” ]
 then
 echo …ppp link is up
 else
 connect $USER $PASS 0
 connlinkppp
 sleep 10
 setdefaultroute
 fi 

 if [ ip link show dev ppp1 |grep ppp1 |awk '{ print $2 }' == “ppp1:” ]
 then
 echo …Resetting all ppp connections
 killall redial
 killall pppd
 else
 echo all ppp connections seems good
 fi
 echo returning to main loop…
 sleep $TIMEOUT
done
{% endcodeblock %}
	
Reboot the router!
