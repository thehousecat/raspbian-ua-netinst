raspbian-ua-netinst
===================

The minimal Raspbian unattended netinstaller.

This project provides [Raspbian][1] power users the possibility to install a minimal base system unattended using latest Raspbian packages regardless when the installer was built.

The installer with default settings configures eth0 with DHCP to get Internet connectivity and completely wipes the SD card from any previous installation.

There are different kinds of "presets" that define the default packages that are going to be installed. Currently, the default one is called _server_ which installs only the essential base system packages including _OpenNTPd_ and _OpenSSH_ to provide a sane minimal base system that you can immediately after install ssh in and continue installing your software.

Other presets include _minimal_ which has even less packages (no logging, no text editor, no cron) and _base_ which doesn't even have networking. You can customize the installed packages by adding a small configuration file to your SD card before booting up.

Features
--------
 - completely unattended, you only need working Internet connection through the Ethernet port
 - DHCP and static ip configuration (DHCP is the default)
 - always installs the latest version of Raspbian
 - configurable default settings
 - installation takes well under **15 minutes** with fast Internet from power on to sshd running
 - fits in 512MB SD card
 - default install includes fake-hwclock to save time on shutdown
 - default install includes OpenNTPd to keep time
 - /tmp is mounted as tmpfs to improve speed
 - no clutter included, you only get the bare essential packages

Requirements
------------
 - a Raspberry Pi Model B
 - SD card of at least 512MB
 - working Ethernet with Internet connectivity

Obtaining prebuilt installer image
----------------------------------
Latest prebuilt image is **16MB** xz compressed and **32MB** uncompressed. It contains a single vfat filesystem with essential firmware files and the latest installer.

http://hifi.iki.fi/raspbian-ua-netinst/raspbian-ua-netinst-latest.img.gz

To flash your SD card on Linux:

    xzcat /path/to/raspbian-ua-netinst-latest.img.gz > /dev/sdX

Replace _/dev/sdX_ with the real path to your SD card.

Installing
----------
In normal circumstances, you can just power on your Pi and cross your fingers.

If you don't have a display attached you can monitor the Ethernet card leds to guess activity. When it finally reboots after installing everything you will see them going out and on a few times when Raspbian configures it on boot.

If you do have a display, you can follow the progress and catch any possible errors in the default configuration or your own modifications.

First boot
----------
The system is almost completely unconfigured on first boot. Here are some tasks you most definitely want to do on first boot.

The default **root** password is **raspbian**.

> Configure your default locale: `dpkg-reconfigure locales`  
> Configure your timezone: `dpkg-reconfigure tzdata`  
> Set new root password: `passwd`  

Reinstalling or replacing an existing system
--------------------------------------------
If you want to reinstall with the same settings you did your first install you can just move the original _config.txt_ back and reboot. Make sure you still have _kernel_emergency.img_ and _installer.cpio.gz_ in your _/boot_ partition. If you are replacing your existing system which was not installed using this method, make sure you copy those two files in and the installer _config.txt_ from the original image.

    mv /boot/config-reinstall.txt /boot/config.txt
    reboot

**Remember to backup all your data and original config.txt before doing this!**

Installer customization
-----------------------
While defaults should work for most power users, some might want to customize default configuration or the package set even further. The installer provides support for this by reading a configuration file _installer-config.txt_ from the first vfat partition. The configuration file is read in as a shell script so you can abuse that fact if you so want to.

Easiest way to do this is to first _xzcat_ the image to your SD card and then mount the first partition to add your configuration file.

The format of the file and the current defaults:

    preset=server
    packages= # comma separated list of extra packages
    mirror=http://mirrordirector.raspbian.org/raspbian/
    release=wheezy
    hostname=pi
    rootpw=raspbian
    cdebootstrap_cmdline=
    bootsize=+50M # /boot partition size as given to _fdisk_
    timeserver=time.nist.gov
    ip_addr=dhcp
    ip_netmask=0.0.0.0
    ip_broadcast=0.0.0.0
    ip_gateway=0.0.0.0
    ip_nameservers=
    online_config= # URL to extra config that will be executed after installer-config.txt

All of the configuration options should be clear. You can override any of these in your _installer-config.txt_. The time server is only used during installation and is for _rdate_ which doesn't support the NTP protocol.

Available presets: _server_, _minimal_ and _base_.

Presets set the `cdebootstrap_cmdline` variable. For example, the current _server_ default is:

> _--flavour=minimal --include=fake-hwclock,ifupdown,net-tools,isc-dhcp-client,openntpd,openssh-server,vim-tiny,iputils-ping,wget,ca-certificates,rsyslog,dialog,locales,less,man-db_

There's also a post-install script support which is executed just before unmounting the filesystems. You can use it to tweak and finalize your automatic installation. The script should reside in the first vfat partition and have a name of _post-install.txt_. See `scripts/etc/init.d/rcS` for more details what kind of environment your script will be run in.

Disclaimer
----------
I take no responsibility for ANY data loss. You will be reflashing your SD card anyway so it should be very clear to you what you are doing and will lose all your data on the card. Same goes for reinstallation.

See LICENSE for license information.

  [1]: http://www.raspbian.org/ "Raspbian"