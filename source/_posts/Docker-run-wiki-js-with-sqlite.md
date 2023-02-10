---
title: Docker run wiki.js with sqlite
typora-root-url: ../../source
date: 2023-02-10 11:56:02
tags: [wiki, docker]
---



## Problem

You want to run a simple wiki instance for your own or simple team use.



## Solution

Use [wiki.js](https://js.wiki/) and dockder:

### 1, Run wiki.js docker without mount volumn:

```bash
docker run -d \
    -p 6060:3000 \
    -e "DB_TYPE=sqlite" \
    -e "DB_FILEPATH=/wiki/db.sqlite" \
    ghcr.io/requarks/wiki:2
```

This step is just for letting wiki.js generate an empty sqlite database, so we can later map it for persistent saving.

Once the container is running, grab the container id, copy the generated db.sqlite to your local file system (we are using `/home/john/data/db.sqlite` for this example):

```bash
docker cp <container-id>:/wiki/db.sqlite /home/john/data/db.sqlite
```

### 2, Now stop the container:

```bash
docker stop <container-id>
```

### 3, Re-run the container, this time with the sqlite file mapping:

```bash
docker run -d \
    -p 6060:3000 \
    -e "DB_TYPE=sqlite" \
    -e "DB_FILEPATH=/wiki/db.sqlite" \
    -v /home/john/data/db.sqlite:/wiki/db.sqlite \
    ghcr.io/requarks/wiki:2
```

All done, now you can visit localhost:6060 to see the wiki.js page.



If you want to run it under your own domain, simply put Nginx to proxy your localhost 6060



## Reference

* https://docs.requarks.io/
* https://medium.com/@marius.dras/docker-host-wiki-js-sqlite-with-persistent-data-5afd24f34f2d
