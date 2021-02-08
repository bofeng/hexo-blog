---
title: 'Run Python WSGI app with uWSGI'
date: 2019-03-10 14:38:13
tags: [python,uwsgi]
published: true
hideInList: false
feature: 
---
用uWSGI跑python的wsgi程序。
<!-- more -->


1，安装

```
pip install uWSGI
```

2，uWSGI可以直接运行跑http协议，相当于http server，可以在测试时使用：

```
uwsgi --http=:8080 --processes=3 --module=wsgi:application
```

3，运行uWSGI协议，相当于uWSGI server。此时Web Server (Apache / Nginx)需要和uWSGI server通讯来处理http request。Request -> Nginx -> uWSGI Server -> Nginx -> Client。

跑uWSGI Server的配置文件sample：

```
; In most of cases, you only need to change the first 2 or 3
[uwsgi]
; project home folder
project_folder = /home/project/abc
; virtualenv folder
virtualenv = venv
; uwsgi server listen on, either use unix sock or a socket
socket = %(project_folder)/.uwsgi.sock
; socket = 127.0.0.1:59090
; change to dir before app running
chdir = %(project_folder)
; run under user id, this is nobody
uid = 65534
; run under group id, this is nogroup
gid = 65534
; # of processes running by uwsgi server
processes = 15
; unlikely to change lines below
wsgi-file = wsgi.py
chmod-socket = 666
master = true
pidfile = .uwsgi.pid
max-requests = 5000
vhost = true
reload-mercy = 10
vacuum = true
limit-as = 512
harakiri = 180
post-buffering = 8192
die-on-term = true
daemonize = logs/uwsgi-daemon.log
; to enable log-route, make sure you have libpcre3-dev installed
logger = errlog file:%(project_folder)/logs/uwsgi-err.log
log-route = errlog (Traceback)|(HTTP/1.\d 500)
; when reach 10M log, create new
log-maxsize = 10485760
```

4，注意当用uWSGI配合virtualenv跑的时候，为了避免各种奇怪的问题（No Python Appication Found），最好保证uWSGI运行的python版本和virtualenv里的python版本一致，也就是说，不要全局安装uwsgi，而是在virtualenv里安装uwsgi再使用，这样可以保证uwsgi跑的python版本就是venv里的版本：
```
virtualenv venv
source venv/bin/activate
pip install uwsgi
uwsgi --http :80 --wsgi-file=wsgi.py
```

5，当uwsgi运行daemon时会生成pid文件，如果此时更新了代码，想要重新加载，可以使用：`uwsgi --reload pidfile.pid` 或者更简单的 `touch wsgi.py`

6，一个简单的wsgictl文件：

```
#!/usr/bin/env bash
if [ $# -eq 1 ]; then
    if [ $1 = "start" ]; then
        uwsgi --ini etc/uwsgi.ini
    elif [ $1 = "reload" ]; then
        uwsgi --reload .uwsgi.pid
    elif [ $1 = "restart" ]; then
        uwsgi --stop .uwsgi.pid
        sleep 3
        uwsgi --ini etc/uwsgi.ini
    elif [ $1 = "stop" ]; then
        uwsgi --stop .uwsgi.pid
    fi
else
    echo "Usage: uwsgictl <start | reload | stop>"
fi
```

#### Reference

1. [uWSGI Options](https://uwsgi-docs.readthedocs.io/en/latest/Options.html)
2. [Things need to know about uWSGI](https://uwsgi-docs.readthedocs.io/en/latest/ThingsToKnow.html)
3. [The system-wide python version linked to uwsgi needs to be the same as the one of the virtualenv](https://stackoverflow.com/questions/42086692/django-pyenv-uwsgi-modulenotfounderror-no-module-named-django)
4. [Django Nginx+uwsgi 安装配置](http://www.runoob.com/django/django-nginx-uwsgi.html)
5. [uWSGI Management](https://uwsgi-docs.readthedocs.io/en/latest/Management.html)
6. [uWSGI Config Sample](https://github.com/source-nerd/Python-flask-with-uwsgi-and-nginx/blob/master/uwsgi.ini)