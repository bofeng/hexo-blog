---
title: 'MacOS的Battery显示充电状态不对的问题'
date: 2020-08-12 23:51:27
tags: [macos]
published: true
hideInList: false
feature: 
isTop: false
---
已经第二次遇到这个问题了，MBP显示只剩1%的电了，就算插着充电器，也充不进电，而且电量一直在下降。就算重启也没用。
重置了电池的SMC之后就好了，方法：
1，关机
2，按住Ctrl + Shift + Option + Power Button 五秒
3，按Power Button开机
然后再充电，电池状态就显示正常了。