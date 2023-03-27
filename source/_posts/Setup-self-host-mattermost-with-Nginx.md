---
title: Setup self-host mattermost with Nginx
typora-root-url: ../../source
date: 2023-03-26 23:56:51
tags: [mattermost,nginx]
---



## Problem

You want to self host [mattermost](https://mattermost.com/) on your own server, run mattermost under http:// with simple docker command, then put it behind Nginx.



## Solution

You need to build the docker image and run it yourself. Assume you are at the `/home/project` folder:

1, Clone repo and config env:

```bash
git clone https://github.com/mattermost/docker
mv docker mattermost # now our root folder is /home/project/mattermost
mkdir logs   # for later config nginx
mkdir static # for later config nginx
cp env.example .env
```

Now open the `.env` file, at the first line, change `DOMAIN` to your own domain that you want to host mattermost, like `mm.yourdomain.com` (there are other configs but for minimal configuration, you don't need to change them).

2, Create the required directories and set their permissions.

```bash
mkdir -p ./volumes/app/mattermost/{config,data,logs,plugins,client/plugins,bleve-indexes}
sudo chown -R 2000:2000 ./volumes/app/mattermost
```

3, Since we are going to run mattermost just under http://, then use https in Nginx, nginx will proxy to mattermost's http server, we don't need to config the https cert for mattermost server, just need to do that in Nginx layer.

Now we can just do `docker-compose` and run, if `docker-compose` is not found, make sure install it first:

```bash
su
apt install docker-compose
```

Now we can create 2 scripts to start and stop the server:

1 ) start.sh

```bash
#!/usr/bin/env bash
docker-compose -f docker-compose.yml -f docker-compose.without-nginx.yml up -d
```

2 ) stop.sh

```bash
#!/usr/bin/env bash
docker-compose -f docker-compose.yml -f docker-compose.without-nginx.yml down
```

4, To start the service, just run `start.sh`, to stop it, use `stop.sh`. After run `start.sh`, now the mattermost server should be running at http://127.0.0.1:8065. You can use `docker ps` to confirm the docker image is running, like:

```bash
CONTAINER ID   IMAGE                                          COMMAND                       
80780ddaea53   mattermost/mattermost-enterprise-edition:7.1   "/entrypoint.sh matt…"   
```

5, Now it is just the easy part - config Nginx:

```nginx
upstream mm_backend {
   server 127.0.0.1:8065;
   keepalive 32;
}

proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=mattermost_cache:10m max_size=3g inactive=120m use_temp_path=off;


server{
    listen 443 ssl http2;
    server_name mm.yourdomain.com;

    http2_push_preload on; # Enable HTTP/2 Server Push

    ssl_early_data on;
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384';
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:50m;

    ssl_certificate /path/to/your/cert.pem;
    ssl_certificate_key /path/to/your/key.pem;

    add_header Strict-Transport-Security max-age=15768000;

    error_log /home/project/mattermost/logs/nginx-err.log warn;
    access_log /home/project/mattermost/logs/nginx-access-$year$month$day.log combined;

    root /home/project/mattermost/static;

    location ~ /api/v[0-9]+/(users/)?websocket$ {
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection "upgrade";
       client_max_body_size 100M;
       proxy_set_header Host $http_host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;
       proxy_set_header X-Frame-Options SAMEORIGIN;
       proxy_buffers 256 16k;
       proxy_buffer_size 16k;
       client_body_timeout 60;
       send_timeout 300;
       lingering_timeout 5;
       proxy_connect_timeout 90;
       proxy_send_timeout 300;
       proxy_read_timeout 90s;
       proxy_http_version 1.1;
       proxy_pass http://mm_backend;
   }

   location / {
       client_max_body_size 50M;
       proxy_set_header Connection "";
       proxy_set_header Host $http_host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;
       proxy_set_header X-Frame-Options SAMEORIGIN;
       proxy_buffers 256 16k;
       proxy_buffer_size 16k;
       proxy_read_timeout 600s;
       proxy_cache mattermost_cache;
       proxy_cache_revalidate on;
       proxy_cache_min_uses 2;
       proxy_cache_use_stale timeout;
       proxy_cache_lock on;
       proxy_http_version 1.1;
       proxy_pass http://mm_backend;
   }
}
```



All done! Restart Nginx, now you can go to https://mm.yourdomain.com to use mattermost. P.S, if you want to turn on notification, you need to go to go to **System Console > Environment > Push Notification Server > Enable Push Notifications**, then select **Use TPNS connection to send notifications to iOS and Android apps**.



## Reference

* [Install Mattermost via Docker](https://docs.mattermost.com/install/install-docker.html)
* [Configure NGINX as a proxy for Mattermost server](https://docs.mattermost.com/install/config-proxy-nginx.html)
* [Mobile push notifications](https://docs.mattermost.com/deploy/mobile-hpns.html)
