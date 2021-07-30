---
title: 'Docker compose: flask + redis'
typora-root-url: ../../source
date: 2021-07-30 13:16:07
tags: [docker, flask, redis]
---

## File structure

4 files here:

* app.py (app needed)
* requirements.txt (app needed)
* Dockerfile
* docker-compose.yml



## App

File `requirements.txt`:

```
flask
redis
```



File `app.py`

```python
import os

from flask import Flask
import redis

app = Flask(__name__)

redis_host = os.environ.get("REDIS_HOST", "localhost")
rclient = redis.Redis(host=redis_host, port=6379)

@app.route('/')
def hello():
	rclient.incr("counter", 1)
	count = rclient.get("counter")
	return f"You visited this page for {count} times"
```



To connect to redis, we use the env variable `REDIS_HOST` here if it exists, otherwise, use `localhost`. In the `docker-compose.yml` below you will set the `REDIS_HOST` to be `redis-cache`, so within the same docker network, our application can connect to the redis container. (Docker will auto point the DNS "redis-cache" to the redis-container IP)



## Docker

File `Dockerfile`:

```dockerfile
FROM python:3.9-slim

RUN useradd -m webapp
WORKDIR /home/webapp
COPY app.py ./
COPY requirements.txt ./
RUN pip install -r requirements.txt
RUN rm requirements.txt
RUN chown -R webapp:webapp .
USER webapp

EXPOSE 5000

# make sure it listens on 0.0.0.0
ENTRYPOINT ["flask", "run", "--host", "0.0.0.0"]

```

File `docker-compose.yml`:

```yaml
services:
    redis-cache:
        image: redis
    app:
        build: .
        environment:
            REDIS_HOST: redis-cache
        ports:
            - "5000:5000"
        depends_on:
            - redis-cache
```

We set the env variable  `REDIS_HOST` to be `redis-cache`. 



## Run

Start:

```bash
$ docker-compose build
$ docker-compose up -d
```

Then visit our app at "http://127.0.0.1:5000"

Stop:

```bash
$ docker-compose down
```

