---
title: 'Setup wireguard VPN'
date: 2020-05-26 17:18:24
tags: [wireguard,vpn,linux,ubuntu]
published: true
hideInList: false
feature: /post-images/setup-wireguard-vpn.png
isTop: false
---
It is really easy to configure wireguard VPN, compare to the old IPsec and OpenVPN methods.

In wireguard method, there is no server/client, they are all peers. So a "server" is  a peer, a client is also a peer. If we want to let multi clients to connect to a server, they are actually multi peers connecting to one peer (server).

## 1, Server configuration

On ubuntu < 19.10, you need to do `sudo add-apt-repository ppa:wireguard/wireguard` first. Version >= 19.10, you can just run: `sudo apt install wireguard`

Once done, generate private-key and public-key pair for your server:
```bash
$ wg genkey
# this will generate a private key, e.g. sL9JUT8axlP4wr6udFCraTLxIVNnYLKHNKtxt/JC42w=
# now use this private key to generate a public key
$ echo "sL9JUT8axlP4wr6udFCraTLxIVNnYLKHNKtxt/JC42w=" | wg pubkey
# this will generate a public key, e.g. +cITDHN+T/AseY9eA4SuZQilDUALxC0lCwtHNC6L8yU=
```

Now go to `/etc/wireguard/` folder, add a config file: `wg0.conf`:
```
[Interface]
Address = 10.0.0.1/24
DNS = 1.1.1.1
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE; ip6tables -A FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE; ip6tables -D FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
ListenPort = 51820
PrivateKey = sL9JUT8axlP4wr6udFCraTLxIVNnYLKHNKtxt/JC42w=
```
In the `PostUp` and `PostDown` step, replace `eth0` if your network is running on a different interface.

Here we are using 51820 as the port. If you are using `ufw`, remember to unblock this port (wireguard is running on UDP protocol)
```bash
$ ufw allow 51820/udp
$ ufw status
```

## 2, Client setup

Client is another peer. Install client here: [https://www.wireguard.com/install/](https://www.wireguard.com/install/)

Wireguard uses public key to identify a peer, so the first setup is still generate private/public key pair:
* MacOS user: open the Wireguard app, click "Add Tunnel" > "Empty Tunnel", a window will pop-up to show generated private/public key, save those keys somewhere, then click "Discard" (Don't click save)
* Windows user: open the Wireguard app, click "Add Tunnel" > "Empty Tunnel", a window will pop-up to show generated private/public key, save those keys somewhere, then click "Cancel" (Don't click save)
* Ubuntu user: use the same way which used by server in above step. i.e. the "wg genkey" etc. method.

Once you have the private key and public key, create a config file anywhere (If you are using Ubuntu, put that config in `/etc/wireguard/wgclient.conf`), the content is like this:
```
[Interface]
DNS = 1.1.1.1
PrivateKey = <your client private key>
Address = 10.0.0.2/32
# if you have multi client across different devices, use different address here like:
# 10.0.0.3/32, 10.0.0.4/32, etc
[Peer]
PublicKey = <server's public key which is generated in the first step> 
Endpoint = <server's ip>:51820
AllowedIPs = 0.0.0.0/0, ::/0
```

Save this `wgclient.conf` file, for MacOS and Windows user, click "Add Tunnel" > "Add Tunnel from existing config file", then select this config file. After this, configuration on the client side are all set. 

## 3, Add client to server
Now we have client's private key and public key, it is time to add the client public key to server. Back to our `/etc/wireguard/wg0.conf` file in server side, then add:
```
[Peer]
# client 1
PublicKey = <client1's public key>
AllowedIPs = 10.0.0.2/32

[Peer]
# client 2
PublicKey = <client2's public key>
AllowedIPs = 10.0.0.3/32
```

Now activate the wireguard in server side:
```bash
# activate
$ wg-quick up wg0
# to deactivate it, run:
# $ wg-quick down wg0
```

## 4, Connect client to server
For MacOS and Windows user, since we have added the configuration file in step 2, all you need to do is click the "activate" button. For Ubuntu user, just run `wg-quick up wgclient`. Once done, your client peer should be now connected to the server peer.

On server side you can run `wg show` to check the connected peers status.

To disconnect it, MacOS and Windows user can just click the "deactivate" button. Ubuntu user can run `wg-quick down wgclient`.

## Others

1, To make wireguard auto run on system boot, run:
```bash
sudo systemctl enable wg-quick@wg0
```

2, If you have some website running in your VPN server and you want to access it with wireguard VPN connected, the access IP now will be the private IP (e.g. 10.0.0.2, 10.0.0.3). If you website have an IP whitelist, you need to list those IPs as well. For example, in Nginx, you can add:
```
allow 10.0.0.1;
allow 10.0.0.2;
```

You can also add a range, e.g. if you want to allow 10.0.0.1 to 10.0.0.19, a "IP to CIDR" address tool can help: [https://www.ipaddressguide.com/cidr#range](https://www.ipaddressguide.com/cidr#range). For example, for IP range 10.0.0.1 to 10.0.0.19, you can add `allow` like this:
```
allow 10.0.0.1/32;
allow 10.0.0.2/31;
allow 10.0.0.4/30;
allow 10.0.0.8/29;
allow 10.0.0.16/30;
```


## Reference
1. [https://www.linode.com/docs/networking/vpn/set-up-wireguard-vpn-on-ubuntu/](https://www.linode.com/docs/networking/vpn/set-up-wireguard-vpn-on-ubuntu/)
2. [https://tau.gr/posts/2019-03-02-set-up-wireguard-vpn-ubuntu-mac/](https://tau.gr/posts/2019-03-02-set-up-wireguard-vpn-ubuntu-mac/)
3. [https://gist.github.com/chrisswanda/88ade75fc463dcf964c6411d1e9b20f4](https://gist.github.com/chrisswanda/88ade75fc463dcf964c6411d1e9b20f4)


