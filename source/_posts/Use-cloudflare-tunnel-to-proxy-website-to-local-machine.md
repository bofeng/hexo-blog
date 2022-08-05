---
title: Use cloudflare tunnel to proxy website to local machine
typora-root-url: ../../source
date: 2022-08-05 11:43:28
tags: [cloudflare]
---

## How to

Cloudflare tunnel is a service like [ngrok](https://ngrok.com/) but better: you can use your own domain for free.

To create a tunnel, go to https://dash.teams.cloudflare.com , click "Access > Tunnels". Then click "Create Tunnel" to map a domain to your localhost service.

Cloudflare will show a Cli command which guide you install `cloudflared` and run the tunnel. After run the command, you can use `ps axu | grep cloudflared` to confirm the process is running.

Now if you visit the configured domain, it will proxy to your local website.

To stop the service, you can stop the tunnel running on your local machine by: `sudo cloudflared service uninstall`



## Reference

* [Cloudflare tunnel guide](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/#set-up-a-tunnel-remotely-dashboard-setup)
