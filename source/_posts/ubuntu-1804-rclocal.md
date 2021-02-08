---
title: 'Ubuntu 18.04 rc.local'
date: 2019-01-31 02:22:20
tags: [ubuntu,linux]
published: true
hideInList: false
feature: 
---
Ubuntu 18.04 removed the `/etc/rc.local` file, so if you still want to use it:
<!-- more -->


1, su
2, vim /etc/rc.local
```
/usr/bin/env bash

# your start-application code

exit 0
```
3, chmod 755 /etc/rc.local

Test it out: `reboot`.