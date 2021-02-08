---
title: 'Play w/ IPFS'
date: 2021-01-10 18:48:02
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
IPFS is a p2p file system.

## Installation
I am using Pop! OS which is a Ubuntu based, so intall the ipfs is pretty easy:
```bash
snap install ipfs
```
`ipfs` stores settings and files in repo, so if this is the first time you install ipfs, you need to initialize it `ipfs init`.

## Add file 
Now you can use `ipfs` to add file:
```bash
$ ipfs add feeds.json
added QmYzTnTfxduwx1N88MyUg4Jushbm7pw1WPSSedMz8WP9RY feeds.json
```

The `QmYzTnTfxduwx1N88MyUg4Jushbm7pw1WPSSedMz8WP9RY` is your file's hash.

## Take node online
```bash
$ ipfs daemon
...
API server listening on /ip4/127.0.0.1/tcp/5001
WebUI: http://127.0.0.1:5001/webui
Gateway (readonly) server listening on /ip4/127.0.0.1/tcp/8080
Daemon is ready
```
Now your IPFS node is online, your node server is by default running at 5001 port, and the gateway is running at 8080 port. The daemon provides a WebUI which you can directly visit in your browser: http://127.0.0.1:5001/webui
Also since your node is online now, other peers can detect and connect to your node.
See your peers address with command `ipfs swarm peers`

And for the file you added, its address is `ipfs://QmYzTnTfxduwx1N88MyUg4Jushbm7pw1WPSSedMz8WP9RY`, but since brower doesn't understand the protocol, it won't know how to open the file - that's where the gateway does it work - the address will fall back to `http://localhost:8080/ipfs/QmYzTnTfxduwx1N88MyUg4Jushbm7pw1WPSSedMz8WP9RY`. Notice the localhost:8080 is your gateway address, and browser support http protocol, so now you can see your file in your browser. Plus, the best thing is, since it is a P2P network, other nodes can detect and cache your file, you can change it to other gateways, like `http://ipfs.io/ipfs/QmYzTnTfxduwx1N88MyUg4Jushbm7pw1WPSSedMz8WP9RY`

Other popular public gateways are:
* dweb.link
* cloudflare-ipfs.com
* cf-ipfs.com

More public gateway domains you can find it [here](https://ipfs.github.io/public-gateway-checker/).

## Add folder
You can host your static files/website, say you have a folder named "website" which contains your static files:
* index.html
* cook.jpg

Code in the `index.html`:
```html
<!DOCTYPE html>
<html>
<head>
    <title>0xBF's play ground</title>
</head>
<body>
    <center>
    <h1>IPFS Play Ground</h1>
    <h2>tasty salad</h2>
    <p>
        <img src="cook.jpg" width="600px">
    <p>
    </center>
</body>
</html>
```
Now put the whole folder online:
```bash
$ ipfs add -r website
added QmTJtZJAmiVoCo7b8LDLHKoQAt5DBFeFc6D2mPxYiEkGVx website/cook.jpg
added QmUqn9j8j5S557jdZuT5yj9dntk7wzWpZbFW5y838M3PVf website/index.html
added QmdUbiyC685LKYZkJCXfZsMnntku9DoX2PAuJsQtKwv5HL website
```
The last hash `QmdUbiyC685LKYZkJCXfZsMnntku9DoX2PAuJsQtKwv5HL` if your folder's IPFS address. If you visit `http://localhost:8080/ipfs/QmdUbiyC685LKYZkJCXfZsMnntku9DoX2PAuJsQtKwv5HL` you will be able to see your site. And if you change the gateway address to some other public gateway address, it will work too (like https://ipfs.io/ipfs/QmdUbiyC685LKYZkJCXfZsMnntku9DoX2PAuJsQtKwv5HL)

## IPNS
If you made some change to the files in website folder, or add some other files, which is not uncommon to update the website, you need to do the `ipfs add -r` again, but you will find that the folder's hash has changed, now the website address is `https://ipfs.io/ipfs/<some other hash value>`. This is very inconvenient if you shared your website address to others, and after you updated the files, your website address changed and you need to re-share the url address. IPNS can help to solve this problem.

IPNS is a "name server", like the DNS system which point a domain name to some ip address. IPNS is used for pointing a fixed `ipns` hash to a `ipfs` hash.

To use the IPNS you need to do `name publish`. Take the above for example, we know the website's **ipfs** hash is `QmdUbiyC685LKYZkJCXfZsMnntku9DoX2PAuJsQtKwv5HL`, now just need to publish it:
```bash
$ ipfs name publish QmdUbiyC685LKYZkJCXfZsMnntku9DoX2PAuJsQtKwv5HL
Published to k51qzi5uqu5dg85apeizh0th2b008n2xmywwnf96coy9an98dd0k1esuyxi0zw: /ipfs/QmdUbiyC685LKYZkJCXfZsMnntku9DoX2PAuJsQtKwv5HL
```
Now we get the **ipns** hash: `k51qzi5uqu5dg85apeizh0th2b008n2xmywwnf96coy9an98dd0k1esuyxi0zw`
Then you can visit your website at: `https://ipfs.io/ipns/k51qzi5uqu5dg85apeizh0th2b008n2xmywwnf96coy9an98dd0k1esuyxi0zw` , notice here in the middle of the path, it is `ipns`, not `ipfs`.

Say now we made some change to our website folder, let's add it again to IPFS:
```bash
$ ipfs add -r website
added QmTJtZJAmiVoCo7b8LDLHKoQAt5DBFeFc6D2mPxYiEkGVx website/cook.jpg
added QmeFPgorCSs6F2u58hmnujKaW9h7GopuXWHgvDTn3vkCsy website/index.html
added Qmd3yZhAKsyFuSgG81d3VM8nRUyzrfttsyZXka4AfPgAWP website
```
`Qmd3yZhAKsyFuSgG81d3VM8nRUyzrfttsyZXka4AfPgAWP` is the new **ipfs** hash for our website folder. We need to publish it again:
```bash
$ ipfs name publish Qmd3yZhAKsyFuSgG81d3VM8nRUyzrfttsyZXka4AfPgAWP
Published to k51qzi5uqu5dg85apeizh0th2b008n2xmywwnf96coy9an98dd0k1esuyxi0zw: /ipfs/Qmd3yZhAKsyFuSgG81d3VM8nRUyzrfttsyZXka4AfPgAWP
```
You can see the ipns hash stay unchanged. Now you fresh the above ipns address (https://ipfs.io/ipns/k51qzi5uqu5dg85apeizh0th2b008n2xmywwnf96coy9an98dd0k1esuyxi0zw)in the browser, you will be able to see the updated version. Under the hood, the ipns address now points to the new ipfs hash.

## When your server is offline
You can take your server down, in the command line, just use Ctrl-C to quit the `ipfs daemon`. After that, your file might still be visited from other gateway or nodes. It depends on how other node handle your file. For example, after you visit the above ipfs.io gateway will cache your files for several hours, so even your server is taken down, you can still visit your website at that address for several hours until the gameway ipfs.io purge its cache. If the node do further - not only cache your file but also `pin` your file - your file will be stored in their server, then you can keep visiting the address until other nodes take it down.


## Reference
* [Installation](https://docs.ipfs.io/install/command-line/)
* [Modify Your Webpage and Republish to IPNS](https://dweb-primer.ipfs.io/publishing-changes/modify-republish)
* [IPFS Repo](https://github.com/ipfs/ipfs)
* [IPFS Companion](https://docs.ipfs.io/install/ipfs-companion/)
* [After my local server stops hosting the file, where does the file go?](https://discuss.ipfs.io/t/after-my-local-server-stops-hosting-the-file-where-does-the-file-go/1438)
* [Getting started with Python and IPFS](https://medium.com/python-pandemonium/getting-started-with-python-and-ipfs-94d14fdffd10)


