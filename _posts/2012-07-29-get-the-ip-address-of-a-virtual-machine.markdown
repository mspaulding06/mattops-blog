---
layout: post
title: Get the IP address of a Virtual Machine
date: '2012-07-29 05:16:00'
tags:
- python
- linux
---

<p>Often times when I&rsquo;m working I&rsquo;ll need to use multiple virtual machines, and I prefer to access them using SSH. The problem is that in order to do that, I would need to know the IP address of the VM. Of course you could always use the good old <i>arp</i> command, though there are a couple caveats. First, entries in the arp table don&rsquo;t stay there forever. If you started the VM a while ago, you may not be able to get the IP address. Second, if you have tons of entries in your arp table it can be hard to figure out which IP address corresponds to the VM you just started.

</p><p>To solve this problem I wrote a short script that will parse libvirt&rsquo;s dhcp leases file and print the IP addresses of all VMs along with their associated virtual machine names. Yay!

</p><pre>
<code>
#!/usr/bin/env python

import libvirt
import xml.dom.minidom
import re
import glob
import sys

def getMacIpMapping(filename):
    data = open(filename, "r").read().split("\n")
    macMap = {}
    for line in data:
        if line.strip():
            (_, mac, ip, _, _) = re.split("\s+", line)
            macMap[mac] = ip
    return macMap

conn = libvirt.openReadOnly(None)
if not conn:
    print "Failed to open connection to hypervisor"
    sys.exit(1)

domIDs = conn.listDomainsID()

nameTable = {}

for id in domIDs:
    dom = conn.lookupByID(id)
    xmldata = dom.XMLDesc(0)
    xmldom = xml.dom.minidom.parseString(xmldata)
    mac = xmldom.getElementsByTagName("mac")[0].getAttribute("address")
    name = xmldom.getElementsByTagName("name")[0].firstChild.nodeValue
    nameTable[mac] = name

macMap = {}

for f in glob.glob("/var/lib/libvirt/dnsmasq/*.leases"):
    macMap.update(getMacIpMapping(f))

for key in nameTable.keys():
    if key in macMap:
        print nameTable[key], "\t", macMap[key]

</code>
</pre>

<p>Here&rsquo;s an example run:

<img src="http://68.media.tumblr.com/tumblr_m7wowsOtlB1r3dq9z.png"/></p>