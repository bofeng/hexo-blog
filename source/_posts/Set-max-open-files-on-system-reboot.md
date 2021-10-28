---
title: Set max open files on system reboot
typora-root-url: ../../source
date: 2021-10-28 13:36:25
tags: ["ubuntu", "linux", "MongoDB"]
---

## Problem

MongoDB 4.4 shows a warning message that the "rlimit current value is 1024, suggest 64000". So I want to increase this limit and make sure this new limit being set on system reboot.



To view the current "rlimit" (max open files) value, we can use:

```bash
$ ulimit -Sa
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 31647
max locked memory       (kbytes, -l) 65536
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 31647
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

You can see the "open files" value is 1024.



## Solution

We can first kill the mongodb process, then use `ulimit -n 64000` to increase this number, then start mongodb again. This new value is only applied to this session. 



And if we want to auto increase this number on start mongodb on system reboot, we can do this:



1, Add start mongodb in `/etc/rc.local`:

```bash
sudo -H -u mongodb bash -c "/home/mongodb/mongodb4.4/bin/mongod -f /etc/mongodb44.conf"
```

This will start a shell and run the command under user "mongodb".



2, Set value in file `/etc/security/limits.conf`:

```
mongodb soft nofile 64000
mongodb hard nofile 64000
```

This will set "max open files" only for the user "mongodb"



3, Open file `/etc/pam.d/sudo`, add line:

```
session    required   pam_limits.so
```

The full file looks like this:

```
#%PAM-1.0

session    required   pam_env.so readenv=1 user_readenv=0
session    required   pam_env.so readenv=1 envfile=/etc/default/locale user_readenv=0
session    required   pam_limits.so
@include common-auth
@include common-account
@include common-session-noninteractive
```



This will make sure these new limits being read when we run the `sudo` command in `/etc/rc.local`



Now just restart your server, the mongod will be running with the new limit. Also, if you switch to `mongodb` user and run `ulimit -Sa`, you will see the new "open files" value is now 64000.

