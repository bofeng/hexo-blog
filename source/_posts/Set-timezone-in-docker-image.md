---
title: Set timezone in docker image
typora-root-url: ../../source
date: 2021-08-14 12:29:16
tags: [docker, timezone]
---



## Problem

You want to set timezone in your docker image.



## Solution

As long as your base image installed `tzdata`, you can set its timezone by using `ENV TZ` method. Some images don't have `tzdata` installed by default like Ubuntu, while others have, like Debian and CentOS.

So a general way is install tzdata first then set the TZ env var:

```dockerfile
RUN apt update && apt install tzdata -y
ENV TZ="America/New_York"
```

