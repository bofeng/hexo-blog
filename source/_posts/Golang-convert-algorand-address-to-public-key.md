---
title: 'Convert algorand address to public key with Golang'
typora-root-url: ../../source
date: 2023-03-20 11:46:17
tags: [golang,algorand]
---



## Problem

You have a algorand account address, you want to derive the public key from it with Golang.



## Solution

Install Algorand SDK:
```bash
$ go get -u github.com/algorand/go-algorand-sdk/...
```


```go

import (
    "crypto/ed25519"
    "crypto/sha512"
    "encoding/base64"
    
    algocrypto "github.com/algorand/go-algorand-sdk/crypto"
    "github.com/algorand/go-algorand-sdk/types"
)


func algoAddrToPublicKey(address string) (ed25519.PublicKey, error) {
	addr, err := types.DecodeAddress(address)
	if err != nil {
		return nil, err
	}
	bytes := [sha512.Size256]byte(addr)
	return ed25519.PublicKey(bytes[:]), nil
}

func main() {
	addr := "<wallet address, 58 characters>"
	pubKey, _ := algoAddrToPublicKey(addr)
}
```

With the public key we can verify if a signed message is valid:

```go
msg := "message to sign"
rawSig := "<base64 signature string>"
sig, _ := base64.StdEncoding.DecodeString(rawSig)
valid := algocrypto.VerifyBytes(pubKey, []byte(msg), sig)
if valid {
  fmt.Println("signature is valid")
}
```

