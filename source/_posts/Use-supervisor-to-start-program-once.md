---
title: Use supervisor to start program once on system boot
typora-root-url: ../../source
date: 2022-08-05 12:01:52
tags: [ubuntu,supervisor]
---



Ubuntu 22.04 disabled `rc.local` by default, although we can enable it again by run some commands, since it is deprecated, need to find other ways to run command on system boot.

Supervisor is an easy alternative way to start a long-running service on boot, but if you just want to run a command and quit, supervisor can do it too. For example, if you just want to run a script and quit on system boot, like echo the date, you can do something ike this:

```toml
[program:echodate]
command=sh -c "date && sleep 10"
directory=/tmp
autostart=true
autorestart=false
stdout_logfile=/tmp/info.log
stderr_logfile=/tmp/err.log
```

There are several key configs here:

* using `sh -c` to  run date and `sleep`. We need to put `sleep` here, otherwise, if the process quit too quickly, supervisor will be failed to get the running pid, and thought it failed to run, then it will keep trying to run the command several times, and at last still thought it failed to run the command.
* set `autostart` to `true`. This will run the command on system boot/reboot.
* set `autorestart` to `false`. Since we only want to run the command once on system boot/reboot, after it quits, we don't want supervisor to auto restart the process again, we need to set `autorestart` to `false`.

