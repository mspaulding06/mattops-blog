---
layout: post
title: Configuring PXE with Multiple DHCP Servers
date: '2016-01-15 21:19:00'
tags:
- devops
- networking
---

Configuring a PXE boot server can be a complicated task.  It requires a number of different services to be running in order for PXE to be useful, such as an ftp server, tftp server, and dhcp server.  One other complication that can arise happens when one is attempting to run a PXE boot server when there is already another dhcp server on the network.  In my case this is my home router.  After settings up a PXE boot server I found that the "next server" being used when using PXE boot was in fact my router and not the PXE boot server I had configured on the network.  Why did this happen?  It turns out that the router's firmware does not disable PXE boot support by default and the firmware does not support turning the feature off.  After doing some digging I found that it is possible to disable PXE boot by adding two configuration lines to the dnsmasq configuration for the router. 

```
dhcp-vendorclass=pxestuff,PXEClient
dhcp-ignore=pxestuff
```

So first in order to modify the router's configuration I had to install the latest DD-WRT router firmware which allows modification of the dnsmasq configuration.  Under the services tab there is a box to enter additional dnsmasq options.  It worked! 

Sources: 

[http://lists.thekelleys.org.uk/pipermail/dnsmasq-discuss/2011q1/004819.html](http://lists.thekelleys.org.uk/pipermail/dnsmasq-discuss/2011q1/004819.html)