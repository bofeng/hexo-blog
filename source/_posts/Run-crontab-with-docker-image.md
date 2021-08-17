---
title: Run crontab with docker image
typora-root-url: ../../source
date: 2021-08-17 15:08:23
tags: ["docker", "crontab"]
---



## Problem

You have put your application in docker image, and you have cron task to run.



## Solution

Put your crontab script to the image too, and run both from `docker-compose`.

1. Say you have a crontab task like this: `10 * * * * python3 /app/del_tmp_file.py`

2. Put those lines to a file like `doc/project.cron`

3. In `Dockerfile`:

   ```dockerfile
   COPY doc/project.cron /etc/cron.d/project.cron
   RUN chmod 0644 /etc/cron.d/project.cron && crontab /etc/cron.d/project.cron
   ```

   This will add your cron task to crontab

Now you already have put the crontab and application to your docker image. Edit `docker-compose` to start both:

```yaml
version: "3"

services:
  app:
    image: yourimage:latest
    entrypoint: ["./app.py"]
    ports:
      - "127.0.0.1:6767:6767"
    volumes:
      - ./etc:/home/app/etc
      - ./logs:/home/app/logs
      - ./public:/home/app/public
  cron:
    image: yourimage:latest
    user: root
    entrypoint: [ "cron", "-f" ]
    volumes:
      - ./etc:/home/app/etc
      - ./logs:/home/app/logs
```

The key is for run the crontab, make the entrypoint to be `["cron", "-f"]`.

