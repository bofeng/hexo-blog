---
title: Search transaction on Algorand with Golang
typora-root-url: ../../source
date: 2023-03-20 11:55:47
tags: [algorand,golang]
---



## Search Transaction with Golang

```go

package main

import (
	"context"
	"encoding/json"
	"fmt"
	"os"
	"strings"

	"github.com/algorand/go-algorand-sdk/client/v2/common"
	"github.com/algorand/go-algorand-sdk/client/v2/indexer"
)

// here we use purestake api, 
const (
	APIEndpoint string = "https://testnet-algorand.api.purestake.io/idx2"
	APIToken    string = "<your token>"
)

func main() {
	client, _ := common.MakeClientWithHeaders(
		APIEndpoint,
		"X-API-Key",
		APIToken,
		nil,
	)

	if len(os.Args) < 2 {
		fmt.Println("Missing txn id parameter")
		return
	}

	// get the txnid to search from command line
	txnId := os.Args[1]
	indexerClient := indexer.Client(*client)
	txnFinder := indexerClient.LookupTransaction(txnId)
	resp, err := txnFinder.Do(context.TODO())
	if err != nil {
		fmt.Println(err)
		return
	}
	sender := resp.Transaction.Sender
	recver := resp.Transaction.PaymentTransaction.Receiver
	note := resp.Transaction.Note
	fmt.Println("Sender:", sender)
	fmt.Println("Recver:", recver)
	fmt.Println("Note:", string(note))
	fmt.Println(strings.Repeat("=", 20))
	txn, _ := json.MarshalIndent(resp.Transaction, "", "  ")
	fmt.Println(string(txn))
}
```

Run it:
```bash
$ go run main.go <txnid to search>
```

Result:
```
Sender: ZJHG3Z4CRJ4DOEPA5UYFMDWWYY6MA2VSMRM5DHUEQJBNPO2FP2PEKYA43Y
Recver: BOF4NLVXWDKIE7M27BYTKGE76SK26HGOMX7F6RMZIFEBL3VDVUDHIWPPMI
Note: sha256 hash
====================
{
  "application-transaction": {
    "global-state-schema": {},
    "local-state-schema": {}
  },
  "asset-config-transaction": {
    "params": {}
  },
  "asset-freeze-transaction": {},
  "asset-transfer-transaction": {},
  "confirmed-round": 13904215,
  "fee": 1000,
  "first-valid": 13904212,
  "genesis-hash": "SGO1GKSzyE7IEPItTxCByw9x8FmnrCDexi9/cOUJOiI=",
  "id": "<txnid>",
  "keyreg-transaction": {},
  "last-valid": 13905212,
  "note": "c2hhMjU2IGhhc2g=",
  "payment-transaction": {
    "receiver": "BOF4NLVXWDKIE7M27BYTKGE76SK26HGOMX7F6RMZIFEBL3VDVUDHIWPPMI"
  },
  "receiver-rewards": 15219,
  "round-time": 1619979036,
  "sender": "ZJHG3Z4CRJ4DOEPA5UYFMDWWYY6MA2VSMRM5DHUEQJBNPO2FP2PEKYA43Y",
  "sender-rewards": 49484,
  "signature": {
    "logicsig": {
      "multisig-signature": {}
    },
    "multisig": {},
    "sig": "jHbHlhrIhDoOBe59Lw9jKxbs98D6Tc+an47RAd9EfsIYZlqFvnWeiwAdm1RMO2cQDeu0LAwiBvTutFW26IFjDA=="
  },
  "tx-type": "pay"
}
```
