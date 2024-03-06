---
title: ERC-20 Token Transaction Fetching
date: 2024-02-23T03:00:55+08:00
summary: Decoding tx details, interpreting ERC-20 token tx data from blockchain and etherscan.io
tags:
 - blockchain

draft: false
---

Fetching tx data directly from blockchain using block number will return the following result, which I'll refer to ass 'dataA' for clarity:

```json
{
	"blockHash": "0x08c18cfe02a25bf726e07da5dc51a4b5da476fe5818fc9fe8ebc54ddde27d69f", 
	"blockNumber": "0x12086e3", 
	"from": "0x5fff56c053d5428af6c34341535e0f6f9d5ebb4e", 
	"gas": "0x9c40", 
	"gasPrice": "0x342770c00", 
	"maxFeePerGas": "0x342770c00", 
	"maxPriorityFeePerGas": "0x77359400", 
	"hash": "0x7adc2347f01d052974adf647332c2696d9d5a7f84f1f225284edb5f93c5c00a6", 
	"input": "0xa9059cbb000000000000000000000000b8ba36e591facee901ffd3d5d82df491551ad7ef000000000000000000000000000000000000000000000046821749635299f000", 
	"nonce": "0x179", 
	"to": "0xfc98e825a2264d890f9a1e68ed50e1526abccacd", 
	"transactionIndex": "0x3d", 
	"value": "0x0", 
	"type": "0x2", 
	"accessList": [], 
	"chainId": "0x1", 
	"v": "0x0",
	"r": "0xf7016189d42aeb0beb537798880730b89d6670b0832256aa4f0ae801728c1d27", 
	"s": "0x309ca50e1f8c14b2b449c9dba48d81d3d75c46b3b79261c48fdf684a5127c303", 
	"yParity": "0x0"
}
```

While downloading csv data directly from etherscan.io will yield the following, which I'll refer to as 'dataB':

```python
Txhash,Blockno,UnixTimestamp,DateTime (UTC),from,From_PrivateTag,to,To_PrivateTag,Quantity,Method

0x7adc2347f01d052974adf647332c2696d9d5a7f84f1f225284edb5f93c5c00a6,18908899,1704067259,2024/1/1 00:00,0x5fff56c053d5428af6c34341535e0f6f9d5ebb4e,-,0xb8ba36e591facee901ffd3d5d82df491551ad7ef,-,"1,300.65",Transfer

...
```

We can compare the <mark>to</mark> and <mark>from</mark> in dataA and dataB. The 'from' fields are the same, representing the address from which this transfer request originated.

However, the 'to' fields are not quite the same.

In dataA it is `0xfc98e825a2264d890f9a1e68ed50e1526abccacd`, this is the MCO2 Token address, indicating we are interacting with MCO2. 
In dataB it is `0xb8ba36e591facee901ffd3d5d82df491551ad7ef`, the target address of this transfer. But where does this target go when we directly fetched the data from the blockchain(dataA)?

The 'input field' of raw tx data from blockchain conatains the encoded function call. After parsing it we can interpret the following information:

```
input = 0xa9059cbb000000000000000000000000b8ba36e591facee901ffd3d5d82df491551ad7ef000000000000000000000000000000000000000000000046821749635299f000
```
```python
---
byte 0-8: method -> 0xa9059cbb
"""
after looking up in 4byte.dictionary.com,
the function is probably transfer(address,uint256)
"""
---
next 64 bytes: recipient address -> 
"""
0x000000000000000000000000b8ba36e591facee901ffd3d5d82df491551ad7ef
"""
---
last 64 bytes: amount transferred in hexadecimal, 'wei' as unit
"""
0x000000000000000000000000000000000000000000000046821749635299f000
= 1300646127000000000000 wei
= 1300.646127 eth
"""
---
```
