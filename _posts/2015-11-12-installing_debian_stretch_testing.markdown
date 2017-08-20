---
layout: post
title: Installing Debian Stretch (Testing)
date: '2015-11-12 02:20:00'
tags:
- linux
---

If you want to install Debian for your Linux distribution and you want a current desktop then you'd benefit from installing the testing branch.  The current stable branch, Jessie, does not have the latest versions of different desktops.  In my case, I want to use XFCE which only version 4.10 is available in Jessie.  If you use Stretch then you will find the latest version 4.12 is there.  This is necessary for me since the multi-monitor configuration in 4.12 is much better and supports extending the desktop rather than just mirroring it. 

Get the latest net install ISO for stretch from the mirror repository.  At the time of writing the version used was Alpha 4.  Download it here. 

#### Write the ISO to a USB stick by inserting it and using dd to write the ISO as root.

```
$ dd if=firmware-stretch-DI-alpha4-amd64-netinst.iso of=/dev/sdb bs=4M
```

The installer properly detected the hardware for the Lenovo X220 laptop used for the install.  Since the net installer includes non-free firmware it was able to find the Realtek wireless card for installation.  The majority of the installation went as expected.  During package installation there was a failure.  After the failure, start with installing the GRUB bootloader and complete the install.  At that point the computer booted to a console.  No XFCE desktop as I had hoped.  From the console I then had to configure the ethernet card for DHCP and install packages. 

#### Install Packages

```
$ apt-get install xfce4 network-manager-gnome lightdm
```

[http://cdimage.debian.org/cdimage/unofficial/non-free/cd-including-firmware/stretch_di_alpha4/amd64/iso-cd/firmware-stretch-DI-alpha4-amd64-netinst.iso](http://cdimage.debian.org/cdimage/unofficial/non-free/cd-including-firmware/stretch_di_alpha4/amd64/iso-cd/firmware-stretch-DI-alpha4-amd64-netinst.iso)