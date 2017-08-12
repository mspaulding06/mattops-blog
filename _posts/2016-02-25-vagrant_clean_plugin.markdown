---
layout: post
title: Vagrant Clean Plugin
date: '2016-02-25 01:21:00'
---

Decided to create a Vagrant plugin to solve a problem I've been having.  When using Vagrant I find that I leave resources running that I forget about.  This plugin solves the problem by cleaning up all running Vagrant resources.  Just run `vagrant clean` and all running resources on all providers will be destroyed.

#### Installing the plugin

```
$ vagrant plugin install vagrant-clean
```

So far the plugin has been tested and working on Linux (Fedora 23) and Mac OSX.  It's just the first version, there are no tests and little documentation.  I plan on adding that in the future.  So just a warning, your mileage may vary here.