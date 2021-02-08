---
title: 'Config Postfix as a Send-Only SMTP Server on Ubuntu 18.04'
date: 2019-02-22 02:26:41
tags: [linux,ubuntu,mail,smtp]
published: true
hideInList: false
feature: 
---
每次新装服务器都忘了怎么安装这个mail sender的服务，记一下。
<!-- more -->


#### 1，安装postfix

```
$ apt update
$ hostnamectl set-hostname yourdomain.com
$ apt install mailutils postfix
```

上面这一步在安装postfix的时候会提示Postfix Configuration选项：

1，General type of mail configuration 选 Internet site
2，System mail name 里写域名就好

然后选择OK，安装完毕。

#### 2，修改postfix config file

```
$ vim /etc/postfix/main.cf
```

修改以下两行：
```
inet_interfaces = loopback-only
myhostname=yourdomain.com
```

保存后退出，重启postfix: 
```
$ systemctl restart postfix
```

#### 3，测试

```
$ echo "Send-Only Server" | mail -s "Mail Testing" to@yourmail.com
```