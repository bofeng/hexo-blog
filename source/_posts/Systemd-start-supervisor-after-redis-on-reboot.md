---
title: 'Systemd: start supervisor after redis on reboot'
typora-root-url: ../../source
date: 2022-08-10 13:27:22
tags: [systemd,ubuntu,supervisor,redis]
---



I am using _supervisor_ to run my web application on system boot, and one of the application is using _redis_, so that requires _redis_ start first.

_supervisor_ doesn't have a good way to support start dependency, but _systemd_ support this.

## Steps

0, Create your _redis_ start config file, e.g. `6379.conf`, and make sure the `daemonize` is set to `no`

1, Create redis systemd file `redis.service`:
```
[Unit]
Description=Redis
After=network.target

[Service]
User=redis
Group=redis
ExecStart=/usr/local/bin/redis-server /home/redis/6379.conf
ExecStop=kill -s HUP $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

2, Copy this file to path:
```bash
cp redis.service /lib/systemd/system/
```

3, Start redis with systemd:
```
systemctl start redis
```

4, Set start redis on system boot:
```
systemctl enable redis
```
If you changed the service file content, you need to reload it:
```
systemctl daemon-reload
```

5, Go to folder `/etc/systemd/system/multi-user.target.wants`, and find `supervisor.service`, under the `[Unit]` section, add line:
```
Requires=redis.service
```

All done. Now on the system reboot, _redis_ will start before _supervisor_, so if you applications are booted by _supervisor_, now they can safely connect to _redis_.
