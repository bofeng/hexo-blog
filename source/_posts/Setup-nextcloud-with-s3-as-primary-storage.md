---
title: Setup nextcloud with s3 as primary storage
typora-root-url: ../../source
date: 2022-12-05 16:33:39
tags: [nextcloud, s3]
---



## Problem

Setup nextcloud instance on your own server behind a reverse proxy (nginx) and use s3 as the primary storage



## Solution

### 1, Create nextcloud folder

This folder will be used for saving nextcloud files and logs, I use `/home/project/nextcloud` as example. Inside create 2 sub folders `data` and `logs`

### 2, Run nextcloud docker

Create a `rundocker.sh` shell:

```bash
#!/usr/bin/env bash
docker run -d \
    --restart unless-stopped \
    -p 5808:80 \
    -v /home/project/nextcloud/data:/var/www/html \
    nextcloud
```

Here we run the nextcloud container at port 5808, and map the `/home/project/nextcloud/data` folder to `/var/www/html` inside the container. Run this script:

```bash
$ bash rundocker.sh
```

### 3, Setup nginx:

nextcloud.conf

```nginx
server {
    listen 443 ssl;
    server_name drive.yourdomain.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    error_log /home/project/nextcloud/logs/nginx-err.log warn;
    access_log /home/project/nextcloud/logs/nginx-access-$year$month$day.log combined;

    add_header Strict-Transport-Security "max-age=63072000" always;

    location / {
        proxy_pass http://localhost:5808;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host  $host;
        proxy_set_header X-Forwarded-Port  $server_port;
    }

    location /.well-known/carddav {
        return 301 $scheme://$host/remote.php/dav;
    }

    location /.well-known/caldav {
        return 301 $scheme://$host/remote.php/dav;
    }

    client_max_body_size 0;
}
```

Load this config to `sites-enabled` and restart nginx

### 4, Change nextcloud config

`cd` to `/home/project/nextcloud/data/config` folder, find the `config.php` file, then add:

```php
  'trusted_proxies' =>
  array (
    0 => '172.16.0.0/12',
    1 => '172.17.0.0/12',
  ),
  'overwriteprotocol' => 'https',
  'objectstore' => array(
    'class' => '\OC\Files\ObjectStore\S3',
    'arguments' => array(
      'bucket' => 'bucket_name',
      'key' => 'key',
      'secret' => 'secret',
      'region' => 'us-east-1',
      'hostname' => 'wasabisys.com',
      'port' => '',
      'objectPrefix' => "urn:oid:",
      'autocreate' => true,
      'use_ssl' => true,
      // required for some non Amazon S3 implementations
      'use_path_style' => true,
      // required for older protocol versions
      'legacy_auth' => false
     )
   ),
```

Here the `trusted_proxies` is the docker network, the default network is 172.16, but you can see your nextcloud container's IP with command:

```bash
$ docker inspect container_id | grep IPAddr
```

If the IP address is "172.17.0.x", then you should add the 2nd line "172.17.0.0/12". If it is "172.18.0.x", then you should add the 18.

The `overwriteprotocol` is also important. The nextcloud container is running under plain "http", and since your domain is most likely running under "https://" like "https://drive.yourdomain.com", here we need to set "overwriteprotocol" to "https".

The `objectstore` part is the key to use s3 as nextcloud's primary storage. I am using wasabi as example, you can use any s3 compatible storage system, just change the bucket name, key, secret, region and hostname.

Lastly, save this `config.php` file and visit your custom domain https://drive.yourdomain.com, it should work now.

## Reference

* [nextcloud docker](https://github.com/docker-library/docs/blob/master/nextcloud/README.md)
* [nextcloud reverse proxy](https://docs.nextcloud.com/server/stable/admin_manual/configuration_server/reverse_proxy_configuration.html)

* [nextcloud: s3 primary storage](https://docs.nextcloud.com/server/latest/admin_manual/configuration_files/primary_storage.html)
