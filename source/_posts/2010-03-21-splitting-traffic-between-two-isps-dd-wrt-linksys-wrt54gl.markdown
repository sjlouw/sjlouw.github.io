---
layout: post
title: "Splitting traffic between two ISPs DD-WRT Linksys WRT54GL"
date: 2010-03-21
comments: true
categories: linux other
---
{% img left /images/blog_posts/ddwrt2.png %}

Setting up a Linksys WRT-54GL 1.1 wireless router to split traffic between two ISPs and do traffic accounting. This was done to least cost route data, in South Africa, due to much cheaper local bandwidth and over priced international bandwidth.
<!--more-->
<br>

**NOTE:** You will need to have the [SD mod](http://www.hendlsofen.de/WRT54GL/eng/WRT54GL_SDMod.html) installed on your router:

**Firmware used, DD-WRT, eko branch:**
[http://www.dd-wrt.com/dd-wrtv2/downloads/others/eko/V24_TNG/svn13491-snow/NEWD/dd-wrt.v24-13491_NEWD_std.bin](http://www.dd-wrt.com/dd-wrtv2/downloads/others/eko/V24_TNG/svn13491-snow/NEWD/dd-wrt.v24-13491_NEWD_std.bin)

**MMC/SD Card Support must be enabled on the router's web interface:**

* GPIO pins select Manual
* GPIO pins DI:2  D0:4  CLK:3  CS:7

##Installing BWLOG for traffic accounting:

SSH to your router

{% codeblock lang:bash %}
cd /mmc/jffs/scripts
wget http://www.krikkit.net/download/wrtbwlog_cust_exp.tgz

tar -zxvf wrtbwlog_cust_exp.tgz
{% endcodeblock %}

The web page will be accessible by going to `http://your_router_ip:8000`

##Installing scripts for traffic splitting:

On the router's web interface:

* Administration --> Commands --> Startup

**Startup script:**

{% codeblock lang:bash %}
#!/bin/sh
PATH=/usr/sbin:/sbin:/usr/bin:$PATH

umount /jffs
mount --bind /mmc/jffs /jffs

killall redial
killall pppd

#INTERNATIONAL
INTLUSER=isp1_username
INTLPASS=isp1_password
#LOCAL
LOCALUSER=isp2_username
LOCALPASS=isp2_password
#OTHER SETTINGS
INTRFACE=nic-vlan1
SAIXSMTP=196.43.2.142
ROUTESERVER=196.38.40.110
INTL=ppp0
LOCL=ppp1
TIMEOUT=120

setintlroutes () {
echo ...setting International routes
route add -host $ROUTESERVER $INTL
route add -host $SAIXSMTP $INTL
}

setdefaultroute () {
echo ...applying default route
route del default
route del default
route del default
route add default $INTL
}

getloclroutes () {
echo Downloading Local routes...
sleep 5
wget -T 15 "http://developers.locality.co.za/routes.txt" -O /tmp/routes.dat
sleep 7

if [ ! -f /tmp/routes.dat ]
  then
    echo ...restoring backup routes.txt file
    cp /mmc/jffs/scripts/routes.dat.bak /tmp/routes.dat
    sleep 7
fi
}

backuploclroutes () {
echo ...backing up existing routes.txt file
cp /tmp/routes.dat /mmc/jffs/scripts/routes.dat.bak
rm -rf /tmp/routes.dat
}

setloclroutes () {
echo ...setting Local routes 
for IP in `cat /tmp/routes.dat`
  do
    gpio enable 7
    route add -net $IP $LOCL
    gpio disable 7
  done
}

connect () {
gpio disable 3; sleep 1
pppd plugin /usr/lib/rp-pppoe.so $INTRFACE noipdefault noauth nodefaultroute noaccomp noccp nobsdcomp nodeflate nopcomp novj novjccomp nomppe nomppc usepeerdns user $1 password $2 default-asyncmap mtu 1492 mru 1492 persist lcp-echo-interval 60 lcp-echo-failure 10 maxfail 0 unit $3
gpio enable 3; sleep 1
}

connlinkintl () {
while true
  do
    if [ `ip link show dev ppp0 |grep ppp0 |awk '{ print $2 }'` == "ppp0:" ]
      then
        echo ...International ppp link is up
        break
      else
        echo ...waiting for International to connect
        gpio disable 3; sleep 1
        gpio enable 3; sleep 1
    fi
done
}

connlinklocl () {
while true
  do
     if [ `ip link show dev ppp1 |grep ppp1 |awk '{ print $2 }'` == "ppp1:" ]
       then
         echo ...Local ppp link is up
         break
       else
         echo ...waiting for Local to connect
         gpio disable 3; sleep 1
         gpio enable 3; sleep 1
     fi
done
}

sleep 20
cd /mmc/jffs/scripts/bwlog/
./start.sh &

echo Starting up Traffic Splitting... Please wait...
sleep 40

while true
  do
    if [ `ip link show dev ppp0 |grep ppp0 |awk '{ print $2 }'` == "ppp0:" ]
      then
         echo ...International ppp link is up
      else
         connect $INTLUSER $INTLPASS 0
         connlinkintl
         sleep 10
         setintlroutes
         setdefaultroute
    fi 
    if [ `ip link show dev ppp1 |grep ppp1 |awk '{ print $2 }'` == "ppp1:" ]
      then
         echo ...Local ppp link is up
      else 
         connect $LOCALUSER $LOCALPASS 1  
         connlinklocl
         sleep 10
         setdefaultroute
         getloclroutes
         setloclroutes
         backuploclroutes
         setdefaultroute
    fi
    if [ `ip link show dev ppp2 |grep ppp2 |awk '{ print $2 }'` == "ppp2:" ]
      then
         echo ...Resetting all ppp connections
         killall redial
         killall pppd
      else
         echo all ppp connections seems good
    fi
   echo returning to main loop...
   sleep $TIMEOUT
done
{% endcodeblock %}

Click Save Startup.

**Firewall script:**

{% codeblock lang:bash %}
#!/bin/sh
PATH=/usr/sbin:/sbin:/usr/bin:$PATH

iptables -t nat -I POSTROUTING -o ppp+ -j MASQUERADE
{% endcodeblock %}

Click Save Firewall.

Reboot the router!
