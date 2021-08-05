---
title: Init ubuntu server
typora-root-url: ../../source
date: 2021-08-04 15:21:35
tags: [ubuntu]
---

## Install common software

```bash
$ apt update && apt install vim acl nginx python3-pip build-essential python3-dev git gcc fail2ban socat supervisor -y
```



## Set timezone

```bash
$ timedatectl set-timezone "America/New_York"
```



## Add user

### Add user for login

```bash
$ useradd -m bofeng
$ cd /home/bofeng
$ su bofeng
$ mkdir .ssh
$ cd .ssh
$ vim authorized_keys  # add key to this file
$ chmod 640 authorized_keys

```

### Add user for run project

```bash
$ useradd -m project
```

### Add user to sudoers

```bash
$ adduser $USER sudo
```

### Set user's sudo password

```bash
$ su
$ passwd $USER
```



## Config sshd

```bash
vim /etc/ssh/sshd_config
# set:
Port 32200
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no
# restart
service ssh restart
```



## Config vim

```bash
$ mkdir ~/.vimbackup
$ cd ~ && wget https://gist.githubusercontent.com/bofeng/3a5a70bed544e7b63584471d53a59dbf/raw/91f5ac8bb1149d3fa4565594fae067abedf926cb/vim.vimrc && mv vim.vimrc .vimrc
```



## Config ufw

```bash
$ ufw allow 80
$ ufw allow 443
$ ufw allow 32200
$ ufw enable
$ ufw status
```



## Install docker

```bash
$ apt install apt-transport-https ca-certificates curl gnupg lsb-release
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
$ echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
$ apt update
$ apt install docker-ce docker-ce-cli containerd.io
$ apt install docker-compose
```



Solve problem for "docker.sock permission denied":

```bash
$ sudo usermod -aG docker $USER
$ sudo setfacl --modify user:$USER:rw /var/run/docker.sock
```



## Install redis

### Install with source code

```bash
$ wget https://download.redis.io/releases/redis-6.2.5.tar.gz
$ tar zxf redis-6.2.5.tar.gz
$ cd redis-6.2.5
$ make
$ make install
```

### Install with docker

```bash
$ su project
$ mkdir -p redis/16379
$ cd redis/16379
```

add docker-compose.yml:

```yaml
version: "3"
services:
  redis-cache:
    image: redis:6.2.5-alpine
    ports:
      - 127.0.0.1:16379:6379
    volumes:
      - ./data:/data
```

Connect via local installed `redis-cli`:

```bash
$ redis-cli -p 16379
```

Connect via docker `redis-cli`:

```bash
$ docker exec -it <container-id> redis-cli
```



## Add nginx config

### nginx.conf

```nginx
user project;
worker_processes auto;
worker_cpu_affinity auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 1024;
    multi_accept on;
}

http {
  	sendfile on;
  	tcp_nopush on;
  	types_hash_max_size 2048;
    server_tokens off;
    
  	# mime types
  	include /etc/nginx/mime.types;
  	default_type application/octet-stream;
  
  	# ssl
  	ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
  	ssl_prefer_server_ciphers on;
  
    # log
  	access_log /var/log/nginx/access.log;
  	error_log /var/log/nginx/error.log;
  
    # gzip
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_buffers 16 8k;
    gzip_http_version 1.1;
    gzip_types text/plain text/css application/json application/javascript application/x-javascript text/javascript text/xml application/xml application/rss+xml application/atom+xml application/rdf+xml image/svg+xml;

    # before include other sites' conf
    map $time_iso8601 $year {
        default '0000';
        "~^(\d{4})-(\d{2})-(\d{2})" $1;
    }
    map $time_iso8601 $month {
        default '00';
        "~^(\d{4})-(\d{2})-(\d{2})" $2;
    }
    map $time_iso8601 $day {
        default '00';
        "~^(\d{4})-(\d{2})-(\d{2})" $3;
    }
    # ...
    add_header Permissions-Policy "interest-cohort=()";
    add_header X-Xss-Protection "1; mode=block" always;
  
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

### site.conf:

```nginx
upstream mywebsite_server {
  server unix:/home/project/demo/.gunicorn.sock;
  server 127.0.0.1:8964;
  keepalive 128;
}

server {
    listen 80;
    listen 443;
    server_name mywebsite.com *.mywebsite.com;
    
    ssl_certificate /etc/cert/cloudflare/mywebsite.com/cert.pem;
    ssl_certificate_key /etc/cert/cloudflare/mywebsite.com/key.pem;
  
    # increase client request buffer, if need to upload files
    client_body_buffer_size 20M;
    client_max_body_size 55M;

    # increase upstream response buffer
    proxy_buffers 16 1M;
    proxy_buffer_size 1M;
  
    error_log /home/project/demo/logs/nginx-err.log warn;
    access_log /home/project/demo/logs/nginx-access-$year$month$day.log combined;
  
    location /static/  {
        alias /home/project/demo/static/;

        location ~* \.(eot|ttf|otf|woff|woff2)$ {
            add_header 'Access-Control-Allow-Origin' '*';
      			try_files $uri /static/404.html;
        }

        location ~* \.(?:ico|css|js|gif|jpe?g|png|svg)$ {
            expires 30d;
            try_files $uri /static/404.html;
        }
        try_files $uri /static/404.html;
    }

    location /favicon.ico {
        alias /home/project/demo/static/images/favicon.ico;
    }

    # websocket request
    location /ws/ {
        proxy_pass "http://localhost:8080/";
        proxy_http_version 1.1;
        proxy_read_timeout 1800;
        proxy_send_timeout 1800;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
    }

    # http request
    location / {
        proxy_http_version 1.1;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $host;
        proxy_set_header Connection "";
        proxy_redirect off;
        proxy_pass http://upstream_server;
    }
}
```



## Install Go

```bash
$ wget https://golang.org/dl/go1.16.6.linux-amd64.tar.gz
$ rm -rf /usr/local/go && tar -C /usr/local -xzf go1.16.6.linux-amd64.tar.gz
$ export PATH=$PATH:/usr/local/go/bin  # or add to $HOME/.bashrc
$ go version
```



## Disable ping

```bash
$ echo "net.ipv4.icmp_echo_ignore_all = 1" >> /etc/sysctl.conf && sysctl -p 
```

