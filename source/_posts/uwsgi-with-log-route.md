---
title: 'uWSGI with log-route'
date: 2019-03-19 14:53:10
tags: [uwsgi,logroute]
published: true
hideInList: false
feature: 
---
本来在Mac上配置了Nginx + uWSGI, uWSGI里根据[uWSGI官网的参考](https://uwsgi-docs.readthedocs.io/en/latest/Logging.html)，配置了`log-route` :
<!-- more -->


```
daemonize = logs/uwsgi-daemon.log
logger = errlog file:%(project_folder)/logs/uwsgi-err.log
log-route = errlog (Traceback)|(HTTP/1.\d 500)
```

这样一般的log都会放到uwsgi-daemon.log里，当log里含有`Traceback`或者`(HTTP/1.1 500)`的字符时，会被记录到`uwsgi-err.log`里。测试有效后，把同样的配置部署到正式机(Ubuntu 18.04)上，但是发现Traceback或者500的log仍然会被记录到uwsgi-daemon.log中。

比对uWSGI在本地和服务器上的版本，都是2.0.18；使用`uwsgi --plugins-list`列出enabled的plugins，本地和服务器上的版本也相同。

搜了好久找到这篇[Enabling internal routing in uWSGI](https://stackoverflow.com/questions/27624414/enabling-internal-routing-in-uwsgi) ，这个时候回去看服务器上uwsgi-daemon.log里，果然在启动之初有警告：

```
!!! no internal routing support, rebuild with pcre support !!!
```

这说明uwsgi启动时检测到log-route的设置，当时发现自己并不支持log-route里的正则匹配，所以提示需要`pcre`这个库的支持。

根据上面Stackoverflow文章的提示，需要先安装：
```
apt install libpcre3 libpcre3-dev -y
```

然后重新安装uWSGI。

1，什么是libpcre3-dev:

Old Perl 5 Compatible Regular Expression Library - development files

2，重新安装uWSGI时需要注意，不能使用 `pip install uWSGI`，否则pip还是会使用之前cache的版本。要让它不使用cache的版本，两个方法：
1）删除cached wheels： `rm -rf ~/.cache/pip/wheels`，然后使用`pip install uWSGI` 
2）强制pip不使用cache的版本：`pip install -I --no-cache-dir uwsgi`

#### TL;DR Solution

```
apt install libpcre3 libpcre3-dev -y
pip install -I --no-cache-dir uwsgi
```
