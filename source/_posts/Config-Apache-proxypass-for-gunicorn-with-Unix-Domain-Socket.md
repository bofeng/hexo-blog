---
title: Config Apache proxypass for gunicorn + Unix Domain Socket + https
typora-root-url: ../../source
date: 2021-03-25 20:50:24
tags: [apache,gunicorn,uds]
---



## Problem

This cost me half of a day to figure it out. If you are using Apache to Proxy to your backend Gunicorn server with Unix Domain Socket, you probably have searched how to config it and found something like this:

```apache
ProxyPass / unix:/tmp/helloHTTP.s|http://127.0.0.1/
ProxyPassReverse / unix:/tmp/helloHTTP.s|http://127.0.0.1/
```

It will not work!!!!!!!!



The problem is in the `ProxyPassReverse` line, when you are using the Unix Domain Socket for the `ProxyPass` line, the `ProxyPassReverse` line shouldn't be the same as `ProxyPass`, it should only catch the `http://` part, so really it should be:

```apache
ProxyPass / unix:/tmp/helloHTTP.s|http://127.0.0.1/
ProxyPassReverse / http://127.0.0.1/
```



The idea behind this is, `ProxyPass` line will rewrite the host to whatever you set ( `http://127.0.0.1` in the above example, but you can use `http://localhost` too), then send the request to the backend server, when your backend server processed the request and sent out the response, say a 303 See Other response, its header will contain something like `Location: http://127.0.0.1/otherpage`, now in order to return the correct header to user (user need to see the location to `https://yourdomain.com/otherpage`), the `ProxyPassReverse` line's value must match the host in `Location: http://127.0.0.1/otherpage`. In this case, it needs to be `http://127.0.0.1`, no need to put the extra `unix:/tmp...` before it, otherwise, Apache will fail to match to replace it with your website's real domain, then the user will get a redirect Location to `http://127.0.0.1/otherpage`!



## 2 Solutions

Once you know how it works, it really easy to config. There are 2 ways:

1) If  you want to preserve the host, i.e. you want your gunicorn server handler to get the real Host, not the rewritten one, then you can use the `ProxyPreserveHost On` directive. Since now the Host is not being rewritten (it keeps using your actual domain),  the `ProxyPassReverse`'s value must match your domain:

```apache
<VirtualHost *:443>
	ServerName yourdomain.com
	
	# return https to user, require headers.load module
	RequestHeader set X-Forwarded-Proto "https"
	
	ProxyPreserveHost On
	ProxyPass / unix:/home/project/htmleditorbuilder/.gunicorn.sock|http://localhost/
	ProxyPassReverse / http://yourdomain.com/
	# if with ProxyPreserveHost On, ProxyPassReverse should match the ServerName
	# the ending http:// part of ProxyPass line doesn't matter since we are not using it
	...
</VirtualHost>
```



2) If  you don't need to preserve the host, then the `ProxyPassReverse` 's value must match whatever you set at the end of `ProxyPass` line:

```apache
<VirtualHost *:443>
	ServerName yourdomain.com
	
	# return https to user, require headers.load module
	RequestHeader set X-Forwarded-Proto "https"
	
	ProxyPass / unix:/home/project/bizcanvas/.gunicorn.sock|http://localhost/
	ProxyPassReverse / http://localhost/
	# if with ProxyPreserveHost Off, ProxyPassReverse should match 
	# the http:// part in ProxyPass line, the ServerName doesn't matter
	...
</VirtualHost>
```



The key idea is, `ProxyPassReverse`'s value must match the Host returned by  your backend server!



## Other Config

Once you set this up, when your backend server handle request, in the response's `Server` header, it will show `gunicorn/20.0.4` -ish, you don't want to expose that for security reason. One way to hide it is just unset the header:

```apache
	# both require headers.load module
	RequestHeader set X-Forwarded-Proto "https"
	Header unset Server
	
	ProxyPass / unix:/home/projectfolder/.gunicorn.sock|http://localhost/
	ProxyPassReverse / http://localhost/
```

If you don't need to preserve the host, these 4 lines will work for most of cases.



## Use Nginx

Config this with Nginx is painless, no matter you are using HTTP or UDS, just set that in your backend upstream:

```nginx
upstream backend_server {
    server unix:/home/project/abc/.gunicorn.sock;
}

location / {
    proxy_pass http://backend_server;
}
```



## Reference:

* https://stackoverflow.com/questions/24351782/proxypassreverse-dropping-https
* https://httpd.apache.org/docs/current/mod/mod_proxy.html#proxypass

