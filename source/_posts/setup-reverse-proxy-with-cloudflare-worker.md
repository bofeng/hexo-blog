---
title: setup reverse proxy with cloudflare worker
typora-root-url: ../../source
date: 2022-05-14 12:08:11
tags: [cloudflare, proxy]
---



We can setup a reverse proxy for our website with Cloudflare's "workers" feature.

## Steps:

### 1, Setup workers.dev domain

Sign in to cloudflare, then go to the "workers" tab on the left. If you haven't used it before, it will ask you to setup some subdomain under workers.dev. For example, here we choose 0xbf.workers.dev

### 2, Create a service

Click "Create a Service", give the service a name, for example, "proxy-pass", then select "HTTP Handler". Then click "Create service"

### 3, Add reverse proxy code

Once you have this service, click "Quick Edit", paste the code below:

```javascript
addEventListener("fetch", event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  const method = request.method
  // here we only allow the GET method
  if (method !== "GET") {
    return new Response("405 Method not allowed", {
      status: 405,
    })
  }
  const url = new URL(request.url)
  const pathname = url.pathname
  if (!pathname.startsWith("/abc/")) {
    return new Response("404 Not found", {status: 404})
  }
  const subpath = pathname.substring(5) // 5 is the length of "/abc/"
  return fetch("https://abc.example.com/proxy/" + subpath)
}
```

What the code does is that, when you visit "proxy-pass.0xbf.workers.dev/abc/xxx", it will proxy this request to "abc.example.com/proxy/xxx", the content will be fetched from abc.example.com but the url in the browser stays unchanged.

Once you have this code, click "Save and Deploy", then test it with "proxy-pass.0xbf.workers.dev/abc/xxx"

### 4, Use your custom domain

The code works for the workers.dev subdomain, if you want to make it work with your own domain, say you want to make it work with proxy.mydomain.com, you can attach this worker to your domain.

Click "websites", then click your own domain (mydomain.com) in cloudflare, then click "workers" on the left. This time, click "Add Route", in the route rule, set "route" to "proxy.mydomain.com/abc/*", set "service" to the "proxy-pass" service you just created in the previous step, then click save.

Here we are using proxy.mydomain.com as an example. If you don't have such subdomain setup, go to the DNS tab on the left, then create a new A name: "proxy" to IP "192.0.2.1" (The IP doesn't really matter here, since all the traffic will be routed to abc.example.com, not to a real server. Here we are using 192.0.2.1 b/c it is test network ip that not used by anyone). Also make sure the "proxy status" is turned on with "proxied", then click save.

Now you can just visit proxy.mydomain.com/abc/xxx, the traffic will be routed to abc.example.com/proxy/xxx, and you get reponse from there without a url-change in the browser address bar.



## Reference

* https://servebolt.com/help/article/cloudflare-workers-reverse-proxy/

* https://owenconti.com/posts/create-a-reverse-proxy-with-cloudflare-workers
* https://gist.github.com/mashhoodr/0640e3cbdf007029af725091b6493eb0
* https://community.cloudflare.com/t/setup-workers-on-personal-domain/88012/3
