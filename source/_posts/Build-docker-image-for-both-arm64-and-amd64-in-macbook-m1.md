---
title: Build docker image for both arm64 and amd64 in macbook m1
typora-root-url: ../../source
date: 2021-08-07 23:58:07
tags: [macos, docker]
---



## Problem

You are using macbook with m1 chip, and you want to build docker image that can run under both arm64 and amd64.



If you don't do any special config, when you build an image in your macbook m1, then pull the image and run in Linux/amd64, it may fail to run or raise a warning message:

> WARNING: The requested image's platform (linux/arm64/v8) does not match the detected host platform (linux/amd64) and no specific platform was requested

If it just raises this warning message, then put the parameter `--platform` could solve it:

```bash
$ docker run --rm -p 3000:3000 --platform linux/arm64/v8 yourimage:1.0.0
```

Another solution is, when build your image, make it runnable on both arm64 and amd64 by default:



## Solution

Steps:

1. Open your file `~/.docker/config.json`, change the field `experimental` from `disabled` to `enabled`

   ```javascript
   {
     //...
     "experimental" : "enabled"
   }
   ```

2. Restart your "Docker Desktop for Mac"

3. Run `docker buildx create --use`, this will generate a builder that let you specify multilple docker platforms at once.

4. Run `docker buildx build --platform linux/amd64,linux/arm64 --push -t <tag_to_push> .`, this will generate 2 images and push it to your docker registry. PLEASE NOTICE, here you have to specify the `--push` and push it to your registry, for example:

   ```bash
   $ docker buildx build --platform linux/amd64,linux/arm64 --push -t registry.gitlab.com/myaccount/mydockerregistry/helloworld:1.0.0 .
   ```

5. All set, you can now run `docker run <tag>` on both macbook with m1 and on your linux server. It will first pull the image from your registry based on your CPU's arch, then run it. Since the image it pulled based on the CPU arch, you won't have any warning message on either platform.



## Reference:

* https://blog.jaimyn.dev/how-to-build-multi-architecture-docker-images-on-an-m1-mac/
