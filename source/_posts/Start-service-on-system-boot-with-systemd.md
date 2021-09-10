---
title: Start service on system boot with systemd
typora-root-url: ../../source
date: 2021-09-10 15:31:29
tags: [systemd, linux, ubuntu]
---



## systemd

`systemd` counld be used for running program on system boot/reboot.

常用的格式和字段：

### Unit

* Description
* Before
* After



### Service

* EnvironmentFile
* ExecStart
* ExecStop
* ExecReload
* ExecStartPre
* ExecStartPos
* ExecStopPost
* Type
* Restart



### Install

* WantsBy



## Examples

### Create redis.service

Here we will write a `redis.service` to start redis-server on system boot:

```ini
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



Then:

1, Copy this file to path `/lib/systemd/system/`

2, Start service with `systemctl start redis`. Make sure your redis config file set the `daemonize` to `no`, if you set to `yes`, in the later run `systemctl start` step, it will fail. Now your redis-server should be running, you could check its running status with `systemctl status redis`.

3, To stop it, run `systemctl stop redis`



If you changed the service file content, you need to reload it:

```bash
systemctl daemon-reload
```



To make it run on system boot, you need:

```bash
systemctl enable redis
```

This will create a symbol link to your file under `/etc/systemd/system/multi-user.target.wants/` folder, and after your system reboot, it will auto start this redis service.



### Other examples

In this `/etc/systemd/system/multi-user.target.wants` folder, there are some other service files we can learn as example:

#### ufw.service

```ini
[Unit]
Description=Uncomplicated firewall
Documentation=man:ufw(8)
DefaultDependencies=no
Before=network.target

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/lib/ufw/ufw-init start quiet
ExecStop=/lib/ufw/ufw-init stop

[Install]
WantedBy=multi-user.target
```



#### cron.service

```ini
[Unit]
Description=Regular background program processing daemon
Documentation=man:cron(8)
After=remote-fs.target nss-user-lookup.target

[Service]
EnvironmentFile=-/etc/default/cron
ExecStart=/usr/sbin/cron -f $EXTRA_OPTS
IgnoreSIGPIPE=false
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target
```



#### supervisor.service

```ini
[Unit]
Description=Supervisor process control system for UNIX
Documentation=http://supervisord.org
After=network.target

[Service]
ExecStart=/usr/bin/supervisord -n -c /etc/supervisor/supervisord.conf
ExecStop=/usr/bin/supervisorctl $OPTIONS shutdown
ExecReload=/usr/bin/supervisorctl -c /etc/supervisor/supervisord.conf $OPTIONS reload
KillMode=process
Restart=on-failure
RestartSec=50s

[Install]
WantedBy=multi-user.target
```



## Compare to supervisor

`supervisor` is a good tool to run program too. So I think if the service is the basic infra applications, like redis, mysql, mongodb, you could set it under `systemd`, b/c supervisor doesn't have a good "dependency" running order supported. Then in the application layer, you could just use `supervisor` which have more flexible settings for applications.



## Reference

* [Systemd 入门教程：命令篇](https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)
* [Systemd 入门教程：实战篇](https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-part-two.html)
* [Systemd 服务管理教程](https://cloud.tencent.com/developer/article/1516125)
* [Supervisor Configuration](http://supervisord.org/configuration.html)

