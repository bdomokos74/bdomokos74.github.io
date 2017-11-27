---
layout: post
title:  "Send some bitcoins from command line!"
date:   2017-11-22 22:06:33 +0100
categories: jekyll update
---
So you want to write some awesome bitcoin apps, but first you want to send some bitcoins manually from the command line to see how it works..

Here's one way to do it using Javascript and <https://bitcoinjs.org/>. First build a transaction string, then push it to the bitcoin network with <https://api.blockcypher.com/v1/btc/main>.

Blockcypher has a mostly RESTful API, you can use to query objects and push transaction to the network.

Start with creating a Node project and install some dependencies like this:
```bash
mkdir bitcoinsend ; cd bitcoinsend
npm init # answer some questions here
npm install bitcoinjs-lib
npm install assert
node
```
Now having a Node REPL up and running, create a private/public key pair (the wallet):
```javascript
var assert = require('assert')
var bitcoin = require('bitcoinjs-lib')

var net = bitcoin.networks.bitcoin;
// Here it is possible to use bitcoin.networks.testnet to experiment first. You can get free tBTCs (test bitcoins) to get started.
var newKeyPair = bitcoin.ECPair.makeRandom(
  {network: net}
);

newKeyPair.toWIF(); // this is your private key
// L3q7PXQK2CH3SVn6xwterkX6SyJWGyHdtCQ3NTrFmMRSwLXut4Ao
// Hmm, you shouldn't post your private key to the internet if you plan to store a lot of bitcoins here...

newKeyPair.getAddress(); // this is your address, the public key
// 141qda2EbVYxTbZurhKgV5fuCrd1yfWJ3Q
```

Use the new address to receive some bitcoins from a wallet or an exchange: send some bitcoins to the new address. When done with that, find the transaction. Just paste the transaction id (your wallet should show) to the search box in <https://live.blockcypher.com/>.

You get something like: [https://live.blockcypher.com/btc/tx/f6c70b2fddbcdd14df35ed7...](https://live.blockcypher.com/btc/tx/f6c70b2fddbcdd14df35ed723de6238b5567eb2aafcc7a9f4c07e669b5ff288b/)

But it can be done without the browser, curl and [jq](https://stedolan.github.io/jq/) can get the necessary data without leaving the command line:
```bash
curl https://api.blockcypher.com/v1/btc/main/txs/f6c70b2fddbcdd14df35ed723de6238b5567eb2aafcc7a9f4c07e669b5ff288b | jq '.outputs[] | {addr: .addresses[0], val: .value}'
```
The results are as follows:
```json
{
  "addr": "141qda2EbVYxTbZurhKgV5fuCrd1yfWJ3Q",
  "val": 709555
}
{
  "addr": "1Mj3z4Doa3RwqzZWuSZWBoxitAqHq8ssZ4",
  "val": 1703290133
}
```

Note the the address and the amount from the above (unfortunately the first output is mine):
```
var totalAmount = 709555;

var fee = 452*170;

var target1Address = '1K1KCb9CzxtdHAJ9aYBLeK9tmDJboqMke4';
var target1Amount = 196146;

var target2Address = '121jAd6mUF9NYocrUvopyFfCgH99SRJHZg';
var target2Amount = totalAmount-target1Amount-fee;

console.log(`Sending \n${target1Amount} to ${target1Address}\n${target2Amount} to ${target2Address}\nfee=${fee}`)
```
Gives:
```
Sending
196146 to 1K1KCb9CzxtdHAJ9aYBLeK9tmDJboqMke4
436569 to 121jAd6mUF9NYocrUvopyFfCgH99SRJHZg
fee=76840
```

I got the fee 170 from <https://bitcoinfees.earn.com/>. This shows the fee as Satoshis/byte. Then I selected 452 as the length of the transaction string. In hindsight, I should have divided it by 2 to get the number of bytes instead of the number of chars :)

Now build the transaction (get tx id from above):

```javascript
var builder = new bitcoin.TransactionBuilder(net);
var input = 'f6c70b2fddbcdd14df35ed723de6238b5567eb2aafcc7a9f4c07e669b5ff288b';
builder.addInput(input, 0);
assert((target1Amount+target2Amount+fee)===totalAmount);
builder.addOutput(target1Address, target1Amount);
builder.addOutput(target2Address, target2Amount);
builder.sign(0, newKeyPair);
builder.build().toHex();
```
Then use curl from your command line to push the transaction to the bitcoin network. You need to register to blockcypher and get an API token. It is free if you are not doing large volume of transactions, you have to upgrade if you need more. Replace YOUR_API_TOKEN with it below (and the transaction of course):
```bash
curl -d '{"tx":"01000000018b28ffb569e6074c9f7accaf2aeb67558b23e63d72ed35df14ddbcdd2f0bc7f6000000006b483045022100c410cdae2da49a60805cc9dc7e9bb3cc5aa28f7c0cb8db4b97d7710a09f4dbee02206705bc633bca1043c83f57b269fa5d012a37d6dd0018ed13daff9b9369db104d01210238c3ba35dc612c862d331e0945d652ccb8d02caca67bd04a4ee13d3a28e06fa5ffffffff0232fe0200000000001976a914c58174dfc5cd07d94bffa427f86f4239c9d3b87388ac59a90600000000001976a9140b1b52f8b4d12d5f70d15996c5a19f0fbc0f252088ac00000000"}' https://api.blockcypher.com/v1/btc/main/txs/push?token=YOUR_API_TOKEN
```
Gives:
```json
{
  "tx": {
    "block_height": -1,
    "block_index": -1,
    "hash": "c9d9d441705d47e49c522fc4580aecfdb80cb3d89334b84f9c391c7f23696d9f",
    "addresses": [
      "141qda2EbVYxTbZurhKgV5fuCrd1yfWJ3Q",
      "1K1KCb9CzxtdHAJ9aYBLeK9tmDJboqMke4",
      "121jAd6mUF9NYocrUvopyFfCgH99SRJHZg"
    ],
    "total": 632715,
    "fees": 76840,
    "size": 226,
    "preference": "high",
    "relayed_by": "31.10.155.31",
    "received": "2017-11-21T22:19:33.634722732Z",
    "ver": 1,
    "double_spend": false,
    "vin_sz": 1,
    "vout_sz": 2,
    "confirmations": 0,
    "inputs": [
      {
        "prev_hash": "f6c70b2fddbcdd14df35ed723de6238b5567eb2aafcc7a9f4c07e669b5ff288b",
        "output_index": 0,
        "script": "483045022100c410cdae2da49a60805cc9dc7e9bb3cc5aa28f7c0cb8db4b97d7710a09f4dbee02206705bc633bca1043c83f57b269fa5d012a37d6dd0018ed13daff9b9369db104d01210238c3ba35dc612c862d331e0945d652ccb8d02caca67bd04a4ee13d3a28e06fa5",
        "output_value": 709555,
        "sequence": 4294967295,
        "addresses": [
          "141qda2EbVYxTbZurhKgV5fuCrd1yfWJ3Q"
        ],
        "script_type": "pay-to-pubkey-hash",
        "age": 495482
      }
    ],
    "outputs": [
      {
        "value": 196146,
        "script": "76a914c58174dfc5cd07d94bffa427f86f4239c9d3b87388ac",
        "addresses": [
          "1K1KCb9CzxtdHAJ9aYBLeK9tmDJboqMke4"
        ],
        "script_type": "pay-to-pubkey-hash"
      },
      {
        "value": 436569,
        "script": "76a9140b1b52f8b4d12d5f70d15996c5a19f0fbc0f252088ac",
        "addresses": [
          "121jAd6mUF9NYocrUvopyFfCgH99SRJHZg"
        ],
        "script_type": "pay-to-pubkey-hash"
      }
    ]
  }
}
```
Copy the transaction id and browse to
[https://live.blockcypher.com/btc/tx/c9d9d441705d47e49c...](https://live.blockcypher.com/btc/tx/c9d9d441705d47e49c522fc4580aecfdb80cb3d89334b84f9c391c7f23696d9f/)
Your receiving wallet should notify you that you are getting bitcoins. Depending on the fee you used, the time to confirm will be varying.

## Further links

I found the following resources very interesting:
<https://crypto.stanford.edu/cs251_fall15/>

With a link to the textbook here: <https://crypto.stanford.edu/cs251/syllabus.html>

<https://github.com/bitcoinjs/bitcoinjs-lib>

Blockcypher API DOC: <https://www.blockcypher.com/dev/bitcoin/>

For processing JSON from the command line:
<https://stedolan.github.io/jq/>
