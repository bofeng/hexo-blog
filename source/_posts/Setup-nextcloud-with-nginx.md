---
title: Setup nextcloud with nginx in ubuntu
typora-root-url: ../../source
date: 2021-09-02 17:46:02
tags: ["nextcloud", "nginx", "php", "ubuntu"]
---



## Problem

You want to setup NextCloud with Nginx. But Nextcloud doesn't officially support Nginx, its go-to server is Apache.

> Please note that webservers other than Apache 2.x are not officially supported.
>
> -- Nextcloud Doc



## Solution

To run with Nginx, you need to install the `php-fpm` first, then proxy all php requeste through the fpm unix domain socket.

### Step 1

```bash
apt update
apt install php7.2 php7.2-fpm php7.2-mysql php7.2-xml php7.2-zip php7.2-mbstring php7.2-gd php7.2-curl
```

Now go to `/etc/php/` folder, you should be able to see the "7.2" folder.



### Step 2

Change php-fpm running user:group. Go to `/etc/php/7.2/fpm/pool.d`, edit the "www.conf" file. Change the "user" and "group" to the user you want, e.g. "nobody", "nogroup".

Then restart fpm:

```bash
$ service php7.2-fpm restart
```



### Step 3

You could now check the fpm's running UDS at path: `/var/run/php/php-fpm.sock`. Change its permission if needed.



Now setup your nginx config:

```nginx
upstream nextcloud-handler {
    server unix:/var/run/php/php-fpm.sock;
}

server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name yourdomain.com;

    ssl_certificate /path/to/fullchain.cer;
    ssl_certificate_key /path/to/yourdomain.key;

    # HTTP response headers borrowed from Nextcloud `.htaccess`
    add_header Referrer-Policy                      "no-referrer"   always;
    add_header X-Content-Type-Options               "nosniff"       always;
    add_header X-Download-Options                   "noopen"        always;
    add_header X-Frame-Options                      "SAMEORIGIN"    always;
    add_header X-Permitted-Cross-Domain-Policies    "none"          always;
    add_header X-Robots-Tag                         "none"          always;
    add_header X-XSS-Protection                     "1; mode=block" always;
    add_header X-Frame-Options "SAMEORIGIN";
  
    fastcgi_hide_header X-Powered-By;

    #### config log file
    error_log /path/to/nextcloud_logs/nginx-err.log warn;
    access_log /path/to/nextcloud_logs/nginx-access-$year$month$day.log combined;
    #### end of config log file
  
    // if you only allow certain IPs to access
    allow 127.0.0.1;
    allow 67.65.102.103;
    deny all;
  
    client_max_body_size 1024M;
    fastcgi_buffers 64 4K;

    gzip off;

    root /path/to/nextcloud/folder;
  
    location = / {
        if ( $http_user_agent ~ ^DavClnt ) {
            return 302 /remote.php/webdav/$is_args$args;
        }
    }

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }
  
    location ^~ /.well-known {
        # The rules in this block are an adaptation of the rules
        # in `.htaccess` that concern `/.well-known`.

        location = /.well-known/carddav { return 301 /remote.php/dav/; }
        location = /.well-known/caldav  { return 301 /remote.php/dav/; }

        location /.well-known/acme-challenge    { try_files $uri $uri/ =404; }
        location /.well-known/pki-validation    { try_files $uri $uri/ =404; }

        # Let Nextcloud's API for `/.well-known` URIs handle all other
        # requests by passing them to the front-end controller.
        return 301 /index.php$request_uri;
    }
  
    location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)(?:$|/)  { return 404; }
    location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console)                { return 404; }
  
  	# if you want to change shared files' access
    # shared file in nextcloud will start with /index.php/s/
    # so here we add "allow all" for this path
    location ~ ^/index.php/s/ {
    		allow all;
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        set $path_info $fastcgi_path_info;

        try_files $fastcgi_script_name =404;

        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $path_info;
        fastcgi_param HTTPS on;

    		fastcgi_param modHeadersAvailable true;
    		fastcgi_pass nextcloud-handler;

    		fastcgi_intercept_errors on;
        fastcgi_request_buffering off;
    }
  
    location ~ \.php(?:$|/) {
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        set $path_info $fastcgi_path_info;

        try_files $fastcgi_script_name =404;

        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $path_info;
        fastcgi_param HTTPS on;

        fastcgi_param modHeadersAvailable true;
        fastcgi_pass nextcloud-handler;
        # uncomment the next line if you don't want to show "index.php" in the url
        # fastcgi_param front_controller_active true;  
        
        fastcgi_intercept_errors on;
        fastcgi_request_buffering off;
    }
  
    location ~ \.(?:css|js|svg|gif|png|jpg|ico)$ {
        try_files $uri /index.php$request_uri;
        expires 6M;         # Cache-Control policy borrowed from `.htaccess`
        access_log off;     # Optional: Don't log access to assets
    }

    location ~ \.woff2?$ {
        try_files $uri /index.php$request_uri;
        expires 7d;         # Cache-Control policy borrowed from `.htaccess`
        access_log off;     # Optional: Don't log access to assets
    }

    # Rule borrowed from `.htaccess`
    location /remote {
        return 301 /remote.php$request_uri;
    }

    location / {
        try_files $uri $uri/ /index.php$request_uri;
    }
}
```



## Others

After you config this and restart nginx, if nginx show 500 error but didn't log anywhere, to debug,  you can just go to your nextcloud folder, find the `index.php` file, then before each line where throw the 500 error, just add `echo` to print that error out in the webpage. For example:

```php
echo $ex;
OC_Template::printExceptionErrorPage($ex, 500);
```

Remember to remove these lines after you found the error and fixed it.



## Reference:

* https://docs.nextcloud.com/server/latest/admin_manual/installation/nginx.html
* https://github.com/nextcloud/server/issues/15708
