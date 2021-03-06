# Asset API

Blockchains are about transferring value and ownership, so why can't that value extend beyond a blockchain's native token? Thanks to the possibility of embedding arbitrary data through *null-data* outputs, a blockchain can!

However, the act of creating these alternate forms of value---and choosing a format that was universally readable---can be fraught with difficulty. We built our Asset API to simplify that process. Now instead of spelunking through a blockchain's native transactions and interpreting them yourself, you can utilize this set of APIs to easily issue, check, and transfer assets. This set of endpoints currently leverages the [Open Assets Protocol](https://github.com/OpenAssets/open-assets-protocol), a simple, robust method of embedding assets across any blockchain that supports *null-data* outputs.

The endpoints (and code examples) are ordered below to demonstrate how to create asset addresses, issue assets, transfer them, then query the assets/asset addresses you initially created.

<aside class="notice">
The Asset API only works with confirmed transactions; the API will return an error if asset transfer or issuance is attempted using unconfirmed transaction outputs. It also requires the use of <a href="#rate-limits-and-tokens">user tokens</a>, you can <a href="https://accounts.blockcypher.com/">register for one here.</a>
</aside>

## Generate Asset Address Endpoint

```shell
# generate on bitcoin mainnet
curl -X POST https://api.blockcypher.com/v1/btc/main/oap/addrs?token=YOURTOKEN

{
  "private": "c0e09905fddb1b4a42d297cae1a5f20ca4cbe813e92a7dc4035ffc54561c05de",
  "public": "02e9431a3ba8d756a26c03c07ffda2a7e28a17b62051f77ea3944c21dc090ba45b",
  "oap_address": "akFjrBkcu9QQoJeBkwWeH54jnymaGVSUBAs",
  "original_address": "15mxwaoZfvX7SQ2PttY6RCqDeAQ6UqVjuM",
  "wif": "L3ge2aRsGReaBtYVYg9Lau2LxDkRW4Cak1dQsuttShEnRXcA1Vfy"
}

# generate on blockcypher testnet
curl -X POST https://api.blockcypher.com/v1/bcy/test/oap/addrs?token=YOURTOKEN

{
  "private": "0eb369746401c3369517239314a6bc0f2bda6124a4dda15643887f86dc0590cf",
  "public": "026b9f2786b96f7f1aae011a08f5aaccc70f2249606fac4c400d6aa35f50bf7373",
  "oap_address": "1ByJUiocpifLPaYVTALpA7JYa9DxpGxXKKP",
  "original_address": "ByJUiocpifLPaYVTALpA7JYa9DxpLQwte4",
  "wif": "BopcB66fRGgKkwiT2fKyxcawQBJ57DgNyvFq65XuVQdTboxsBkRm"
}

# generate another oap_address to use in examples below
curl -X POST https://api.blockcypher.com/v1/bcy/test/oap/addrs?token=YOURTOKEN

{
  "private": "442abd64783a6411d9e6eb0ed9575c6676bb51824aa3ad4b53577e3d344910c6",
  "public": "03121e17e0996513813c504c63016cea9a7cf9689025e33f9157bccea67ba60846",
  "oap_address": "1C3nrGhUDxBbr393u2Wq4PiE8T6oEYjYhrK",
  "original_address": "C3nrGhUDxBbr393u2Wq4PiE8T6oEYU9At1",
  "wif": "BqcYBWUvXszsRi1A5cD6t21CT7sbBoJQEkRSnFuoznJDn4hw5aGg"
}
```

Resource | Method | Request Object | Return Object
-------- | ------ | -------------- | -------------
/oap/addrs | POST | *nil* | [AddressKeychain](#addresskeychain)

The Generate Asset Address Endpoint returns a private/public keypair and the associated **oap_address** (the asset address) and **original_address** (native address on parent blockchain). Asset addresses are just like regular bitcoin addresses, and are generated from a private-public keypair. They only differ by a prefix (ex: 'a' for instead of '1') to distinguish them.

<aside class="notice">
On Bitcoin's blockchain, <b>oap_address</b> will have an "a" prefix, while the <b>original_address</b> will have a "1" prefix. Note that on BlockCypher's testnet, <b>oap_address</b> will have a "1" prefix, while <b>original_address</b> will have a "B" or "C" prefix. This can be confusing, but unfortunately can't be prevented due to the way <b>oap_address</b> is generated.
</aside>

## Issue Asset Endpoint

```shell
# Since this is a new address, I need to fund underlying bcy original_address 
# to enable issuance transaction (otherwise no money available for mining fees)

# Going to use a faucet thanks to testnet
curl -d '{"address": "ByJUiocpifLPaYVTALpA7JYa9DxpLQwte4", "amount": 1000000}' https://api.blockcypher.com/v1/bcy/test/faucet?token=YOURTOKEN
{"tx_ref": "e7ca58724100f20b81e82ac24cb83cc112627be9cbfd2d09b96a87fbafe9e636"}

# using same underlying (now-funded address) to issue 1000 assets
# also could have used a different, funded bcy address to issue assets to a new oap_address
curl -d '{"from_private": "0eb369746401c3369517239314a6bc0f2bda6124a4dda15643887f86dc0590cf", "to_address": "1ByJUiocpifLPaYVTALpA7JYa9DxpGxXKKP", "amount": 1000, "metadata": "1a2b3c4d5e6f"}' https://api.blockcypher.com/v1/bcy/test/oap/issue?token=YOURTOKEN

{
  "ver": 1,
  "assetid": "1Npqwstp55vgThp4pwAC9UhYkvPJ28b2Ui",
  "hash": "56253cffa1b3508d106391da3646cda2aee0bd080db427321c77ad11739e4239",
  "received": "2015-10-25T05:41:45.092075094Z",
  "double_spend": false,
  "oap_meta": "1a2b3c4d5e6f",
  "inputs": [],
  "outputs": [
    {
      "address": "1ByJUiocpifLPaYVTALpA7JYa9DxpGxXKKP",
      "value": 1000,
      "original_output_index": 0
    }
  ]
}
```

Resource | Method | Request Object | Return Object
-------- | ------ | -------------- | -------------
/oap/issue | POST | [OAPIssue](#oapissue) | [OAPTX](#oaptx)

The Issue Asset Endpoint issues new assets onto an OAP **to_address**, using your private key. Notice that an issuance returns an [OAPTX](#oaptx) with no **inputs**, since it is the original issue.

<aside class="notice">
If you use the same private key to issue multiple times, it's the same asset. To issue different assets, a different issuing key and address are needed each time.
</aside>

<aside class="warning">
For both issuing and transferring assets, the underlying <b>original_address</b> derived from the private key must have enough funds (in the parent blockchain's native token) to pay mining fees for transactions. These fees are adaptively calculated by our API; for Bitcoin, they are usually between 2 to 10 cents per issuance/transfer.
</aside>

## Transfer Asset Endpoint

```shell
# using assetid in endpoint as issued above
# from private key
curl -d '{"from_private": "0eb369746401c3369517239314a6bc0f2bda6124a4dda15643887f86dc0590cf", "to_address": "1C3nrGhUDxBbr393u2Wq4PiE8T6oEYjYhrK", "amount": 200, "metadata": "1a2b3c4d5e6f"}' https://api.blockcypher.com/v1/bcy/test/oap/1Npqwstp55vgThp4pwAC9UhYkvPJ28b2Ui/transfer?token=YOURTOKEN

{
  "ver": 1,
  "assetid": "1Npqwstp55vgThp4pwAC9UhYkvPJ28b2Ui",
  "hash": "022e05bdfa2e148bc1882cb7a81506b8316fee6957b11625126d075a8cf8791b",
  "received": "2015-10-25T05:48:13.417949402Z",
  "oap_meta": "1a2b3c4d5e6f",
  "double_spend": false,
  "inputs": [
    {
      "prev_hash": "56253cffa1b3508d106391da3646cda2aee0bd080db427321c77ad11739e4239",
      "output_index": 0,
      "address": "1ByJUiocpifLPaYVTALpA7JYa9DxpGxXKKP",
      "output_value": 1000
    }
  ],
  "outputs": [
    {
      "address": "1C3nrGhUDxBbr393u2Wq4PiE8T6oEYjYhrK",
      "value": 200,
      "original_output_index": 1
    }
  ]
}
```

Resource | Method | Request Object | Return Object
-------- | ------ | -------------- | -------------
/oap/$ASSETID/transfer | POST | [OAPIssue](#oapissue) | [OAPTX](#oaptx)

The Transfer Asset Endpoint transfers already issued assets (represented by ASSETID) onto a different OAP **to_address**, using your private key.

<aside class="warning">
For both issuing and transferring assets, the underlying <b>original_address</b> derived from the private key must have enough funds (in the parent blockchain's native token) to pay mining fees for transactions. These fees are adaptively calculated by our API; for Bitcoin, they are usually between 2 to 10 cents per issuance/transfer.
</aside>

## List Asset TXs Endpoint

```shell
# using asset id as generated above
curl https://api.blockcypher.com/v1/bcy/test/oap/1Npqwstp55vgThp4pwAC9UhYkvPJ28b2Ui/txs?token=YOURTOKEN

[
  "5f13d5c56ed5b7fe673832e2b5b21feceecebfd08aef03f3208ee855d25f050c",
  "022e05bdfa2e148bc1882cb7a81506b8316fee6957b11625126d075a8cf8791b",
  "7216fd3fb2908f87255f9514c98f40cf1299ea6b9182fa4825f5c5bfacf8b038",
  "56253cffa1b3508d106391da3646cda2aee0bd080db427321c77ad11739e4239",
  "dff06ef62b8d67ce6eb0750ed0a414932c1b13b825870d8711cd81219418f6d4"
]
```

Resource | Method | Return Object
-------- | ------ | -------------
/oap/$ASSETID/txs | GET | Array[*String*]

The List Asset TXs Endpoint returns an array of strings representing transaction hashes associated with this ASSETID.

## Get Asset TX Endpoint

```shell
# using asset id as generated above, and first txhash from list asset txs endpoint
curl https://api.blockcypher.com/v1/bcy/test/oap/1Npqwstp55vgThp4pwAC9UhYkvPJ28b2Ui/txs/5f13d5c56ed5b7fe673832e2b5b21feceecebfd08aef03f3208ee855d25f050c?token=YOURTOKEN

{
  "ver": 1,
  "assetid": "1Npqwstp55vgThp4pwAC9UhYkvPJ28b2Ui",
  "hash": "5f13d5c56ed5b7fe673832e2b5b21feceecebfd08aef03f3208ee855d25f050c",
  "block_height": 506539,
  "confirmed": "2015-10-25T04:50:06Z",
  "received": "2015-10-25T04:49:32.37Z",
  "double_spend": false,
  "oap_meta": "1a2b3c4d5e6f",
  "inputs": [],
  "outputs": [
    {
      "address": "1ByJUiocpifLPaYVTALpA7JYa9DxpGxXKKP",
      "value": 1000,
      "original_output_index": 0
    }
  ]
}
```

Resource | Method | Return Object
-------- | ------ | -------------
/oap/$ASSETID/txs/$TXHASH | GET | [OAPTX](#oaptx)

The Get Asset TX Endpoint returns an [OAPTX](#oaptx) by TXHASH and ASSETID.

## Get Asset Address Endpoint

```shell
# using the transfer address from above
curl https://api.blockcypher.com/v1/bcy/test/oap/1Npqwstp55vgThp4pwAC9UhYkvPJ28b2Ui/addrs/1C3nrGhUDxBbr393u2Wq4PiE8T6oEYjYhrK?token=YOURTOKEN

{
  "address": "1C3nrGhUDxBbr393u2Wq4PiE8T6oEYjYhrK",
  "total_received": 200,
  "total_sent": 0,
  "balance": 200,
  "unconfirmed_balance": 0,
  "final_balance": 200,
  "n_tx": 1,
  "unconfirmed_n_tx": 0,
  "final_n_tx": 1,
  "txrefs": [
    {
      "tx_hash": "022e05bdfa2e148bc1882cb7a81506b8316fee6957b11625126d075a8cf8791b",
      "block_height": 506598,
      "tx_input_n": -1,
      "tx_output_n": 1,
      "value": 200,
      "ref_balance": 200,
      "spent": false,
      "confirmations": 26,
      "confirmed": "2015-10-25T05:49:07Z",
      "double_spend": false
    }
  ],
  "tx_url": "https://api.blockcypher.com/v1/bcy/test/oap/1Npqwstp55vgThp4pwAC9UhYkvPJ28b2Ui/txs/"
}
```

Resource | Method | Return Object
-------- | ------ | -------------
/oap/$ASSETID/addrs/$OAPADDR | GET | [Address](#address)

The Get Asset Address Endpoint returns an [Address](#address) representing information about a particular OAPADDR underneath a given ASSETID. Note that while it returns an [Address](#address) object, anything that would have represented "satoshis" now represents "amount of asset." (e.g., **balance**, **value**, etc.)
