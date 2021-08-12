---
title: Get client real ip when use cloudflare as proxy
typora-root-url: ../../source
date: 2021-08-11 22:41:23
tags: [nginx, cloudflare]
---

## Problem

If you put website behind cloudflare, when you log client ip, it logs the ip from cloudflare, not the real client ip



## Solution

You need to tell nginx to set real ip from the `CF-Connection-IP` header when a request is from cloudflare's IP range. For example:

```nginx
set_real_ip_from 103.21.244.0/22;
set_real_ip_from 2a06:98c0::/29;

real_ip_header CF-Connecting-IP;
```

You can get cloudflare's all IPv4 address here: [https://www.cloudflare.com/ips-v4](https://www.cloudflare.com/ips-v4), and all IPv6 address here: [https://www.cloudflare.com/ips-v6](https://www.cloudflare.com/ips-v6). You can name this file `cloudflare.conf` and put it under path: `/etc/nginx/conf.d/cloudflare.conf`:

### cloudflare.conf:

```nginx
set_real_ip_from 103.21.244.0/22;
set_real_ip_from 103.22.200.0/22;
set_real_ip_from 103.31.4.0/22;
set_real_ip_from 104.16.0.0/13;
...
set_real_ip_from 2405:8100::/32;
set_real_ip_from 2c0f:f248::/32;
set_real_ip_from 2a06:98c0::/29;

real_ip_header CF-Connecting-IP;
```



Now the problem is, when cloudflare updated their IP range, you need to update this file too, to make sure your IP list in this configure file is updated with theirs. You can achieve this by running a crontab script:

### nginx_sync_cfip.py

```python
#!/usr/bin/env python3

"""
Use CloudFlare's IP list to generate real_ip config for Nginx
Usage xample:
cp nginx_sync_cfip.py /usr/local/bin/
10 0 * * * python3 /usr/local/bin/nginx_sync_cfip.py >> /var/log/nginx_sync_cfip.log 2>&1
"""

import datetime
import os
import random
import sys
import traceback
from urllib.request import Request, urlopen

CF_IPV4 = "https://www.cloudflare.com/ips-v4"
CF_IPV6 = "https://www.cloudflare.com/ips-v6"

CONF_FILE_PATH = "/etc/nginx/conf.d/cloudflare.conf"
USER_AGENTS = [
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36",
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.107 Safari/537.36",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.114 Safari/537.36",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.1 Safari/605.1.15",
]


def _get_headers():
    return {
        "User-Agent": random.choice(USER_AGENTS)
    }


def _reload_nginx():
    print("Reload nginx")
    os.system("nginx -s reload")
    

def _sync(conf_path):
    outf_lines = []
    ipv4_req = Request(CF_IPV4, headers=_get_headers())
    ipv6_req = Request(CF_IPV6, headers=_get_headers())
    ipv4_list = urlopen(ipv4_req).read().decode().split("\n")
    ipv6_list = urlopen(ipv6_req).read().decode().split("\n")
    ip_list = [*ipv4_list, *ipv6_list]
    for ip in ip_list:
        ip = ip.strip()
        if ip:
            outf_lines.append(f"set_real_ip_from {ip};")
    outf_lines.append("\nreal_ip_header CF-Connecting-IP;")
    latest_content = "\n".join(outf_lines)

    need_update_conf = True
    if os.path.exists(conf_path):
        curr_content = open(conf_path, "r").read()
        need_update_conf = curr_content != latest_content
    
    if need_update_conf:
        print("Update conf file")
        outf = open(conf_path, "w")
        outf.write(latest_content)
        outf.close()
        _reload_nginx()
    else:
        print("Nothing to update")


def nginx_sync_cfip():
    if len(sys.argv) == 2:
        conf_path = sys.argv[1]
    else:
        conf_path = CONF_FILE_PATH

    now = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    print(f"Run @ {now}")
    try:
        _sync(conf_path)
    except Exception:
        print(traceback.format_exc())


if __name__ == "__main__":
    nginx_sync_cfip()

```



This file will detect if there is a change, if there isn't, it won't do anything, otherwise, it will update the `cloudflare.conf` file content then issue a nginx restart. You can run this script every day as crontab under the user `root` (since you need the priviledge to run `nginx -s reload` to restart nginx after the file changes)

```bash
10 0 * * * python3 /usr/local/bin/nginx_sync_cfip.py >> /var/log/nginx_sync_cfip.log 2>&1
```



## Reference

* [Cloudflare - Restoring original visitor IPs](https://support.cloudflare.com/hc/en-us/articles/200170786-Restoring-original-visitor-IPs)

