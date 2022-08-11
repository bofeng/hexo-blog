---
layout: post
title: Fix mongodb libcrypto error on ubuntu 22.04
date: 2022-08-11 12:33:27
tags: [mongodb,ubuntu]
---

## Problem

As the time of writing this post (08/11/2022), mongodb doesn't provide a release of mongod (communtiy editor) for ubuntu 22.04 LTS, if you download the 20.04 version and try to run under ubuntu 22.04, you will get an error:

> error while loading shared libraries: libcrypto.so.1.1: cannot open shared object file: No such file or directory

This is because ubuntu 22.04 is using libssl version 3.0, the libcrypto.so 1.1 is included in lib 1.1 which is not included in ubuntu 22.04 anymore:

```bash
$ openssl version
OpenSSL 3.0.2 15 Mar 2022 (Library: OpenSSL 3.0.2 15 Mar 2022)
$ ldconfig -p | grep crypto
	libk5crypto.so.3 (libc6,x86-64) => /lib/x86_64-linux-gnu/libk5crypto.so.3
	libcrypto.so.3 (libc6,x86-64) => /lib/x86_64-linux-gnu/libcrypto.so.3
	libbd_crypto.so.2 (libc6,x86-64) => /lib/x86_64-linux-gnu/libbd_crypto.so.2
```

From the above you can see the openssl version is 3, and libcrypto.so.1.1 is missing



## Solution

Install libssl 1.1 manually can solve this problem, go to page: http://nz2.archive.ubuntu.com/ubuntu/pool/main/o/openssl/?C=M;O=D to find the libssl 1.1. Find the latest version and download the `.deb` file, then use `apt install` to install it:

```bash
$ wget http://nz2.archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2.16_amd64.deb
$ apt install ./libssl1.1_1.1.1f-1ubuntu2.16_amd64.deb
```

Now although `openssl version` still print 3.0, the libssl and libcrypto1.1 has been installed to the system:

```bash
$ ldconfig -p | grep crypto
	libk5crypto.so.3 (libc6,x86-64) => /lib/x86_64-linux-gnu/libk5crypto.so.3
	libcrypto.so.3 (libc6,x86-64) => /lib/x86_64-linux-gnu/libcrypto.so.3
	libcrypto.so.1.1 (libc6,x86-64) => /lib/x86_64-linux-gnu/libcrypto.so.1.1
	libbd_crypto.so.2 (libc6,x86-64) => /lib/x86_64-linux-gnu/libbd_crypto.so.2
$ ldconfig -p | grep ssl
	libxmlsec1-openssl.so.1 (libc6,x86-64) => /lib/x86_64-linux-gnu/libxmlsec1-openssl.so.1
	libssl3.so (libc6,x86-64) => /lib/x86_64-linux-gnu/libssl3.so
	libssl.so.3 (libc6,x86-64) => /lib/x86_64-linux-gnu/libssl.so.3
	libssl.so.1.1 (libc6,x86-64) => /lib/x86_64-linux-gnu/libssl.so.1.1
```

Now you should be able to run `mongod`.



## Reference:

* https://www.mongodb.com/community/forums/t/installing-mongodb-over-ubuntu-22-04/159931/30
* https://stackoverflow.com/a/72633324
