---
title: '🚁 Proxy Websocket Timeout '
date: 2019-12-12 00:55:43
tags: [websocket,nginx,apache]
published: true
hideInList: false
feature: 
---
之前在[这篇文章](https://bofeng.github.io/post/proxy-websocket-configuration-in-apache-and-nginx/)里提到用Nginx和Apache代理websocket的连接，一个项目在测试机上有人反映如果一分钟没有在ws上输入信息，连接就会断掉。

打log之后在client端显示是ws.onclose的事件被触发了，触发的event的code是1006，reason是""，搜了一圈发现是Nginx的配置问题。Nginx在proxy ws的时候，默认的read timeout是60秒，如果在60秒内ws上没有数据传输，Nginx就会把它断掉。

解决方法是在proxy的配置上添加timeout的设置：
```nginx
location /ws/ {
        proxy_pass "http://localhost:8765/ws/";
        proxy_http_version 1.1;
        proxy_read_timeout 1800;  # 添加
        proxy_send_timeout 1800;  # 添加
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
}
```

在另外Apache的机器上经测试没有这个1分钟超时的问题，因为[Apache默认的IdleTimeout是0](https://httpd.apache.org/docs/trunk/mod/mod_proxy_wstunnel.html#proxywebsocketidletimeout)，也就是永不超时，所以跑Apache代理ws的机器没有这个问题。

### Reference
* [https://github.com/websockets/ws/issues/1598#issuecomment-504794623](https://github.com/websockets/ws/issues/1598#issuecomment-504794623)