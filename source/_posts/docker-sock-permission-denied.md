---
title: docker.sock permission denied
typora-root-url: ../../source
date: 2021-07-28 23:05:18
tags: [docker]
---



After install docker in linux, when you try to run docker container under a normal user, like:

```bash
$ docker run --rm -it debian:latest
```

You might have the error like this:

```
dial unix /var/run/docker.sock: connect: permissoin denied
```



To solve it, just do:

```bash
$ sudo usermod -aG docker $USER
$ sudo setfacl --modify user:$USER:rw /var/run/docker.sock
```



Then you should be able to run the `docker run` successfully.
