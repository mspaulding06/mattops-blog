---
layout: post
title: Configuring Tuned on Fedora 23
date: '2016-01-14 22:15:00'
tags:
- linux
---

Using the tuned service on Fedora is a great way to save power on laptops.  Typically, I have used the balanced settings to get a good balance between performance and power savings.  But today I learned that tuned can work in tandem with powertop to provide an even better configuration.  If you just want to use the balanced settings you can use the tuned commands to configure that.  First you'll need to download tuned and then enable the service. 

```
$ dnf install tuned 
$ systemctl enable tuned 
$ systemctl start tuned
```

Once you have installed the service then you can set your profile. 

```
$ tuned-adm profile balanced
```

For an even better configuration you can create one based on powertop.  Now before you create the profile, you should first calibrate powertop. 

```
$ powertop -c
```

Your computer's screen will turn off and it may take 10 to 20 minutes for the entire process to complete.  Once that is done then you can create a new profile from powertop. 

```
$ powertop2tuned custom
```

This command will create a new profile in the /etc/tuned/ directory based on your powertop configuration.  The thing is that many of the power settings in the generated configuration are commented out.  You will need to edit the configuration and uncomment all of these in order to get the best possible configuration. 

```
$ vim /etc/tuned/custom/tuned.conf
```

Once your changes have been made, enable your custom configuration and restart the tuned service and you're all set. 

```
$ tuned-adm profile custom
$ systemctl restart tuned
```