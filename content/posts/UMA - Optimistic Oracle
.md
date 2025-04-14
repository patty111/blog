---
title: "UMA   Optimistic Oracle
"
date: 2025-04-14T16:20:14+08:00
tags:
 - ETH
 - web3
 - optimistic-oracle
 - oracle
 - game-theory
 - uma
draft: false
---
> First Introduced by @JackChai - XueDAO Core Contributor

## 特點
	- 是一個樂觀預言機、爭議裁決系統
	- 不像傳統預言機只是餵價(Chainlink), 適用於所有 off-chain Data
	- Schellling-point [謝林點](https://www.google.com/search?q=schelling+point&oq=schelling+point&sourceid=chrome&ie=UTF-8) based blockchain oracles
目前主要其他協議是整合 Optimistic Oracle V3
	- 有整合的有: *Cross Chain bridges*, 保險, 預測市場, customizable DAO tooling
參與者
	- requesters
	- proposers (Asseter)
	- disputes
	- Voters

---
## How Does it Work?
1. 是Optimistic, 就如 Optimistic Rollup, 預設大家都是好人。Verified via a Dispute Period. Anybody can start a dispute.
2. Asserter provides data/assert and 保證金
3. When a dispute is made, send the dispute to UMA's DVM(Data Verification Mechanism), UMA Token Holders will vote to validate the dispute
4. If no dispute is made during this period, the assertion will be consider true
5. The statements are Binary (T/F), not only about prices
6. *Secured By UMA Token Staker -> Secured by economic incentive (經濟激勵)*
![](../../content/images/uma%20cycle.png)
#### Problem
Secured by economic incentives -> **We cannot secure a Market/Protocol That has a Higher Value then Itself** -> We secure 1000\$ using 100\$, if I stoled 1000\$ but only lost 100\$ -> Still make a profit 
e.g. PolyMarket now has a market cap of 200M -> PolyMarket got a higher Market Cap and UMA's value  dropped -> compromising UMA can get profit

*-> UMA oracle is established around the 51% mechanism, namely to attack the oracle we need to launch a 51% attack -> As mentioned above, cannot protect a market that has a higher value then UMA*

@Jack Chai: Potential Solution? -> Restaking, 靜默成長，可以重複利用流動性, TVL(Total Value Locked) 可以翻倍成長



 
![[Pasted image 20250413011510.png]]

#### Optimistic Oracle and Bridge


#### Optimistic Oracle V2 vs V3
[link](https://x.com/pumpedlunch/status/1705273519683358932)

![uma oov2 vs oov3](../../content/images/UMA%20oov2%20vs%20oov3.png)