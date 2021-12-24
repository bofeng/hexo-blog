---
title: Upload to IPFS with nft.storage
typora-root-url: ../../source
date: 2021-12-24 00:02:30
tags: [ipfs, golang]
---

## Problem

You want to upload images / assets to IPFS network w/o running your own node



## Solution

There are some paid services like pinata.cloud. And fortunately I found the free service called [nft.storage](https://nft.storage) that provide free API.



### Steps

1, Go to [nft.storage](https://nft.storage) , register an account to get an API token. It is free

2, Upload to it with normal HTTP Post request.



### Examples

**Use curl:**

```bash
curl -X POST --data-binary @art.jpg -H 'Authorization: Bearer YOUR_API_KEY' https://api.nft.storage/upload
```

it will return format like this:

```json
{
  "ok": true,
  "value": { "cid": "bafy..." }
}
```

The `cid` is your uploaded asset ID in IPFS, you can visit it with many IPFS gateways, like:

```bash
https://ipfs.io/ipfs/<cid>
https://<ipfs>.ipfs.dweb.link
https://cloudflare-ipfs.com/ipfs/<cid>
```



Use Golang:

```go
package main

import (
	"context"
	"fmt"
	"io/ioutil"
	"log"

	"github.com/carlmjohnson/requests"
)

const (
	API_URL  = "https://api.nft.storage/upload"
	API_KEY  = "<API Key>"
	ENDPOINT = "https://%s.ipfs.dweb.link"
)

type RespValue struct {
	CID string `json:"cid"`
}

type Resp struct {
	OK    bool      `json:"ok"`
	Value RespValue `json:"value"`
}

func main() {
	fileBytes, err := ioutil.ReadFile("./cat.png")
	if err != nil {
		log.Fatal(err)
	}

	resp := Resp{}
	authToken := fmt.Sprintf("Bearer %s", API_KEY)
	err = requests.URL(API_URL).
		BodyBytes(fileBytes).
		ToJSON(&resp).
		Header("Authorization", authToken).
		Fetch(context.Background())

	if err != nil {
		log.Fatal(err)
	}

	cid := resp.Value.CID
	url := fmt.Sprintf(ENDPOINT, cid)
	fmt.Printf("Uploaded: %s\n", url)
}
```

This will upload a image named `cat.png` to IPFS network:

```bash
$ go run main.go
Uploaded: https://bafybeigp4zbuq2xm5mc4n4oyfkgntyq3ooxb4vqrbguq7hmvbfeb2nh3dq.ipfs.dweb.link
```

You can view this image here from [this link](https://bafybeigp4zbuq2xm5mc4n4oyfkgntyq3ooxb4vqrbguq7hmvbfeb2nh3dq.ipfs.dweb.link)



## References:

* [NFT Storage](https://nft.storage/)
* [CloudFlare IPFS](https://developers.cloudflare.com/distributed-web/ipfs-gateway)

* [HTTP requests for Gophers](https://github.com/carlmjohnson/requests)
