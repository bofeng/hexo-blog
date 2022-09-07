---
title: Missing docker compose command
typora-root-url: ../../source
date: 2022-09-07 10:57:31
tags: [docker]
---



## Problem

After you installed `docker` and try to use its `docker compose` command, it shows the error like "docker doesn't have compose command"



## Solution

```bash
$ apt update
$ apt install docker-compose-plugin
```

Now you can use `docker compose`:

```bash
$ docker compose version
Docker Compose version v2.6.0
```



## Reference

* https://docs.docker.com/compose/install/linux/
