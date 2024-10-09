---
title: "SPL Token & Token Ext"
date: 2024-08-04T16:01:17+08:00
tags:
 - blockchain
 - solana
 - TokenExtensionProgram
 - XueDAO
draft: false
---
> [Original Post for Solana Writeathon](https://vocus.cc/article/66af4edcfd8978000110d57d)
# Account Model
SPL 是 Solana Program Library 的簡稱，是一個用來在 Solana 上創造 fungible/non-fungible token 的工具，內含多個Program Account。

有別於以往在 Ethereum 上，每當想創造新的 token 就需要寫一個智能合約，SPL有點像是API的概念，透過呼叫 token library 讓 token 創建的過程標準化。
### Program Accounts & Data Accounts
前面講到的 Program Account 是什麼？這邊將簡單介紹一下 Solana 鏈上 Account 的區別與從屬架構。

Solana 就如同其他區塊鏈一樣，有許多個 address 在它的鏈上，每一個這樣的地址叫做一個 Account。有別於其他大部分的區塊鏈，Solana 完全的將程式（智能合約）與靜態資料（如錢包裡的$$，你持有的 token 數量等等）分開來儲存。這兩種 Account 就分別叫做 **Program Account** & **Data Account**。
### Programs
Program 又可以做細分，其中一種叫做[Native Program](https://docs.solanalabs.com/runtime/programs)（原生程式（？）），它提供了 Solana 網路底層的功能實現，底下還有更多 program 如 **System Program** & **BPF Loader Program**。

System Program 可以進行帳號創建，各位可以在 solscan 上觀察自己的錢包owner，通常顯示的都會是 system program。
![alt text](../../images/show-system-program-solscan.png)  
BPF 是其他除 Native Program 以外的 Program 的 owner。可以部署、更新、執行客製化 Program 等等。我們的主角 Program Library 說的 Program 也在此列，叫做 Token Program。
### 創造代幣 Account 間的關係
假設我想要創建一個新的 token 叫做 “XD”，我需要一個帳號用來代表我的Token -- 稱之為Mint Account。

Mint Account 會給予另外一個錢包地址鑄造代幣的權限（Mint Authority），這裡我用User Account代表。User Account需要建立一個獨立的 account 來存放XD代幣，稱之為 Associated Token Account，有別於你的錢包地址ownership指向System Program， Associated Token Account會指向你的錢包。如一個Solana錢包地址存放USDC等token的概念。

當想要鑄造代幣，會用擁有 mint authority 的帳戶向 Token Program 發送一個 transaction。Token Program 會驗證Account 是否有權限鑄造代幣，如果驗證通過 transaction 就會被執行。

下圖為整個流程中帳戶之間的關聯圖:

![alt text](../../images/token-accounts-relations.png)  

# Mint Tokens using SPL Program
要在 Solana 上鑄造新的 token 需要使用到token program，功能較為陽春。隨著各種新需求的增加同時兼顧安全性的目的，一個新的 token program - **token 2022** （另名token extension）被開發出來，發佈在一個新的地址上。  
  
Token 2022 在兼顧安全性的同時新增了非常多實用且方便的功能如支援鏈上 metadata，transfer hook，transfer fees等等。

這些差異的比較與實作將在接下來幾篇中提到。為了更深刻理解演化的過程並感受差異，本篇將先帶領讀者用舊版的 token program 搭配 **solana cli** & **spl-token cli** 在 Solana devnet 上鑄造一個新的代幣。
## 下載工具
Run in terminal:
```bash
sh -c "$(curl -sSfL https://release.solana.com/stable/install)"
```
視輸出提示決定是否需要更新 PATH 環境變數。([官方doc](https://docs.solanalabs.com/cli/install))
```bash
// 進入專案資料夾​
mkdir ~/new-token
cd ~/new-token
```
## 準備Account
> 1個Mint Account, 2個 User Account(one for showcasing transfer token)
### Create User Account
```bash
solana-keygen new -o user1.json	// this user will be granted mint authority
solana-keygen new -o user2.json
```
### Solana CLI setting
我們要將user1設定為接下來用於簽章的帳戶，同時先空投SOL用於後續的交易手續費。
```bash
// 設定預設使用的帳戶
solana config set --keypair user1.json
// 空投 5 SOL
solana airdrop 5 user1.json
```
### Create Mint Account
為了方便表示，我將用 `$XD_TOKEN` 代表 new token address。
```bash
spl-token create-token --mint-authority user1.json

// output
Creating account $XD_TOKEN
```
這樣新的token就建立成功了，

這是在solana explore上面我的Token目前的資訊：
![alt text](../../images/solana-token-with-SPL-program.png)
也可以使用指令查看帳戶資訊：
```bash
solana account $XD_TOKEN

// output
Public Key: ...
Balance: ...
Owner: ...
Executable: false
Rent Epoch: ...
Length: ...                                    ..
```
### Mint Token
是否還記得每一個token類別都需要一個專門的帳戶來儲存他，我們需要在User1帳戶底下建立一個Asociated Token Account (ATA)，我會用$ATA_ACC1來代表。

```bash
spl-token create-account $XD_TOKEN --owner $USER1
// output
Creating account $ATA_ACC1
```
```bash
$ spl-token mint $XD_TOKEN 10 $ATA_ACC1 --mint-authority user1.json
```

User1 Account:
![alt text](../../images/user1-account-info.png)
Token Mint Account:
![alt text](../../images/token-mint-account-info.png)
以上均可看到supply變成1000了。

### Token Transfer
```bash
spl-token transfer $XD_TOKEN 100 user2.json --owner user1.json
```
如果直接跑這一行會發現跳出錯誤，這是因為我們也需要為user2建立一個ATA。除了這個方法以外我們也可以在指令末端加入 --fund-recipient，這將由sender出手續費幫接收者建立一個ATA。

---

到這邊應該都很簡單，但是各位是否有發現我的Token名稱還是Unkown Token，也沒有好看的圖片，其他metadata也全無。這是因為在token-2022前要先在外部儲存空間中將metadata上傳，並利用如 metaplex 等第三方工具讓Token指向這些資料連結。

開發者需要額外去熟悉metaplex js庫的語法，搞定外部儲存空間等等麻煩的操作。token-2022讓metadata上鍊不僅將流程簡化，也避免過度依賴第三方系統造成速度、安全、穩定性等影響。

p.s. 網路上有一些廣告主打用他們的no-code工具來mint token，一個收你0.1~0.5 SOL ：）根本搶錢  

# Token Extension Program
## Intro
Token Extension Program（以下簡稱TEP） 原名 Token 2022，是一個基於原本Token Prgram（以下簡稱TP）的擴充版本。TEP被部署到的地址跟TP是不同的，所以不能單純將他當作是一個更新版。TEP不能完全取代TP的存在。  

Solana doc的例子很好的舉例為什麼需要用到TEP：
> 假設一所大學想要將畢業證書用 NFT 的方式發行，要如何確保這個畢業證書永遠不會被轉移給其他人（學位不可轉讓）？在當前的 TP 中是不可能的，開發者需要在transfer指令中加入檢查並拒絕所有的transfer。此問題的一個解法fork一個大學專屬的 TP，並添加這個檢查。但如此一來這個 TP 就會是完全獨立的。大學需要為錢包和用於進行學位檢查的 dApp 提供維護。而如果不同的大學有各自的客製化需求，那麼就會有更多個fork。在 TEP 上線後可以直接使用其中的 **non-transferable token extension**。
## Extensions
依Account分成兩類Extension，Account Extensions & Mint(Account) Extensions。這些extension的資訊會被加在  
`Account` 的尾端。
### 結構
如上面敘述，TPE 是擴充版本，所以大致上的架構仍沿用原版TP。

參考這兩段程式碼，Mint Account的格式，可以觀察到前面的byte結構都一樣：
> [Token Program code @ line 1200](https://github.com/solana-labs/solana-program-library/blob/4064124578e55a098243c6209d23e23270c5bccd/token/program/src/processor.rs) & [Token Extension Program code @ line 1566](https://github.com/solana-labs/solana-program-library/blob/master/token/program-2022/src/extension/mod.rs#L65)
擴展部分新增的資料欄位有：
- padding
- Account Type: 三種，尚未被初始化、Mint、一般Account
```rust
// Extension program code line 1025
pub enum AccountType {
    /// Marker for 0 data
    Uninitialized,
    /// Mint account with additional extensions
    Mint,
    /// Token holding account with additional extensions
    Account,
} 
```
- Extension Type: 使用擴充套件的種類，可參考源碼內容(line 1046)
- length: extension 的長度
- data: 額外的資訊
以下將對兩種裡面的幾個extension進行介紹。
### Mint Extensions
- Non-transferable tokens: 如最上面的例子，代幣禁止轉移
- transfer fees: 讓token transfer被抽取額外的手續費（不是gas fee），以Token-2022發行的第一個代幣 -- BERN，就是使用這個extension
- permanent delegate: 發行者可以永久控制代幣權限，可用於如註銷非法帳號中的資產。權限過大且危險。
- confidential transfer: 進行transaction的時候可以隱藏如交易數量等資訊，適合B2B交易或是公司支付員工薪水
- transfer hook: 類似webhook，每當交易就會被觸發背後的邏輯被執行。可應用在上面文憑例子中的交易檢查，或是NFT的分潤機制。
- metadata & metadata pointer: 兩者通常都會同時出現，可將任何資料存放在代幣上，如前面幾篇所說讓metadata上鏈並簡化設定方法，進行標準規範
### Account Extensions
- Immutable ownership: Account owner不可變動，前面有提到Account均由System Program建立，而像是ATA(Associated Token Account)會在建立後將ownership轉移給你的錢包地址。這個Extension可以加強安全性的防護，所有Token 2022的ATA都預設啟用這個extension。
- CPI Guard: 禁止特定跨Program的程式呼叫，細節可參考 [CPI Guard](../risk-in-cpi-cpi-guard-and-arbitrary-cpi/)
- memo: 有incoming transfer的時候需要寫備註，就像是銀行轉帳會顯示在存摺上的備註
# Token Extension Program Application
接下來要用 Token-2022 和 solana cli 來mint a new token 叫做TYC。
如上述，Token-2022 與 Spl Token 是不同的 Program，所以在呼叫的時候要特別註明以示明區別。
> Token-2022 Program ID 地址: TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb

接下來的指令需要附上一個 --program-id flag，後面加入 token-2022 program Id的地址，我就不再一一附上。
## Mint Account
```bash
// 新增一個帳戶當作mint account
solana-keygen new -o mint.json
```
用Token-2022建立一個新的Mint Account，Extension的部分我開啟了:
- metadata 與 metadata pointer
- transfer fee
- close authority
```bash
spl-token create-token \
--program-id TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb \
--enable-metadata \
--enable-close \
--transfer-fee 200 5000000000 \
mint.json
```
使用方法可以去參考[官方extension的文檔](https://spl.solana.com/token-2022/extensions)
## 建立 User 帳戶的 ATA
```bash
spl-token create-account mint.json --owner user.json
```
## Mint Tokens
```bash
spl-token mint mint.json 10000
```
可以點進去觀察 spl token 跟 token-2022 metadata 豐富度有很明顯的差別，在Solana Explorere上也可以看到標示出兩者是使用不同 Program 建立的。

![alt text](../../images/spl-token-program-created-token-mint-account.png)
<p align="center"><em>spl program created mint account</em></p> 

![alt text](../../images/token2022-created-token-mint-account.png)
<p align="center"><em>token2022 created mint account</em></p>

## Transfer Result
下圖是一個transfer fee 2%，上限5 token轉移 10000 個代幣的交易紀錄，中間被扣除的 5 個 token 就是因為有設定 transfer fee extension 被扣掉的
![alt text](../../images/transfer-result-token-balance.png)
## Adding Metadata
下面的 json 檔案就是我要設定的metadata。要把這個檔案跟想用來當 token 標示的圖片檔放到一個可以公開儲存的空間，這邊我用的是 [web3 storage](https://console.web3.storage/)，不需付費。需注意 json 檔與圖檔的連結需是一點開就可直接存取檔案的。 [example link](https://bafybeihgavlj5qg2lf2glcaujz2435cmwnvhcwpdjuvin5vehzc3uy7o6i.ipfs.w3s.link/metadata.json)
![alt text](../../images/web3storage-metadata-example.png)
<p align="center"><em>token2022 created mint account</em></p>

接下來用下面的指令初始化 metadata
```json
spl-token initialize-metadata mint.json 'TYC' 'TYCC' \
https://bafybeihgavlj5qg2lf2glcaujz2435cmwnvhcwpdjuvin5vehzc3uy7o6i.ipfs.w3s.link/metadata.json
```

等個半分鐘應該就可以看到新增上去的資料: [TYC TOKEN](https://explorer.solana.com/address/4mZd5sSi1gu8gUXB7dhWxBVVwUzrGvrqz2r8qupwwD1X?cluster=devnet)

Solscan & Solana Exporer 上圖檔跟一些詳細的資訊會比較慢才會更新上去，Solana FM 比較快會被更新，可以先在這邊查看。
