---
title: "Solana Pay 101"
date: 2024-08-01T18:27:40+08:00
tags:
 - blockchain
 - solana
 - solana-pay
 - XueDAO
draft: false
---
### Solana是什麼碗糕?
Solana 是一個由Solana Labs於2020推出的區塊鏈網路, 採用PoS與PoH機制，與ETH、BTC相比有著更高的效能與交易頻率，更低的能耗與手續費。
![alt text](../../images/blockchain-network-comparison.png)
![alt text](../../images/phantom-wallet-testnet-pic.png)

### 那Solana Pay呢？用它做什麼？
Solana pay是基於Solana區塊鏈開發的支付協議。可以把它想像成LINE Pay一樣，打開手機上的錢包app，開啟qrcode scanner，掃一下就可以付款了。和以往不同的是現在可以隨意用加密貨幣(SOL, USDC...)直接進行支付。知名的國際電商Shopify就在去年八月導入了Solana Pay作為平台支付方式之一。

### 導入Solana Pay的好處
導入Solana Pay的好處在於消費者在電商平台上交易的時候可以避免匯率轉換的損失和動輒3~4%的信用卡手續費(Solana的gas fee根本可以當做免手續費了!）。由於區塊鏈的透明性與安全性，平台與客戶雙方都能輕鬆的對交易紀錄做追蹤。不再需要第三方平台介入也無需在網站上留下自己的信用卡資料，近一步減少敏感資訊外流的風險。

### 以開發者的角度
從開發者的角度來說Solana Pay的API串接快速容易，不用針對各國規定調整來實作串金流的部分。搞懂sdk使用方式之後一下子就能整合進系統中，部署上去就能直接使用。

Solana本身也支援智能合約。Solana Pay除了單純的transfer request還能透過它執行任何的Solana Transaction (transfer只有交易；transaction是廣義的，包含transfer、智能合約的執行、帳號創建等等）這表示我們也可以做到如在消費者結帳的時候空投代幣給他、自動觸發智能合約進行發貨或是其他服務的提供。我也有看過別人用Solana Pay做一些有趣的小遊戲與應用，如這個[掃qrcode拔河的小遊戲](https://github.com/Woody4618/workshops_fork/tree/main/workshops/tug-of-war)。


### Intro to Solana Pay API
在開始寫程式之前我們需要準備兩個錢包地址用於模擬收發代幣，這邊我用的是phantom wallet。接下來的模擬會在Solana devnet（測試網路）上進行，可以前往wallet中的 設定>開發人員設定>測試網模式 開啟選項。
![alt text](../../phantom-wallet-pic.png)

要匯入模擬用的Solana代幣可以在網路上尋找 Solana faucet
### Dependencies
Solana Pay API 都是基於 js/ts。我們先下載需要用到的套件:  
`npm install @solana/pay bignumber.js @solana/web3.js`

預計使用到的import:
```js
import { PublicKey, Keypair, Connection, clusterApiUrl } from "@solana/web3.js";
import { encodeURL, createQR, findReference } from '@solana/pay';
import BigNumber from 'bignumber.js';
```

### Transfer Flow
![alt text](../../images/solana-pay-transfer-flow.png)
這個是用手機錢包在網站上進行交易的流程圖，其他的交易模式可以參考官方文檔內容。整個交易過程中商家不會知道他在跟誰交易，所以最後會需要一個搜尋並確認有收到款項的動作，透過的方法就是在transfer request url中的reference field留下一個public key，細節可以參考下一段的request url介紹。

基本上一筆Solana Pay的交易內容可以被encode成一個Solana Pay transfer request URL, 格式如下:
```yml
solana:<recipient>
      ?amount=<amount>
      &spl-token=<spl-token>
      &reference=<reference>
      &label=<label>
      &message=<message>
      &memo=<memo>
```

這個url可以被包成一個qrcode或是nfc tag，且不需要靠任何的server運作，只要用錢包掃了qrcode交易就可以直接進行。以下針對幾個特別的fields做介紹:
- recipient: required, 要收錢的錢包地址
- spl-token: optional, 如果想要以SOL以外的代幣交易(如USDC)，需填入spl-token的base58 pulbic key
- reference: optional, 需為base58 encoding，可以用來當作交易識別來確認交易成功。

![alt text](../../images/sol-transfer.png)


### Implementation
透過@solana/pay 提供的api可以容易的將上述種種資訊包裝成request url再包成qrcode。(p.s.如果amount留空則會讓使用者自行輸入支付金額

![alt text](../../images/solana-pay-impl-code.png)

#### 確認交易成功
在成功進行交易後，如上面所述商家需確認有收到款項。這邊我實作了一個簡易的polling，每五秒透過前面reference生成的public key和@solana/web3.js所提供的 findReference()API來檢查等待中的transaction是否成功被放到區塊鏈上(交易成功)。在第二張截圖中可以看到
![alt text](../../images/solana-pay-check-payment-status.png)
![alt text](../../images/solana-pay-payment-success.png)
我們也可以用這個signature直接在solscan的devnet上找到這筆交易
![alt text](../../images/solana-pay-status-solscan.png)

下面影片是將Solana Pay 整合到亞馬遜網路商店購物結帳流程的demo。由於phantom錢包沒有提供USDC測試，後面實際轉帳的流程我改為在devnet上轉0.1SOL代替。

{{< youtube fRSia1JoIr0 >}}  
<br/>

不用在結帳的時候到處翻找你的信用卡，不再需要麻煩的信用卡資訊填寫，敏感資料因商家保管不當/被駭而外流的機率直接消失，跟LINE Pay一樣方便更無須擔心海外消費不支援，實際感受一次是不是覺得超級方便的呢?

以上就是本次介紹，其他實作如空投、Solana Pay tx request、Handle Transfer Request as Wallet Provider等等，礙於篇幅不再贅述。希望這篇文章能讓你對Solana Pay有初步的認識


API文件參考:
- https://docs.solanapay.com/api/core  
- https://docs.solanapay.com  

文中範例程式碼:  
- [amazon clone integration](https://github.com/patty111/amazon-clone-SOLPay)
