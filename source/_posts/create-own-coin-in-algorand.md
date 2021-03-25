---
title: Create your own coin in Algorand
typora-root-url: ../../source
date: 2021-03-10 20:42:24
tags: [algorand,blockchain]
---



## Intro

I was checking [Algorand](www.algorand.com) recently, found it is very exciting: Fast transaction speed (< 5s), super cheap transaction fee ($0.001 ~ 0.002), staking has 6% APY, support smart contract and most importantly, it has a healty dev community, provides SDK for many popular languages, like Java, Go, Javascript, Python (heard Rust is also under development), it even has a [flutter package](https://pub.dev/packages/dart_algoran).

Algorand provides method to create your own coin, called ASA, like ERC-20 coin in Etherum. I am trying to create one coin with Python and document the steps below.



## Steps

### 1, Create Account

create an account in [PureStake](purestake.io) to use its API. We can host our local server though, the easiest method is using some web API (those website will host the testnet for you), all we need to do is using some SDK to run the code. Once you create an account in [PureStake's developer portal](https://developer.purestake.io/), you will get the API key, save it somewhere, we will need that in the following steps.



### 2, Install python SDK:

```bash
$ pip install py-algorand-sdk
```



### 3, Now write our Python code.

The basic flow is:

1. Create an account
2. Dispense some tokens to this account with [TestNet Dispenser](https://bank.testnet.algorand.network/)
3. Use this account to create a "CreateAssetTransaction"
4. Use this account to sign this transaction
5. Send this transaction to TestNet/MainNet.



Let's do it one by one, first create account. Here we write a Python script to create account:

```python
#!/usr/bin/env python

from algosdk import account, mnemonic


def create_account():
    private_key, public_addr = account.generate_account()
    mn = mnemonic.from_private_key(private_key)
    print(f"Public Address: {public_addr}\nMnemonic: {mn}")


if __name__ == "__main__":
    create_account()
```

Run it to generate the account:

```bash
$ python create_account.py
Public Address: 73XNVFL4MFBBKRAMJZSXOXU5E5XT3EIFYNYDUAHIHB4Q252ZIV6GAAIYVU
Mnemonic: <Your Mnemonic Passphrase>
```



### 4, Put $ALGO in the address

Go to [TestNet dispenser](https://bank.testnet.algorand.network/) to put some tokens into that public address, this will put 100 $ALGO to it.

![](https://tva1.sinaimg.cn/large/008eGmZEly1gofv46s16ej30re0d075l.jpg)



### 5, For the rest steps we create another python script to do it:

```python
#!/usr/bin/env python

from algosdk.v2client import algod
from algosdk import mnemonic, transaction


ALGOD_ENDPOINT = "https://testnet-algorand.api.purestake.io/ps2"
ALGOD_TOKEN = "cK4H1j6j6Z7q0JTxtsuQR4b3OvJ5MzRV1y1bpMdn"


def create_coin():
    mn = input("Your mnemonic words:\n")
    public_addr = mnemonic.to_public_key(mn)
    private_key = mnemonic.to_private_key(mn)

    headers = {
        "X-API-Key": ALGOD_TOKEN,
    }
    # create algod client
    client = algod.AlgodClient(
        algod_token="",
        algod_address=ALGOD_ENDPOINT,
        headers=headers,
    )

    # generate common parameters
    params = client.suggested_params()

    # create this transaction
    txn = transaction.AssetConfigTxn(
        sender=public_addr,
        fee=params.fee,
        first=params.first,
        last=params.last,
        gh=params.gh,
        total=10000000,
        default_frozen=False,
        unit_name="boron",
        asset_name="BoronCoin",
        manager=public_addr,
        reserve=public_addr,
        freeze=public_addr,
        clawback=public_addr,
        url="https://github.com/bofeng",
        decimals=0,
    )

    # sign the transaction
    signed_txn = txn.sign(private_key)

    # send the transaction
    txid = client.send_transaction(signed_txn)
    print(f"Transaction ID: {txid}")


if __name__ == "__main__":
    create_coin()

```



Now let's run it to create coin:

```bash
$ python create_coin.py
Your mnemonic words:
<Paste Your Mnemonic Passphrase Here, Hit Enter>
Transaction ID: UZ5Y3SLUKBEWK5GNZ3QLTW63FXV4WWRIMIB54R3MEGSE37GHXR7A
```



Ok, we got the returned Transaction ID, now use that ID to search the asset info in the [TestNet Explorer](https://goalseeker.purestake.io/algorand/testnet). Put the transaction ID in the input box, we will see our [asset info](https://goalseeker.purestake.io/algorand/testnet/transaction/UZ5Y3SLUKBEWK5GNZ3QLTW63FXV4WWRIMIB54R3MEGSE37GHXR7A):

![](https://tva1.sinaimg.cn/large/008eGmZEly1gofv26i4r2j318w0u0wwy.jpg)



## Reference

* [Build your own coin on Algorand](https://medium.com/algorand/build-your-own-coin-on-algorand-69f5d071181d)
* [Getting Started with the PureStake API Service](https://developer.algorand.org/tutorials/getting-started-purestake-api-service/)
* [Create your Own Coin on TestNet](https://developer.algorand.org/tutorials/create-dogcoin/)
* [Working with ASA using Python](https://developer.algorand.org/tutorials/asa-python/)
* [Build App with 3rd-party API](https://developer.algorand.org/docs/build-apps/setup/#1-use-a-third-party-service)
* [Check txn info in GoalSeeker](https://goalseeker.purestake.io/algorand/mainnet)

