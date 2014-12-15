---
layout: post
title: "Building a secure Gentoo Linux kiosk system - part 2"
date: 2010-04-24
comments: true
categories: linux other
---
{% img left /images/blog_posts/gentoo.png %}

This is part 2 of 2 in building and setting up a secure Gentoo Linux kiosk system. Here I will configure the X system, extras and lock the system down. Part one can be followed [here](/blog/2010/04/24/building-a-secure-gentoo-linux-kiosk-system-part-1/).
<!--more-->
<br>
<br>
<br>

##Installing and configuring the X system:

{% codeblock lang:bash %}
emerge xorg-server (if you get "A file is not listed in the Manifest"... do, vi /etc/make.conf and FEATURES=-strict)

env-update
source /etc/profile

/etc/init.d/hald start
rc-update add hald default

Xorg -configure

cp /root/xorg.conf.new /etc/X11/xorg.conf
rm -rf /root/xorg.conf.new
{% endcodeblock %}

{% codeblock lang:bash %}
vi /etc/X11/xorg.conf
{% endcodeblock %}

In the Section "`Screen`" add the following under Monitor:

{% codeblock lang:bash %}
DefaultDepth 24
{% endcodeblock %}

Then in SubSection "`Display`", Depth 24 add the following under `Depth 24`:

{% codeblock lang:bash %}
Modes "1024x768" "800x600"
{% endcodeblock %}


Under Section "`InputDevice`", replace:

{% codeblock lang:bash %}
Driver "kbd"
{% endcodeblock %}
with
{% codeblock lang:bash %}
Driver "evdev"
{% endcodeblock %}

and

{% codeblock lang:bash %}
Driver "mouse"
{% endcodeblock %}
with
{% codeblock lang:bash %}
Driver "evdev"
{% endcodeblock %}

##Adding the client user:

{% codeblock lang:bash %}
useradd -m -G users,audio,wheel clientsales
{% endcodeblock %}

**Automatic login the client user:**

{% codeblock lang:bash %}
vi /etc/inittab
{% endcodeblock %}

and replace

{% codeblock lang:bash %}
c1:12345:respawn:/sbin/agetty 38400 tty1 linux
{% endcodeblock %}
with
{% codeblock lang:bash %}
c1:12345:respawn:/sbin/mingetty --autologin clientsales --noclear tty1

#Disable ctrl+alt+delete key combination
# What to do at the "Three Finger Salute".
#ca:12345:ctrlaltdel:/sbin/shutdown -r now
{% endcodeblock %}

{% codeblock lang:bash %}
vi /etc/sudoers
{% endcodeblock %}

Under the "`# User privilege specification`" section add the following:

{% codeblock lang:bash %}
clientsales ALL=(ALL) NOPASSWD: ALL
{% endcodeblock %}

##Installaing a VNC server:

{% codeblock lang:bash %}
emerge net-misc/tigervnc
vncpasswd
su clientsales
vncpasswd
exit
{% endcodeblock %}

{% codeblock lang:bash %}
vi /etc/X11/xorg.conf
{% endcodeblock %}

Add the following to Section "Module"

{% codeblock lang:bash %}
        Load  "vnc"
{% endcodeblock %}

Add the following to Section "Screen"

{% codeblock lang:bash %}
        Option     "PasswordFile"    "/home/clientsales/.vnc/passwd"
{% endcodeblock %}

##Installing sound:

{% codeblock lang:bash %}
emerge alsa-utils
alsamixer to configure sound levels
alsaconf
rc-update add alsasound boot
{% endcodeblock %}

**Note:** Firefox 3.6 and newer does NOT work as below with the Java plugin.

##Installing the Firefox browser with all dependencies:

{% codeblock lang:bash %}
emerge mozilla-firefox
emerge -C mozilla-firefox
cd /opt
wget http://mirror.atratoip.net/mozilla/firefox/releases/3.5.7/linux-i686/is/firefox-3.5.7.tar.bz2
bzip2 firefox-3.5.7.tar.bz2
{% endcodeblock %}

**Installing Adobe Flash plugin:**

Download the tar.gz version from Adobe and copy the gz file to `/opt/`.

{% codeblock lang:bash %}
tar -zxvf install_flash_player_10_linux.tar
{% endcodeblock %}

move the .so file to `/opt/firefox/plugins/`

**Installing Sun Java Runtime Environment:**

{% codeblock lang:bash %}
emerge -C dev-java/icedtea6-bin
{% endcodeblock %}

Download Sun JRE bin file and install in `/opt/`

{% codeblock lang:bash %}
chmod +x jre-6u20-linux-i586.bin
./jre-6u20-linux-i586.bin
ln -s /opt/jre1.6.0_18/plugin/i386/ns7/libjavaplugin_oji.so /opt/firefox/plugins/libjavaplugin_oji.so
{% endcodeblock %}

**Adding environment variables for JAVA, lockdown and make X automatically start at boot for the clientsales user profile.**

{% codeblock lang:bash %}
su clientsales
{% endcodeblock %}

{% codeblock lang:bash %}
vi /home/clientsales/.bashrc (Add the following:)
{% endcodeblock %}

{% codeblock %}
export J2RE_HOME=/opt/jre1.6.0_18
export PATH=$J2RE_HOME/bin:$PATH
sudo /bin/stty intr undef
sudo /bin/stty kill undef
sudo /bin/stty quit undef
sudo /bin/stty susp undef
sudo startx &>/dev/null
{% endcodeblock %}

{% codeblock lang:bash %}
exit
{% endcodeblock %}

{% codeblock lang:bash %}
vi startx
{% endcodeblock %}

Paste the following code and save:

{% codeblock lang:bash %}
userclientrc=$HOME/.xinitrc
sysclientrc=/etc/X11/xinit/xinitrc

userserverrc=$HOME/.xserverrc
sysserverrc=/etc/X11/xinit/xserverrc
defaultclientargs=""
defaultserverargs="-nolisten tcp -br"
clientargs=""
serverargs=""

if [ -f $userclientrc ]; then
    defaultclientargs=$userclientrc
elif [ -f $sysclientrc ]; then
    defaultclientargs=$sysclientrc

fi
if [ -f $userserverrc ]; then
    defaultserverargs=$userserverrc
elif [ -f $sysserverrc ]; then
    defaultserverargs=$sysserverrc
fi

whoseargs="client"
while [ x"$1" != x ]; do
    case "$1" in
      /''*|\.*) if [ "$whoseargs" = "client" ]; then
                  if [ "x$clientargs" = x ]; then
                      clientargs="$1"
                  else
                      clientargs="$clientargs $1"
                  fi
              else
                  if [ "x$serverargs" = x ]; then
                      serverargs="$1"
                  else
                      serverargs="$serverargs $1"
                  fi
              fi ;;
      --) whoseargs="server" ;;
      *) if [ "$whoseargs" = "client" ]; then
                  if [ "x$clientargs" = x ]; then
                      clientargs="$defaultclientargs $1"
                  else
                      clientargs="$clientargs $1"
                  fi
              else
                  case "$1" in
                      :[0-9]*) display="$1"; serverargs="$serverargs $1";;
                      *) serverargs="$serverargs $1" ;;
                  esac
              fi ;;
    esac
    shift
done

if [ x"$clientargs" = x ]; then
 clientargs="$defaultclientargs"
fi
if [ x"$serverargs" = x ]; then
 serverargs="$defaultserverargs"
fi

if [ x"$XAUTHORITY" = x ]; then
    XAUTHORITY=$HOME/.Xauthority
    export XAUTHORITY
fi

removelist=

# set up default Xauth info for this machine
case `uname` in
Linux*)
 if [ -z "`hostname --version 2>&1 | grep GNU`" ]; then
  hostname=`hostname -f`
 else
  hostname=`hostname`
 fi
 ;;
*)
 hostname=`hostname`
 ;;
esac

authdisplay=${display:-:0}
mcookie=`/usr/bin/mcookie`
dummy=0

# create a file with auth information for the server. ':0' is a dummy.
xserverauthfile=$HOME/.serverauth.$$
xauth -q -f $xserverauthfile << EOF
add :$dummy . $mcookie
EOF
serverargs=${serverargs}" -auth "${xserverauthfile}

# now add the same credentials to the client authority file
# if '$displayname' already exists don't overwrite it as another
# server man need it. Add them to the '$xserverauthfile' instead.
for displayname in $authdisplay $hostname$authdisplay; do
     authcookie=`xauth list "$displayname" \
       | sed -n "s/.*$displayname[[:space:]*].*[[:space:]*]//p"` 2>/dev/null;
    if [ "z${authcookie}" = "z" ] ; then
        xauth -q << EOF
add $displayname . $mcookie
EOF
 removelist="$displayname $removelist"
    else
        dummy=$(($dummy+1));
        xauth -q -f $xserverauthfile << EOF
add :$dummy . $authcookie
EOF
    fi
done

cleanup() {
    [ -n "$PID" ] && kill $PID > /dev/null 2>&1

if [ x"$removelist" != x ]; then
    xauth remove $removelist
fi
if [ x"$xserverauthfile" != x ]; then
    rm -f $xserverauthfile
fi

if command -v deallocvt > /dev/null 2>&1; then
    deallocvt
fi
}

trap cleanup 0

xinit $clientargs -- $serverargs -deferglyphs 16 &

PID=$!

wait $PID

unset PID
{% endcodeblock %}

{% codeblock lang:bash %}
chmod +x startx
cp startx /usr/bin/startx
{% endcodeblock %}

{% codeblock lang:bash %}
vi xinitrc
{% endcodeblock %}

Paste the following code and save:

{% codeblock lang:bash %}
userresources=$HOME/.Xresources
usermodmap=$HOME/.Xmodmap
xinitdir=/etc/X11
sysresources=$xinitdir/Xresources
sysmodmap=$xinitdir/Xmodmap

# merge in defaults and keymaps

if [ -f $sysresources ]; then
    xrdb -merge $sysresources
fi

if [ -f $sysmodmap ]; then
    xmodmap $sysmodmap
fi

if [ -f $userresources ]; then
    xrdb -merge $userresources
fi

if [ -f $usermodmap ]; then
    xmodmap $usermodmap
fi

# First try ~/.xinitrc
if [ -f "$HOME/.xinitrc" ]; then
 XINITRC="$HOME/.xinitrc"
 if [ -x $XINITRC ]; then
  # if the x bit is set on .xinitrc
  # it means the xinitrc is not a
  # shell script but something else
  exec $XINITRC
 else
  exec /bin/sh "$HOME/.xinitrc"
 fi
# If not present, try the system default
elif [ -n "`/etc/X11/chooser.sh`" ]; then
 exec "`/etc/X11/chooser.sh`"
# Failsafe
else
 # start some nice programs
        #twm &
        #xclock -geometry 50x50-1+1 &
        #xterm -geometry 80x50+494+51 &
        #xterm -geometry 80x20+494-0 &
        #exec xterm -geometry 80x66+0+0 -name login
        exec /opt/firefox/firefox
fi
{% endcodeblock %}

{% codeblock lang:bash %}
chmod +x xinitrc
cp xinitrc /etc/X11/xinit/xinitrc
{% endcodeblock %}

**Adding some cron entries if needed:**

{% codeblock lang:bash %}
crontab -e
{% endcodeblock %}

**Reboot the system:**

{% codeblock lang:bash %}
init 6
{% endcodeblock %}

When the system returns from boot, first set all Firefox preferences. Not saving passwords, not using cookies, exc. Set the following URL as the default homepage: `file:///home/clientsales/index.html`

{% codeblock lang:bash %}
vi /home/clientsales/.mozilla/firefox/THIS_WILL_BE_DIFFERENT.default/localstore.rdf
{% endcodeblock %}

{% codeblock %}
sizemode="maximized"
width="1024"
height="768"
{% endcodeblock %}

{% codeblock lang:bash %}
vi /home/clientsales/.mozilla/firefox/THIS_WILL_BE_DIFFERENT.default/prefs.js
{% endcodeblock %}

Add the following 3 lines and save:

{% codeblock %}
user_pref("accessibility.browsewithcaret", false);
user_pref("accessibility.warn_on_browsewithcaret", false);
user_pref("browser.startup.homepage", "file:///home/clientsales/index.html");
{% endcodeblock %}

{% codeblock lang:bash %}
mkdir /home/clientsales/.startpage
mkdir /home/clientsales/.startpage/images
{% endcodeblock %}

{% codeblock lang:bash %}
vi /home/clientsales/.startpage/defaulthtml
{% endcodeblock %}

Paste the following code and save:

{% codeblock lang:html %}
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
  <title>Unconfigured System!</title>
  </head>
  <body bgcolor="#000000" text="#FFFFFF" link="#FFFFFF" vlink="#FFFFFF">
    <br>
    <br>
    <br>
    <br>
    <br>
    <br>
    <center><img src="./.startpage/images/stop.png" /></center>
    <br>
    <br>
    <center><font size="3"><b>This system needs to be configured.
    <br>
    <br>
    <font size="8">
    Network problem detected!
    </font>
    <br>
    <br>
    Please contact the HELPDESK for support!</b></b></font></center>
  </body>
</html>
{% endcodeblock %}

Save the 2 images below to `/home/clientsales/.startpage/images/` with names `exclam.png` and `stop.png`

{% img /images/blog_posts/stop.png %}
{% img /images/blog_posts/exclam.png %}

{% codeblock lang:bash %}
chown -R clientsales:clientsales /home/clientsales/
{% endcodeblock %}

{% codeblock lang:bash %}
vi /etc/prep_default_page.sh
{% endcodeblock %}

And paste the following code:

{% codeblock lang:bash %}
#!/bin/bash

##URL in prefs.js must be pointing to: file:////home/clientsales/index.html

ipaddr=`ifconfig |grep Bcast |sed 's/Bcast.*//' |sed 's/          inet addr://'`
if [ "$ipaddr" == "" ]
   then
      cat /home/clientsales/.startpage/defaulthtml > /home/clientsales/index.html
   else
      cat /home/clientsales/.startpage/defaulthtml |sed "s/Network problem detected!/IP Address: $ipaddr/" |sed 's/stop.png/exclam.png/' > /home/clientsales/index.html
fi
{% endcodeblock %}

{% codeblock lang:bash %}
chmod +x /etc/prep_default_page.sh
chown root:root /etc/prep_default_page.sh
{% endcodeblock %}

{% codeblock lang:bash %}
vi /etc/conf.d/local.start
{% endcodeblock %}

Add the following line:

{% codeblock lang:bash %}
/etc/prep_default_page.sh
{% endcodeblock %}

{% codeblock lang:bash %}
vi /etc/conf.d/rc
{% endcodeblock %}

Make the following two changes:

{% codeblock %}
RC_PARALLEL_STARTUP="yes"
RC_INTERACTIVE="no"
{% endcodeblock %}

##Cleaning up the system and make it smaller:

**Purge unused locales:**

{% codeblock %}
vi /etc/locale.gen
{% endcodeblock %}

Uncomment the following entries:

{% codeblock %}
en_US ISO-8859-1
en_US.UTF-8 UTF-8
{% endcodeblock %}

{% codeblock %}
locale-gen
{% endcodeblock %}

**The following will remove features not needed and gain space:**

**NOTE:** After these commands ran, kernel source, portage, man pages, exc will be removed. You will not be able to add more software to this system via emerge anymore.

{% codeblock lang:bash %}
rm -rf /home/clientsales/.serverauth.*
rm -rf /usr/portage/*
rm -rf /var/tmp/portage/*
rm -rf /usr/share/man/*
rm -rf /usr/src/linux-*
rm -rf /usr/share/portage/*
rm -rf /usr/share/man/*
rm -rf /var/tmp/*
rm -rf /usr/share/doc
rm -rf /var/log/*
rm -rf /var/db/pkg/*
rm -rf /root/.mozilla
rm -rf /usr/share/locale/a*
rm -rf /usr/share/locale/b*
rm -rf /usr/share/locale/c*
rm -rf /usr/share/locale/d*
rm -rf /usr/share/locale/f*
rm -rf /usr/share/locale/g*
rm -rf /usr/share/locale/h*
rm -rf /usr/share/locale/i*
rm -rf /usr/share/locale/j*
rm -rf /usr/share/locale/k*
rm -rf /usr/share/locale/m*
rm -rf /usr/share/locale/n*
rm -rf /usr/share/locale/o*
rm -rf /usr/share/locale/p*
rm -rf /usr/share/locale/q*
rm -rf /usr/share/locale/r*
rm -rf /usr/share/locale/s*
rm -rf /usr/share/locale/t*
rm -rf /usr/share/locale/v*
rm -rf /usr/share/locale/w*
rm -rf /usr/share/locale/x*
rm -rf /usr/share/locale/y*
rm -rf /usr/share/locale/z*
rm -rf /usr/share/locale/el*
rm -rf /usr/share/locale/eo*
rm -rf /usr/share/locale/es*
rm -rf /usr/share/locale/et*
rm -rf /usr/share/locale/lg*
rm -rf /usr/share/locale/li*
rm -rf /usr/share/locale/lt*
rm -rf /usr/share/locale/lv*
rm -rf /usr/share/locale/ug*
rm -rf /usr/share/locale/ur*
rm -rf /usr/share/locale/uz*
find / \( -iname ".serverauth.*" \) -delete
{% endcodeblock %}

**Give this build a version number:**

{% codeblock lang:bash %}
echo "1.000-2.6.31r6-i686" > /etc/build
{% endcodeblock %}

##Install Firefox kiosk mode plugin:

Install the Firefox r-kiosk plugin from [here](https://addons.mozilla.org/en-US/firefox/addon/1659).

Reboot your system!

This can now be packaged into a nice self-installer CD for rapid deployment to similar hardware.

**HINT:** The default homepage URL can be changed in batch with scripting. The URL and all other Firefox specific settings can be altered inside `/home/clientsales/.mozilla/firefox/-somenumber-/prefs.js`
