---
title: 'Use cloudflare to https github page'
date: 2016-03-31 22:22:05
tags: [cloudflare,github]
published: true
hideInList: false
feature: 
---
Steps:

1, Add "CNAME" file to github page, the content is your domain. Later in your settings, you can confirm it.

![Custom Domain](/image/github-domain.png)

2, The following operation are all in cloudflare: add your domain to cloudflare, in DNS settings, add CNAME:

![Cloudflare DNS](/image/cloudflare-dns.png)

3, Go to "Crypto" settings, change SSL to "full".

![Cloudflare SSL](/image/cloudflare-crypto.png)

4, In "Page rules" settings, add forward rule, so when user visit every page w/o https, will be redirect to https page.

![Cloudflare Forward](/image/cloudflare-pr.png)



Wait for a while, you should visit your custom domain via https, which points to your github page.
