---
title: 'Fix RasperryPi apt install error: overwrite xxx but also in yyy'
typora-root-url: ../../source
date: 2021-03-29 16:47:18
tags: [linux,rasperrypi]
---

Try to run `apt -f install` on my Rasperry Pi 4 (Raspbian OS), but get following error:

```
Preparing to unpack .../gnome-software_3.30.6-5_armhf.deb ...
Unpacking gnome-software (3.30.6-5) ...
dpkg: error processing archive /var/cache/apt/archives/gnome-software_3.30.6-5_armhf.deb (--unpack):
 trying to overwrite '/usr/share/dbus-1/services/org.freedesktop.PackageKit.service', which is also in package pi-package-session 0.5
Errors were encountered while processing:
 /var/cache/apt/archives/gnome-software_3.30.6-5_armhf.deb
E: Sub-process /usr/bin/dpkg returned an error code (1)
```



The problem looks like it tries to overwrite some file but that file is being used by a package `pi-package-session`.

The solution is, remove that package first:

```bash
$ sudo dpkg -r pi-package
$ sudo dpkg -r pi-package-session
```



Now try `apt -f install` again, it should work now.