---
title: 'Nginx安装启动与配置'
date: 2019-03-09 14:28:19
tags: [nginx,web]
published: true
hideInList: false
feature: 
---
安装，启动与配置
<!-- more -->

## 1, 安装

#### Ubuntu 18.04

```
apt update
apt install nginx
```

配置文件在 `/etc/nginx/sites-enabled` 文件夹里。

**启动**：

1. Start: `service nginx start`
2. Stop: `service nginx stop`
3. Restart `service nginx restart`


#### Mac OS

```
brew install nginx
```

nginx会被安装到`/usr/local/etc/nginx`，为了跟ubuntu路径一致，可以创建一个软连接：`ln -s /usr/local/etc/nginx /etc/nginx`

配置文件放在 `/etc/nginx/servers` 的文件夹里。 

**启动**：

1. Start: `brew services start nginx`
2. Stop: `brew services stop nginx`
3. Restart: `brew services restart nginx`


## 2, 配置SSL

使用`acme.sh`可以用来生成Let's Encrypt提供的SSL证书。运行后有4个文件：

> Your cert is `/root/.acme.sh/abc.com/abc.com.cer`
> Your cert key is `/root/.acme.sh/abc.com/abc.com.key`
> The intermediate CA cert is `/root/.acme.sh/abc.com/ca.cer`
> And the full chain certs is: `/root/.acme.sh/abc.com/fullchain.cer`

配置nginx，只用使用到第4个和第2个：

```
server {
    ...
    ssl on;
    ssl_certificate /root/.acme.sh/abc.com/fullchain.cer;
    ssl_certificate_key /root/.acme.sh/abc.com/abc.com.key;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers HIGH:!aNULL:!MD5;
}
```


## 3, Example Config file

```
server {
    listen 80;
    server_name abc.com *.abc.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name abc.com *.abc.com;
    
    allow 10.0.0.1;
    allow 10.0.0.2;
    deny all;
    
    # ssl section
    ssl on;
    ssl_certificate /root/.acme.sh/abc.com/fullchain.cer;
    ssl_certificate_key /root/.acme.sh/abc.com/abc.com.key;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers HIGH:!aNULL:!MD5;
    # end ssl
    
    # gzip
    gzip on;
    gzip_http_version 1.1;
    gzip_vary on;
    gzip_comp_level 6;
    gzip_proxied any;
    gzip_types text/plain text/css application/json application/javascript application/x-javascript text/javascript text/xml application/xml application/rss+xml application/atom+xml application/rdf+xml;
    gzip_buffers 16 8k;
    # Disable gzip for certain browsers.
    gzip_disable "MSIE [1-6].(?!.*SV1)";
    # end gzip
    
    # log
    error_log /home/project/abc/logs/nginx-err.log warn;

    if ($time_iso8601 ~ "^(\d{4})-(\d{2})-(\d{2})") {
        set $year $1;
        set $month $2;
        set $day $3;
    }
    access_log /home/project/abc/logs/nginx-access-$year$month$day.log combined;
    # end log
    
    # alias
    location /static/  {
        alias /home/project/abc/static/;

        location ~* \.(eot|ttf|otf|woff|woff2)$ {
            add_header 'Access-Control-Allow-Origin' '*';
        }

        location ~* \.(?:ico|css|js|gif|jpe?g|png)$ {
            expires 30d;
        }
    }
    # end alias
    
    # uwsgi
    location / {
        include uwsgi_params;
        uwsgi_pass unix:/home/project/abc/.uwsgi.sock;
        # uwsgi_pass 127.0.0.1:59090;
        uwsgi_param Host $host;
        uwsgi_param X-Real-IP $remote_addr;
        uwsgi_param X-Forwarded-For $proxy_add_x_forwarded_for;
        uwsgi_param X-Forwarded-Proto $http_x_forwarded_proto;
    }
    # end uwsgi
}
```

Use Nginx as proxy: 访问`docs.abc.com/python`时，服务器会proxy到github页面，但浏览器端的URL保持不变。
```
server {
    listen 443 ssl;
    server_name docs.abc.com;

    root /home/project/docs;

    location /python/ {
        proxy_pass https://myproj.github.io/python-doc/;
    }
}
```


## 4, 配置Vim高亮Nginx配置语法

```
#!/bin/bash
#
# Highligh Nginx config file in Vim

# Download syntax highlight
mkdir -p ~/.vim/syntax/
wget http://www.vim.org/scripts/download_script.php?src_id=19394 -O ~/.vim/syntax/nginx.vim

# Set location of Nginx config file
cat > ~/.vim/filetype.vim <<EOF
au BufRead,BufNewFile /etc/nginx/*,/etc/nginx/conf.d/*,/usr/local/nginx/conf/* if &ft == '' | setfiletype nginx | endif
EOF
```

Credits to https://gist.github.com/ralavay/c4c7750795ccfd72c2db