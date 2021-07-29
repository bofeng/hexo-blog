---
title: 'Docker: get host ip from container'
typora-root-url: ../../source
date: 2021-07-28 23:14:18
tags: [docker]
---

## Host IP

To get the host's IP from a container:

* in `MacOS` and `Windows`, just use the host: `host.docker.internal`. If you ping this host, you will be able to see the host ip.
* in `Linux`, inside container, run `/sbin/ip route|awk '/default/ { print $3 }'` and usually this print ip: `172.17.0.1`



## docker-compose file

So in your `docker-compose.yml` file, if you want to connect to your host's redis:

```yaml
services:
	app:
		environment:
			REDIS_SERVER: ${DOCKER_HOST:-host.docker.internal}:6379
	
```

The `REDIS_SERVER` env var indicates that if the env var `DOCKER_HOST` exists, if it exists, use that value, otherwise, use string `host.docker.internal` .

So in Linux before we do docker-compose, we can set the `DOCKER_HOST` env var:

```bash
$ export DOCKER_HOST=172.17.0.1 && docker-compose
```

In MacOS and Windows, we don't need to set this variable, and can just do:

```bash
$ docker-compose
```



## Other: check docker network interface

* List all network interface: `docker network ls`
* Inspect a network interface: e.g.  `docker network inspect bridge`, `docker network inspect host`



## Reference

* [Dev.to: Docker Tip - How to use the host's IP Address inside a Docker container](https://dev.to/natterstefan/docker-tip-how-to-get-host-s-ip-address-inside-a-docker-container-5anh)
* [Docker.com: Inspect docker network bridge](https://docs.docker.com/network/network-tutorial-standalone/#use-user-defined-bridge-networks)
* [Stackoverflow.com: How to get the IP address of the docker host from inside a docker container](https://stackoverflow.com/questions/22944631/how-to-get-the-ip-address-of-the-docker-host-from-inside-a-docker-container)

