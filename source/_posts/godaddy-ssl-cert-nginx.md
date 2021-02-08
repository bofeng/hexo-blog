---
title: 'Godaddy SSL Cert + Nginx'
date: 2019-03-31 14:57:44
tags: [nginx,godaddy]
published: true
hideInList: false
feature: 
isTop: false
---
When download SSL certificate from Godaddy, you will have 3 files like:
* gd_bundle-g2-g1.crt
* c90e3e53c899df65.crt
* c90e3e53c899df65.pem
<!-- more -->

The content in `c90e3e53c899df65.pem` and `c90e3e53c899df65.crt` are the same (not sure why Godaddy gave same file with different extentions), remember the `.pem` file is NOT your Certificate Key file, only YOU have the your own key file! So if you are updating SSL cert for your website, YOU DON'T NEED TO CHANGE THE CERTIFICATE KEY FILE (unless you want to generate a totally new certificate from godaddy). 

Now let's handle those 2 `.crt` files. To config it properly with Nginx, first combine those 2 files:

```bash
cat c90e3e53c899df65.crt gd_bundle-g2-g1.crt >> mysite_bundle.crt
```

Then use it in Nginx:

```
ssl_certificate /path/to/mysite_bundle.crt;
ssl_certificate_key /path/to/mysite.key;
```

If you are using Apache, it will be:
```
SSLCertificateFile /path/to/mysite_bundle.crt
SSLCertificateKeyFile /path/to/mysite.key
```

If you don't do the combination and just use `c90e3e53c899df65.crt` as the site certificate file, then when you browse the website, it is still good, but when you use python to request url in this site, it will throw an exception like SSL cert is invalid. If you combine those 2 crt files first then use the combined file, the python exception will be gone.