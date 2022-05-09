---
title: Algorand OG Governor NFT
typora-root-url: ../../source
date: 2022-05-09 15:36:24
tags: [algorand, nft]
---



Received Algorand Governor NFT from Pera Wallet, trying to figure out how is the NFT meta data saved.



## Algorand NFT is an ASA

First of all, the NFT is still an asset, take the OG Governor NFT for example, its asset ID is 727568596, the decimal is 0 and the total supply is 4368, which I am assuming there are 4368 users locked more than 10k algos in the 2nd gov period. Getting this NFT is really Pera sent this mint Asset to your wallet. You can see some info about this ASA here: https://algoexplorer.io/asset/727568596



## Get Asset Data

First let's see its data, from the API here:

```bash
https://algoindexer.algoexplorerapi.io/v2/assets?asset-id=727568596
```

we can get its info:
```json
{
  "assets": [
    {
      "created-at-round": 20836641,
      "deleted": false,
      "index": 727568596,
      "params": {
        "clawback": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAY5HFKQ",
        "creator": "PERAAAA6L3OR2Q66TF3CUWDZBVWKDXQKSMEHR2PGDE2QW3KHOOX27MGNCU",
        "decimals": 0,
        "default-frozen": false,
        "freeze": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAY5HFKQ",
        "manager": "PERAAAA6L3OR2Q66TF3CUWDZBVWKDXQKSMEHR2PGDE2QW3KHOOX27MGNCU",
        "name": "OG Governor",
        "name-b64": "T0cgR292ZXJub3I=",
        "reserve": "PERAAAA6L3OR2Q66TF3CUWDZBVWKDXQKSMEHR2PGDE2QW3KHOOX27MGNCU",
        "total": 4368,
        "unit-name": "oggov2",
        "unit-name-b64": "b2dnb3Yy",
        "url": "ipfs://QmXu9UYxfUF7Dm9LEbaUMWfpfvf3VwddvA4QZuxunNXk9P#arc3",
        "url-b64": "aXBmczovL1FtWHU5VVl4ZlVGN0RtOUxFYmFVTVdmcGZ2ZjNWd2RkdkE0UVp1eHVuTlhrOVAjYXJjMw=="
      }
    }
  ],
  "current-round": 20900464,
  "next-token": "727568596"
}
```

Its name is "OG Governor", the unit-name is "oggov2". The creator and manager's address are the same,  and there is not clawback or freeze address.



## Get its meta info

Now it comes to the interesting part, its url: saved in IPFS:

```
ipfs://QmXu9UYxfUF7Dm9LEbaUMWfpfvf3VwddvA4QZuxunNXk9P#arc3
```

If we follow this url, its content actually saved some meta data:

```json
{
  "name": "OG Governor",
  "description": "This special edition NFT was minted in honour of Algorand Governance Period 2 participants. OG Governor badge proves that the account staked 10K+ Algo in the governance program. Keep this badge in your Pera Wallet to have access to future perks.",
  "decimals": 0,
  "unitName": "oggov2",
  "image": "ipfs://QmUwC4PqcjgJtFw6yudMfJKxnJraYLUhuUfysz4tzSzrxX",
  "image_mimetype": "image/png",
  "animation_url": "ipfs://QmSYGhQmCy5qyRjFX5oNSf1Bj3e9HFE7X2YqjiMmUkULmL",
  "animation_url_mimetype": "model/gltf-binary",
  "properties": {
    "collection": {
      "series_name": "Pera Governance II",
      "series_description": "Special edition NFT series minted in honour of Algorand Governance Period 2 participants.",
      "number_of_copies": 4368,
      "creator_name": "Pera Wallet"
    }
  }
}
```



Basically this contains some info we already saw in the asset info, plus some image info: it contains a static image and an animation image (actually it is a 3D model file in .gltf format), both are saved in the IPFS.



## View the NFT image and 3D model

To view the static image, it is "ipfs://QmUwC4PqcjgJtFw6yudMfJKxnJraYLUhuUfysz4tzSzrxX", which we can visit via any ipfs http gateway, like:

https://cloudflare-ipfs.com/ipfs/QmUwC4PqcjgJtFw6yudMfJKxnJraYLUhuUfysz4tzSzrxX



The 3D modal one, "ipfs://QmSYGhQmCy5qyRjFX5oNSf1Bj3e9HFE7X2YqjiMmUkULmL", or visit it via http gateway:

https://cloudflare-ipfs.com/ipfs/QmSYGhQmCy5qyRjFX5oNSf1Bj3e9HFE7X2YqjiMmUkULmL

if you visit it, your browser will ask for a download, you can save it as "nft.gltf" file, then open it with this site: https://gltf-viewer.donmccurdy.com/ :

![image](https://cdn.jsdelivr.net/gh/bofeng/drive@main/uPic/nft-3d.png)



While I am not familiar with the `.gltf` format, it seems this file enables background and it has 2 built-in background environment in it, one is "Venice Sunset", the other is "Footprint Court". It is cool to see this NFT card (3D model) being put with this background -> this may have lots of potentials in the future.
