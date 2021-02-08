---
title: 'memcached限制ip进行tcp连接'
date: 2010-06-30 13:11:26
tags: [memcached]
published: true
hideInList: false
feature: 
---
最近新增了一台服务器，想把apache那台的memcached分到这台新机器上，我杯具的有个错误认识：

我以为memcached -l 的listen参数是用来监听哪台机进行connection，比如apache的ip是10.0.0.1，memcached那台服务器的ip是10.0.0.2，开始我以为，如果只让memcached允许来自apache机器的connection，只要配置memcached -l 10.0.0.1就可以了，可是这样报错：
<!-- more -->


> bind(): Cannot assign requested address
> failed to listen

问题是-l参数就是绑定本机网卡的，根本不是来限制tcp的connection的，这个限制是要靠iptable来搞的。

先说下-l绑定本机网卡的问题，当使用-l 127.0.0.1运行md的时候，只有本机能connect上这个memcache；让使用-l 10.0.0.2运行的时候（这里假定10.0.0.2是md机器对外的ip），任何机器都可以连上这个memcache。同事有人指出，-l实际上就是在指定绑定机器的那个网卡的问题，比如当使用ifconfig的时候，显示信息类似如下：

```
eth0      Link encap:Ethernet  HWaddr 00:17:a4:3a:e8:56
inet addr:10.0.0.2  Bcast:10.0.0.255  Mask:255.255.255.0
inet6 addr: fe80::217:a4ff:fe3a:e856/64 Scope:Link
UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
RX packets:1635820 errors:0 dropped:0 overruns:0 frame:0
TX packets:20283 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1000
RX bytes:148717543 (141.8 MiB)  TX bytes:2281514 (2.1 MiB)
Interrupt:25

lo        Link encap:Local Loopback
inet addr:127.0.0.1  Mask:255.0.0.0
inet6 addr: ::1/128 Scope:Host
UP LOOPBACK RUNNING  MTU:16436  Metric:1
RX packets:17 errors:0 dropped:0 overruns:0 frame:0
TX packets:17 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:0
RX bytes:1526 (1.4 KiB)  TX bytes:1526 (1.4 KiB)
```

这样当使用-l 127.0.0.1的参数跑的时候是绑定lo，监听来自lo的连接，这样只有本机能connect；当使用-l 10.0.0.2的参数跑的时候是绑定eth0（对外ip），这样外面的任何机器都能connect。

当使用-l制定对外ip的时候，肯定不希望任何机器都能连接上我们服务器的memcached，希望只要指定ip的机器能连上memcached，比如这个例子中的apache（ip：10.0.0.1）和memcached本机（10.0.0.2），这个时候就是需要iptables的时候，先写一个规则文件iptable-rules，类似如下（这里假定11211是memcached跑的端口）：

```
*filter
-A INPUT -i lo -j ACCEPT
#38049
-A INPUT -p tcp -m tcp –dport 11211 -s 10.0.0.1 -j ACCEPT
-A INPUT -p tcp -m tcp –dport 11211 -s 10.0.0.2 -j ACCEPT
-A INPUT -p tcp -m tcp –dport 11211 -j REJECT
COMMIT
```

然后使用命令`iptables-restore < iptable-rules`应用这些规则。成功后使用`iptables -L`可以看到对端口的限制已经生效。
