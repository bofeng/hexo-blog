---
title: Create ASA on Algorand with Golang
typora-root-url: ../../source
date: 2023-03-20 11:54:05
tags: [algorand,golang]
---



There are some tutorials you can find in the internet for creating ASA on [Algorand](http://algorand.com/) with Go are outdated. Most of them are using V1's API. It took some time for me to fix the code with API V2, so just share the code here.

The basic flow doesn't change:
1. Create a "Create Asset Transaction"
2. Sign this Transaction with your account
3. Send it to Algorand mainnet/testnet

## Preparation
1. You need to get a [Purestake.io Algorand API](https://www.purestake.com/technology/algorand-api/), register an account needed
2. In your go project, install Algorand Go module:
```bash
$ go get -u github.com/algorand/go-algorand-sdk/... 
```
This will install all packages in Algorand's Go SDK.
3. The go code below reads some configurations from a config file in JSON format, so you will need to create this `config.json` in your project folder first:

```json
{
    "APIEndpoint": "<your purestake.io api endpoint>",
    "APIToken": "<your purestake.io apikey>",
    "OwnerAddr": "<your algorand wallet public address>"
}
```

## Code

Now you can use the code below to create your own ASA. You will need to input your mnemonic words during the process. Also, feel free to change your ASA's total supply (`coinTotalIssuance`), decimals, unite name and name.

```go
package main

import (
	"bufio"
	"context"
	"encoding/base64"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"log"
	"os"
	"strings"

	"github.com/algorand/go-algorand-sdk/client/v2/algod"
	"github.com/algorand/go-algorand-sdk/client/v2/common"
	"github.com/algorand/go-algorand-sdk/crypto"
	"github.com/algorand/go-algorand-sdk/mnemonic"
	"github.com/algorand/go-algorand-sdk/transaction"
)

type Config struct {
	APIEndpoint string `json:"APIEndpoint"`
	APIToken    string `json:"APIToken"`
	OwnerAddr   string `json:"OwnerAddr"`
}

func main() {
	ctx := context.Background()
	cfg := Config{}
	content, err := ioutil.ReadFile("./config.json")
	if err != nil {
		log.Fatalln("Failed to read config file")
	}
	err = json.Unmarshal(content, &cfg)
	if err != nil {
		log.Fatalln("Unable to parse config file")
	}

	// Create an algod client
	client, err := common.MakeClientWithHeaders(
		cfg.APIEndpoint,
		"X-API-Key",
		cfg.APIToken,
		nil,
	)
	if err != nil {
		log.Fatalln("Failed to make algod client:", err)
	}

	algodClient := algod.Client(*client)

	fmt.Println("Input your mnemonic words:")
	reader := bufio.NewReader(os.Stdin)
	mn, err := reader.ReadString('\n')
	if err != nil {
		log.Fatalln("Failed to read mnemonic words")
	}
	mn = strings.TrimSuffix(mn, "\n")

	// Recover private key from the mnemonic
	privKey, err := mnemonic.ToPrivateKey(mn)
	if err != nil {
		log.Fatalln("Error to get private key from mnemonic key")
	}

	// Get the suggested transaction parameters
	txParams, err := algodClient.SuggestedParams().Do(ctx)
	if err != nil {
		log.Fatalln("Error getting suggested tx params:", err)
	}

	txParams.FlatFee = true
	txParams.Fee = 1000

	// Make transaction
	coinTotalIssuance := uint64(10000000)
	coinDecimalsForDisplay := uint32(0)
	accountsAreDefaultFrozen := false // if you have this coin, you can transact, the freeze manager doesn't need to unfreeze you first
	managerAddress := cfg.OwnerAddr   // the account issuing this is also the account in charge of managing this
	assetReserveAddress := ""
	addressWithFreezingPrivileges := cfg.OwnerAddr // this account can blacklist others from receiving or sending assets, freezing their account
	addressWithClawbackPrivileges := cfg.OwnerAddr // this account is allowed to clawback coins from others
	assetUnitName := "COINUNIT"
	assetName := "COINNAME"
	assetUrl := "https://coinurl.com"
	assetMetadataHash := ""
	tx, err := transaction.MakeAssetCreateTxn(
		cfg.OwnerAddr,
		uint64(txParams.Fee),
		uint64(txParams.FirstRoundValid),
		uint64(txParams.LastRoundValid),
		nil,
		txParams.GenesisID,
		base64.StdEncoding.EncodeToString(txParams.GenesisHash),
		coinTotalIssuance,
		coinDecimalsForDisplay,
		accountsAreDefaultFrozen,
		managerAddress,
		assetReserveAddress,
		addressWithFreezingPrivileges,
		addressWithClawbackPrivileges,
		assetUnitName,
		assetName,
		assetUrl,
		assetMetadataHash,
	)

	if err != nil {
		log.Fatalln("Error creating transaction:", err)
	}

	// Sign the Transaction
	_, bytes, err := crypto.SignTransaction(privKey, tx)
	if err != nil {
		log.Fatalln("Failed to sign transaction:", err)
	}

	// Broadcast the transaction to the network
	txID, err := algodClient.SendRawTransaction(bytes).Do(ctx)
	if err != nil {
		log.Fatalln("failed to send transaction:", err)
	}

	log.Println("Transaction successful with ID: ", txID)
}
```

## Reference
* https://developer.algorand.org/tutorials/asa-go/
* https://medium.com/algorand/build-your-own-coin-on-algorand-69f5d071181d
* https://github.com/algorand/go-algorand-sdk/blob/develop/client/v2/common/common.go
* https://github.com/algorand/go-algorand-sdk/blob/develop/client/v2/algod/algod.go
