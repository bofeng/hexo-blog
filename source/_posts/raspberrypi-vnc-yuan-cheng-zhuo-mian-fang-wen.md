---
title: 'Raspberrypi + VNC远程桌面访问'
date: 2019-03-21 14:53:49
tags: [ raspi,vnc]
published: true
hideInList: false
feature: 
---
Raspberrypi只是主板部分，如果要显示，可以用HDMI外接显示器或者电视，也可以直接安装VNC，然后用电脑连远程桌面。
<!-- more -->

设置VNC：

```bash
$ sudo apt update
$ sudo apt install tightvncserver

# 设置密码，会让输入2次相同的密码，再加一次询问你是否需要设置一个view only的密码
# 可以根据自己的需要来设置，如果只想共享桌面给其他人看你的神操作，
# 那么可以设置一个view only的密码给他
$ vncpasswd

# 设置完成后，启动
$ vncserver
```

在输入vncserver之后，系统会为VNC服务分配一个数字，在通过其他设备连接时需要用到该数字。
回到自己的Mac或者Windows，第一步先现在VNC Viewer的软件：https://www.realvnc.com/en/connect/download/viewer/

打开软件后，在最上面的地址条输入VNC Server地址：

![](/post-images/1571684092177.png)

这里在Mac下这里可以直接用Bonjour地址 raspberrypi.local 或者你也可以用Raspberrypi的IP地址。注意地址后面有:1，是上一步系统为VNC分配的数字。

点击连接后，会让你输入刚才设置的密码，然后就可以看到Raspberrypi的桌面了。

![](/post-images/1571684106583.png)


如果不想下载上面的VNC Viewer的软件，或者在其他人的电脑上没有权限安装此软件，也可以在Chrome下安装Chrome Extension / App ：
https://chrome.google.com/webstore/detail/vnc%C2%AE-viewer-for-google-ch/iabmpiboiopbgfabjmgeedhcmjenhbla?hl=en

安装后打开，步骤跟上述VNC Viewer的操作一致。
