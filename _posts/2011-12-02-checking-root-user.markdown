---
layout: post
title: Checking Root User
date: '2011-12-02 07:19:00'
tags:
- bash
---

```bash
if [ $(id -u) -ne $(id -u root) ]; then
    echo "sorry, you must run this script as root."
    exit 1
fi
```