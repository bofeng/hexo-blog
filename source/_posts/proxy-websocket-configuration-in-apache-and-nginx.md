---
title: 'Proxy Websocket Configuration in Apache and Nginx'
date: 2019-12-06 14:53:23
tags: [webpy,apache,nginx]
published: true
hideInList: false
feature: 
---
Proxy websocket to backend application.

### Apache:
First enable `proxy_wstunnel.load` mod in mods-enabled folder:
```bash
cd /etc/apache2/mods-enabled
ln -s ../mods-available/proxy_wstunnel.load ./
```

Assume your backend websocket application running at port 8765, and the client side needs to connect to `ws://yourwebsite.com/ws/`.

In your site config file, add:

```apache
    ProxyRequests off
    ProxyPass /ws/ ws://localhost:8765/ retry=1
    ProxyPassReverse /ws/ ws://localhost:8765/

    RewriteEngine on
    RewriteCond %{HTTP:UPGRADE} ^WebSocket$ [NC]
    RewriteCond %{HTTP:CONNECTION} ^Upgrade$ [NC]
    RewriteRule /ws/.* ws://localhost:8765%{REQUEST_URI} [P]
```

### Nginx

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

server {
    listen 443 ssl;
    server_name yourwebsite.com

    # ...

    location /ws/ {
        proxy_pass "http://localhost:8765/ws/";
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
    }

    # ...
}
```

### Reference
* https://www.nginx.com/blog/websocket-nginx/
* https://stackoverflow.com/questions/12102110/nginx-to-reverse-proxy-websockets-and-enable-ssl-wss

