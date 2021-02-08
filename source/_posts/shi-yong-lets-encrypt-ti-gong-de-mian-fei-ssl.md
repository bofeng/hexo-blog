---
title: '使用Let''s Encrypt提供的免费SSL'
date: 2016-10-09 21:08:12
tags: [ssl]
published: true
hideInList: false
feature: 
---
Let's Encrypt提供免费的SSL已经有一段时间了，现在他们网站也推荐使用CertBot了，这里以Ubuntu + Apache为例:
<!-- more -->


#### 1)
Go to [https://certbot.eff.org/](https://certbot.eff.org/), 服务器选择Apache，操作系统Ubuntu，页面会显示安装的instruction.

#### 2)
根据提示用wget下载下来certbot，在当前目录户有一个可运行的程序 ```certbot-auto```

#### 3)
把```certbot-auto```拷贝到系统bin目录，以便之后调用 ```/usr/local/bin/certbot-auto```

#### 4)
指定domain生成ssl证书:

```bash
$ certbot-auto --apache -d example.com -d www.example.com
```

这行命令会读取Apache目录下的config文件，列出来所有能选的domain，如果```example.com```没有在这个list里也没有关系，点cancel还是会给这个domain生成证书

#### 5)
证书生成后，会保存在```/etc/letsencrypt/live/example.com```目录下，其中会有几个文件：cert.pem, chain.pem, fullchain.pem, privkey.pem

#### 6)
接下来就是配置Apache，关键的几段SSL代码：

```xml
<VirtualHost *:443>
    ServerName example.com
    ServerAlias www.example.com
    ...
    <IfModule mod_ssl.c>
         SSLEngine on
         SSLCertificateFile /etc/letsencrypt/live/example.com/cert.pem
         SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem
         Include /etc/letsencrypt/options-ssl-apache.conf
         SSLCertificateChainFile /etc/letsencrypt/live/example.com/chain.pem
     </IfModule>
     ...
</VirtualHost>
```

#### 7)
配置完后就可以直接访问 ```https://example.com```，已经生效可以使用了。

#### 8)
最后一步，Lets encrypt提供证书唯一的缺点是，证书有效期只有90天，所以需要快到时间的时候去renew，也可以用crontab来自动renew。首先运行下面命令来试试renew有没有问题：

```bash
$ certbot-auto renew --dry-run
```

当显示没有问题时，就可以设置crontab了，这里设置每2个月的1号自动运行renew script:

```bash
$ crontab -e
```

添加：

```
10 0 1 */2 * certbot-auto renew --quiet --no-self-upgrade
```
