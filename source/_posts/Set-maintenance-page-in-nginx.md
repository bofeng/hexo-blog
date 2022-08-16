---
title: Set maintenance page in nginx
typora-root-url: ../../source
date: 2022-08-16 17:10:58
tags: [nginx]
---



## Problem

When you need to stop the web application and do some maintenance, sometimes it may take a bit while so you want to show a custom maintenance page



## Solution

Config in your application's configuration in Nginx:

```nginx
root /path/to/project;

error_page 503 /maintenance.html;
location = /maintenance.html {
    internal;
}

location / {
    # if under maintenance
    # return 503;
    proxy_http_version 1.1;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto "https";
    proxy_set_header Host $http_host;
    proxy_set_header Connection "";
    proxy_redirect off;
    proxy_pass http://web_server;
}
```

You just need to put a "maintenance.html" page under the `/path/to/project` path, and when in maintenance, just comment out the line of `return 503;`. Then Nginx will return 503 for every request under `/` and it will map 503 to page "maintenance.html".



## Reference

* https://levelup.gitconnected.com/auto-maintenance-mode-with-nginx-71ef5c896851
* [nginx location internal](https://nginx.org/en/docs/http/ngx_http_core_module.html#internal)
