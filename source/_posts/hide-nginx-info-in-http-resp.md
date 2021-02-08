---
title: 'Hide Nginx Info in HTTP Resp'
date: 2019-03-17 02:47:07
tags: [nginx]
published: true
hideInList: false
feature: 
---
使用Nginx的默认安装设置，当发起http请求时，response里有类似的信息：

<!-- more -->

```
Connection	keep-alive
Date	Sun, 17 Mar 2019 15:14:44 GMT
Server	nginx/1.14.0 (Ubuntu)
Transfer-Encoding	chunked
```

这里暴露了nginx的版本和操作系统，如果想屏蔽这部分信息，去修改`/etc/nginx/nginx.conf`把server_tokens更改为off：

```
http {
    ...
    server_tokens off;
    ...
}
```

然后重启nginx：`service nginx restart`，再次用http访问时，显示的信息是：
```
Connection	keep-alive
Date	Sun, 17 Mar 2019 15:18:32 GMT
Server	nginx
Transfer-Encoding	chunked
```

#### Reference

* [Tips and Tricks to secure Nginx](https://www.howtoforge.com/tips-and-tricks-to-secure-your-nginx-web-server/)

