---
layout: post
title: "Building a secure Gentoo Linux kiosk system - part 1"
date: 2010-04-24
comments: true
categories: linux other
---
{% img left /images/blog_posts/gentoo.png %}

This is a two part guide to setup and configure a completely locked down kiosk Linux system that can be used in kiosk booths or as in my case as a client system for Call Centers with a web driven backend. Nice thing is that to have your agents work on different systems (Web Backends), you can update all workstations in batch to point to a new "URL" (backend system) that they need to work on. <!--more-->Agents can't fiddle with the system and break things... No Microsoft licensing costs... No viruses... Cheap hardware... The list goes on.
<br>

* [Download](http://www.gentoo.org/main/en/where.xml) and burn the minimal installation CD.
* Boot with the minimal live CD. (If it hangs at "Scanning... wd7000" reboot and boot with `gentoo noload=pata_qdi`)

##Preparing the live CD environment:
**Giving the live CD a root password:**

{% codeblock lang:bash %}
passwd root
{% endcodeblock %}

**Starting the SSH daemon and check system IP address:**
{% codeblock lang:bash %}
/etc/init.d/sshd start
ifconfig
{% endcodeblock %}

**Now you can SSH to the live CD environment and start the install process:**

{% codeblock lang:bash %}
ssh root@gentoo_ip_address
{% endcodeblock %}


##Configuring disk partitions and filesystems:

* Press p to print partition layout to see if all looks good.
* Press w to write the partition table.

{% codeblock lang:bash %}
fdisk /dev/hda
o
n
p
1
default cylinders
+128M
n
p
2
default cylinders
+2048M
n
p
3
default cylinders
default size
{% endcodeblock %}

**Applying the filesystems and activate the swap partition:**

{% codeblock lang:bash %}
mke2fs /dev/hda1
mke2fs -j /dev/hda3
mkswap /dev/hda2
swapon /dev/hda2
{% endcodeblock %}

Mounting the new partitions:

{% codeblock lang:bash %}
mount /dev/hda3 /mnt/gentoo
mkdir /mnt/gentoo/boot
mount /dev/hda1 /mnt/gentoo/boot
{% endcodeblock %}

##Getting stage3 and portage: (Substitute where stage filename differs)

{% codeblock lang:bash %}
cd /mnt/gentoo
wget ftp://ftp.is.co.za/linux/distributions/gentoo/releases/x86/current-stage3/stage3-i486-20100126.tar.bz2
tar xvjpf stage3-*.tar.bz2
wget ftp://ftp.is.co.za/linux/distributions/gentoo/releases/snapshots/current/portage-latest.tar.bz2
tar xvjf /mnt/gentoo/portage-latest.tar.bz2 -C /mnt/gentoo/usr
{% endcodeblock %}

##Configuring compile options:

{% codeblock lang:bash %}
vi /mnt/gentoo/etc/make.conf
{% endcodeblock %}

Add the following two lines and save the `make.conf` file.

{% codeblock lang:bash %}
FEATURES="ccache distlocks fixpackages parallel-fetch preserve-libs protect-owned sandbox sfperms strict unmerge-orphans userfetch"
INPUT_DEVICES="evdev"
{% endcodeblock %}

{% codeblock lang:bash %}
mkdir -p /mnt/gentoo/usr/lib/ccache/bin
{% endcodeblock %}

**Set the Gentoo mirrors and sync:**

{% codeblock lang:bash %}
mirrorselect -i -o >> /mnt/gentoo/etc/make.conf
mirrorselect -i -r -o >> /mnt/gentoo/etc/make.conf
{% endcodeblock %}

For local South African mirrors:

{% codeblock lang:bash %}
vi /mnt/gentoo/etc/make.conf
{% endcodeblock %}

* add `http://ftp.leg.uct.ac.za/pub/linux/gentoo` to GENTOO_MIRRORS
* replace `SYNC="rsync://ftp.leg.uct.ac.za/gentoo-portage"`

##DNS configuration:

{% codeblock lang:bash %}
cp -L /etc/resolv.conf /mnt/gentoo/etc/
{% endcodeblock %}

##Mounting filesystems and CHROOT into the newly created environment:

Mounting /proc and /dev filesystems:

{% codeblock lang:bash %}
mount -t proc none /mnt/gentoo/proc
mount -o bind /dev /mnt/gentoo/dev
{% endcodeblock %}

CHROOT into the newly created environment:

{% codeblock lang:bash %}
chroot /mnt/gentoo /bin/bash
env-update
source /etc/profile
export PS1="(chroot) $PS1"
{% endcodeblock %}

##Syncing portage:

{% codeblock lang:bash %}
emerge --sync
{% endcodeblock %}

Checking if `make.profile` link looks good:

{% codeblock lang:bash %}
ls -FGg /etc/make.profile
{% endcodeblock %}

##Adding custom USE FLAGS to make.conf:

{% codeblock lang:bash %}
vi /etc/make.conf
{% endcodeblock %}

And make the USE flags line look like this:

{% codeblock lang:bash %}
USE="server zlib nsplugin motif nptl -debug -pic -xcb -gnome -kde -qt3 -qt4 dbus hal nptl X xorg -dmx -ipv6 -kdrive -minimal -sdl -tslib ssl alsa oss midi jpeg png xulrunner nspr nss ntp caps unicode" (or what else you want or don't want between quotes)
{% endcodeblock %}

##Updating portage:

{% codeblock lang:bash %}
emerge portage
{% endcodeblock %}

##Setting the timezone:

{% codeblock lang:bash %}
cp /usr/share/zoneinfo/Africa/Johannesburg /etc/localtime
{% endcodeblock %}

##Compiling the kernel:

**Emerging the Gentoo kernel sources:**

{% codeblock lang:bash %}
emerge gentoo-sources
{% endcodeblock %}

Doing some manual kernel configuration: (NOTE: for kernel 2.6.31-r6)

{% codeblock lang:bash %}
cd /usr/src/linux

(If you're going to recompile your kernel, remember to make "make clean" first)

make menuconfig
{% endcodeblock %}

{% codeblock %}
Processor type and features
 [*] Support for old Pentium 5 / WinChip machine checks

File systems
 <*> Second extended fs support                                                                          
        [*]   Ext2 extended attributes                                                                             
 [*]   Ext2 POSIX Access Control Lists                  
 [*]   Ext2 Security Labels
 [*]   Ext2 execute in place support
File systems
 CD-ROM/DVD Filesystems  --->
  <*> UDF file system support
File systems
 DOS/FAT/NT Filesystems  ---> 
  <*> NTFS file system support
  [*] NTFS write support
File systems
 Network File Systems  --->
  <*> CIFS support (advanced network filesystem, SMBFS successor) 
  [*] CIFS statistics                                                                           
            [*] Extended statistics                                                        
             [*] Support legacy servers which use weaker LANMAN security                                      
            [*] Kerberos/SPNEGO advanced session setup                                                
           [*] CIFS extended attributes                                                          
            [*] CIFS POSIX Extensions
                                                
Device Drivers  ---> 
 <M> Sound card support  --->
  <M> Advanced Linux Sound Architecture  --->
                        <M> Sequencer support
                        <M> Sequencer dummy client                                                                            
                        <M> OSS Mixer API                                                                               
                        <M> OSS PCM (digital audio) API                                                                        
                        [*] OSS PCM (digital audio) API - Include plugin system                                             
                        [*] OSS Sequencer API                                                                                    
                        <M> HR-timer backend support                                                                             
                        [*] Use HR-timer as default sequencer timer
                        [ ] Support old ALSA API   
  PCI sound devices  --->
   <M> Intel/SiS/nVidia/AMD/ALi AC97 Controller
   <M> VIA 82C686A/B, 8233/8235 AC97 Controller
 Graphics support --->
            <*> /dev/agpgart (AGP Support) --->
   <*> ALI chipset support
   <*> ATI chipset support
   <*> NVIDIA nForce/nForce2 chipset support
   <*> VIA chipset support
  <*> Direct Rendering Manager (XFree86 4.1.0 and higher DRI support)  --->
   <*> ATI Radeon
   <*> Intel I810 
  -*- Support for frame buffer devices  --->
                        [*] Enable firmware EDID
   [ ] Enable Tile Blitting Support
   [*] VESA VGA graphics support
                        <*> nVidia Framebuffer Support
                         [*] Enable DDC Support
                        <*> Intel 810/815 support (EXPERIMENTAL)
   <*> Matrox acceleration
                        <*> ATI Radeon display support
                [ ] Bootup logo  --->
 Network device support  --->
  [*] Ethernet (10 or 100Mbit)  --->
   <*> 3c590/3c900 series (592/595/597) "Vortex/Boomerang" support
   <*> 3cr990 series "Typhoon" support
   <*> Broadcom 440x/47xx ethernet support
   [*] Support for older RTL-8129/8130 boards
   [*] Ethernet (1000 Mbit)  --->
   <*> Intel(R) 82575/82576 PCI-Express Gigabit Ethernet support
   <*> JMicron(R) PCI-Express Gigabit Ethernet support
   <*> Broadcom CNIC support

Bus options (PCI etc.)  ---> 
 [*] Enable deprecated pci_find_* API

Kernel hacking --->
 [*] Enable unused/obsolete exported symbols
 {% endcodeblock %}

**Compiling and installing the new kernel:**

{% codeblock lang:bash %}
make && make modules_install
cp arch/i386/boot/bzImage /boot/kernel-2.6.31-gentoo-r6
{% endcodeblock %}

If you have kernel modules that you want to load automatically, follow [this](http://www.gentoo.org/doc/en/handbook/handbook-x86.xml?part=1&chap=7#kernel_modules) documentation.

##Creating new fstab and configuring mount points at boot:

Note that mount points must be defined as sda although your harddrive is hda.
The new kernels does not recognize hda anymore.

{% codeblock lang:bash %}
vi /etc/fstab
{% endcodeblock %}

{% codeblock lang:bash %}
/dev/sda1               /boot           ext2            noauto,noatime          1 2
/dev/sda2               none            swap            sw                      0 0
/dev/sda3               /               ext3            noatime                 0 1
/dev/cdrom              /mnt/cdrom      auto            noauto,user             0 0
#/dev/fd0               /mnt/floppy     auto            noauto                  0 0
proc                    /proc           proc            defaults                0 0
shm                     /dev/shm        tmpfs           nodev,nosuid,noexec     0 0
{% endcodeblock %}

##Configuring the network settings:

{% codeblock lang:bash %}
vi /etc/conf.d/hostname
{% endcodeblock %}

{% codeblock lang:bash %}
HOSTNAME="your_preferred_FQDN_hostname"
{% endcodeblock %}

{% codeblock lang:bash %}
vi /etc/conf.d/net.eth0
{% endcodeblock %}

{% codeblock lang:bash %}
dns_domain_lo="your_preferred_domain_name"
#config_eth0=( "192.168.0.2 netmask 255.255.255.0 brd 192.168.0.255" )
#routes_eth0=( "default via 192.168.0.1" )
config_eth0=( "dhcp" )
{% endcodeblock %}

**Adding the networking config to the default runlevel:**

{% codeblock lang:bash %}
rc-update add net.eth0 default
{% endcodeblock %}

**Configure the hosts file:**

{% codeblock lang:bash %}
vi /etc/hosts
{% endcodeblock %}

##Set the root password:

{% codeblock lang:bash %}
passwd root
{% endcodeblock %}

##Installing essential system tools:

{% codeblock lang:bash %}
emerge syslog-ng
rc-update add syslog-ng default
emerge logrotate
emerge vixie-cron
rc-update add vixie-cron default
emerge jfsutils
emerge dhcpcd
emerge net-misc/ntp
{% endcodeblock %}

##Configuring the clock and NTP:

**Configure the clock:**

{% codeblock lang:bash %}
vi /etc/conf.d/clock
{% endcodeblock %}

**Setting the timezone:**
{% codeblock lang:bash %}
CLOCK="local"
TIMEZONE="Johannesburg"
{% endcodeblock %}

**Configuring NTP:**
{% codeblock lang:bash %}
vi /etc/ntp.conf
{% endcodeblock %}

{% codeblock lang:bash %}
server ntp.time.za.net
{% endcodeblock %}

Adding the NTP daemon to the default runlevel:

{% codeblock lang:bash %}
rc-update add ntpd default
{% endcodeblock %}

{% codeblock lang:bash %}
rm /etc/adjtime
{% endcodeblock %}

**NOTE:** date 012514262010 (for 14:26PM 2010-01-25) Format is: MMDDhhmm[[CC]YY][.ss]

{% codeblock lang:bash %}
hwclock --local --systohc
cd /
touch currtime
find . -cnewer /currtime -exec touch {} \; (Don't worry about errors)
rm -rf /currtime

rc-update add sshd default
rc-update del netmount

emerge mingetty
emerge sudo
{% endcodeblock %}

##Configure the bootloader:

{% codeblock lang:bash %}
emerge grub
{% endcodeblock %}

{% codeblock lang:bash %}
vi /boot/grub/grub.conf
{% endcodeblock %}

{% codeblock lang:bash %}
default 0
timeout 1
#splashimage=(hd0,0)/boot/grub/splash.xpm.gz
title Gentoo Linux 2.6.31-r6
root (hd0,0)
kernel /boot/kernel-2.6.31-gentoo-r6 root=/dev/sda3
{% endcodeblock %}

{% codeblock lang:bash %}
grep -v rootfs /proc/mounts > /etc/mtab
grub-install --no-floppy /dev/hda
{% endcodeblock %}

If you want smaller fonts in the CLI:

{% codeblock lang:bash %}
vi /etc/rc.conf
{% endcodeblock %}

{% codeblock lang:bash %}
CONSOLEFONT="default8x9"
{% endcodeblock %}

##Exit CHROOT, umounting mount points and rebooting into the new system:

{% codeblock lang:bash %}
exit
cd
umount /mnt/gentoo/boot /mnt/gentoo/dev /mnt/gentoo/proc /mnt/gentoo
reboot
{% endcodeblock %}

Follow [part 2](/blog/2010/04/24/building-a-secure-gentoo-linux-kiosk-system-part-2/) for the rest of the setup.
