---
title: "Introduction to Across Protocol"
date: 2025-05-01T15:48:07+08:00
tags:
 - ETH
 - web3
 - bridging
 - intent-based
 - across-protocol
 - cross-chain
summary: "An introduction to Across Protocol and its intent-based cross-chain bridging solution."
draft: false
---
最近要準備面試，內容就是 Across Protocol 的架構理解與相關問題，這篇文章是我的一些小整理與學習筆記。

## What is Intent?
中文直翻是「意圖」，換句話說就是讓使用者只需要描述「我想要做什麼」，不需要自己操作一連串繁瑣的步驟。

##### Example
> Intent vs Tx  
> **Intent**：Uber司機大哥，我要去台大正門口  
> **Tx**：開門、發動車子、轉彎、直走、...最後停在羅斯福路四段1號

There will be a **Solver / Relayer** to fulfill the user's intent. The relayer may even pay the gas fee in advance to speed up the response time.

## Why Intent is Powerful
1. **User Experience**：Users doesn't need to know what chain ID is, or which contract to call and which function to execute. They don't need to worry about gas fees or slippage.
2. **Best Execution**：Multiple Solvers compete to fulfill user intents, automatically optimizing for price and speed. The processing layer is abstracted off-chain, making gas fees no longer a issue.
3. **MEV Resistance**：intents can be off-chain batch/match, making it difficult to frontrun/sandwich.

## Intent Types
### Conditional Intents
Execute when certain conditions are met, such as limit orders(限價單).
e.g.「熱錢包 ETH 不夠時，從冷錢包轉一點過來。」

### Continuous Intents
Same as DCA (Dollar Cost Averaging), automatically buy ETH with DAI every month.
User will only need to send their intent, and the Solver will take care of:
- When to swap
- Whether DAI is enough
- Whether the swap is profitable (after gas and risk costs)
### Multi-Step Intents
## Intent Architecture
Intent architecture can fulfill user's intent super fast, but it requires the protocol to pay the gas fees in advance. The protocol will then verify the intent and repay the relayer after the verification is complete (貨到付款).

The urgent part (filling user swap/deposit) is decoupled from the complex verification. The cons is that the relayer has to loan funds for the duration between the verification period, but in exchange we get a super fast response time and a great user experience.

## Architecture of Across Protocol
We can split the architecture into 3 parts:
1. User Interface and a RFQ (即時報價) system
2. Off-chain Solvers (relayers)
3. On-Chain Pools (contracts) and Verifiers (UMA optimistic oracle)

### Deposit Flow Example
Let's say Alice wants to deposit 1 ETH from Arbitrum to Optimism. The flow is as follows:
1. Alice calls a deposit function on origin chain Spokepool contract (ARB) and deposits 1 ETH.
2. The Spokepool contract will emit a `Deposit` event, which will be picked up by the relayer.
3. The relayers currently is using a polling mechanism to check the `Deposit` event. The polling interval is something like 5 seconds.
4. The relayers will then try to call the `Fulfill` function on the destination chain Spokepool contract (OP) to fulfill the deposit.
5. It is a first come first serve basis, so the first relayer that calls the `fillRelayV3` function successfully and makes the deposit will claim the order. The order will then be locked by the contract to avoid duplicate deposits.
6. When the relayer fills the order, a `Fulfill` event will be emitted.
7. The dataworker (off chain) will periodically polls for the `Fullfill` event and the `Deposit` event to match the two. 
8. After several matches are made, the dataworker will batch the filled orders into a merkle tree and submit it to the HubPool contract (Mainnet).
9. A UMA oracle will verify deposits in the submitted merkle tree. The dispute period is about 1 hour.
10. If no disputes are raised, the HubPool contract will repay the relayer with the rewards. A relayer can decide to take the reward on any chain via setting params in the merkle tree.

![alt text](../../images/across-protocol-arch.png)


After the research I came up with a question: 
Why is the Hubpool Contract on Mainnet? I thought relayers can decide to take the reward on any chain rather than only on Mainnet?

The Main Liquidity is on Mainnet, but there is a **Pool Rebalancing Mechanism** in the HubPool Contract. A dataworker can interact with merkle roots via inclusion proofs to instruct the contract  to send tokens to L2 SpokePools to achieve pool rebalance.

As for why Hubpool is on mainnet, here are a few possible reasons I came up with:
1. UMA is only supported on Mainnet
2. To prevent Liquidity fragmentation, Across don't need to manage multiple pools on different chains. 

The interviewers said they had the same question when they researched the protocol. They came up with another possible reason:
Native bridging from L1 to L2 is seconds to minutes. On the contrary, bridging from L2 to L1 is 7 days.

### Bundle Proposal
A bundle is proposed by the dataworker to the HubPool contract. The bundle contains a merkle root of the filled orders. A worker has to post a bond. If the propose is valid, the worker will get the bond back with some rewards. If the propose is challenged and succeeded, the worker will lose the bond.

After verifying the bundle, it will be sent to the HubPool. Funds and repayment instructions will be made according to the bundle settings.
### What if the Relayers Failed to Fulfill the Order?
There is a **Slow Fill** mechanism. If the relayer fails to fulfill the order in a time window, the user order will be filled on a protocol level. The protocol will use its liquidity to fill the deposit, which is a slow fill. 

## Questions I am curious about / Interview Discussions
1. What may be the cause of the delay in the deposit process?
A: polling interval, deposit tx execution time, relayer's network latency...

1. Possible vulnerabilities in the protocol?
2. What risks do the relayers face?
3. Are the relayers incentivized to fill the orders?
4. Polling is not a good solution, what are the alternatives?
A: perhaps using a socket connection to listen to the events. Or use a queue to let relayers first come first serve.

-> Follow up: 
- If queue is off-chain, who will be ideal to maintain the queue? 
- How to prevent bypassing the queue?