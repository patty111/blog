---
title: "Risk in token 2022 CPI: CPI Guard and Arbitrary CPI"
date: 2024-08-06T15:50:24+08:00
tags:
 - blockchain
 - solana
 - CPI
 - TokenExtensionProgram
 - XueDAO
draft: true
---
> [Original Post for Solana Writeathon](https://vocus.cc/article/66afe9e6fd897800017851b4)

CPI 全名叫做 Cross Program Invocation，它可以讓Solana的program直接呼叫另一個Program中的instruction，這個功能可以讓程式有更高的可組合性。可以把CPI想像成一個API在被呼叫的時候又去呼叫另外一個API。

![alt text](../../images/cross-program-invocation-graph.png)

## CPI Guard
在Token 2022中新增了一個extension叫做 CPI Guard，它的作用是預防某些濫用CPI的行為。聽起來有點抽象，那麼實際上是在預防什麼呢？

基本上當user使用dapp時有三種方法轉移user資產到dapp端：

1. 在transaction中加入transfer instruction
2. 在transaction中加入一個 approve instruction，隨後使用Program的授權進行CPI transfer
3. 在transaction中加入一個不透明指令，在使用者授權之下進行CPI transfer  

前兩種方式是安全的，user都會知道program想對他做什麼。

第一種相當於直接跟使用者說：“商品 10 元請付錢”; 第二種相當於我們去餐廳結帳時將信用卡給服務生，他們只能刷我們餐費的金額; 第三種方式是問題的所在，因為Program(dapp) 後面的CPI call是不透明的，使用者如果又對program進行授權，program將可以對user的token進行任意操作，相當於你將信用卡交給路人。

CPI Guard禁止了一系列跨program間instruction的呼叫來避免這類濫用發生。

## Arbitrary CPI
雖與CPI Guard不同，但提到CPI濫用攻擊時這也是重要的一個安全性議題，在此一起討論。

>可先參考這一篇了解Solana Account Model: [SPL Token & Token Ext](../spl-token--token-ext)  

在進行CPI呼叫的時候通常會傳入一個Program ID來代表目標地址，但我們無法確保這個地址的Program真的是我們想要使用的Program。這個呼叫可能被攻擊者導向其他惡意程式，產生未預期的執行結果。

以這個[範例](https://arc.net/l/quote/zugnqyzx)來說:
```rust
#[program]
pub mod arbitrary_cpi_insecure {
    use super::*;

    pub fn cpi(ctx: Context<Cpi>, amount: u64) -> ProgramResult {
        solana_program::program::invoke(
            &spl_token::instruction::transfer(
                ctx.accounts.token_program.key,
                ctx.accounts.source.key,
                ctx.accounts.destination.key,
                ctx.accounts.authority.key,
                &[],
                amount,
            )?,

            ...

        )
    }
}
#[derive(Accounts)]
pub struct Cpi<'info> {
    source: AccountInfo<'info>,
    destination: AccountInfo<'info>,
    authority: AccountInfo<'info>,
    token_program: AccountInfo<'info>,
}
```
這個program的第六、七行用cpi呼叫了token program裡面的 [transfer instruction](https://github.com/solana-labs/solana-program-library/blob/58221fc9ae05e258ee903b49c5e8f5abbeb7796c/token/program/src/instruction.rs)。

```rust
// transfer instruction​
pub fn transfer(
    token_program_id: &Pubkey,
    source_pubkey: &Pubkey,
    destination_pubkey: &Pubkey,
    authority_pubkey: &Pubkey,
    signer_pubkeys: &[&Pubkey],
    amount: u64,
) 
```
但是我們無法保證第24行 `token_program` 的 Account 在傳入時真的是 SPL Token 的帳戶地址​。避免利用這個漏洞的攻擊發生開發者應該加一段針對地址正確性的檢查動作。

讀者可以在這一個[commit](https://github.com/solana-labs/solana-program-library/commit/58221fc9ae05e258ee903b49c5e8f5abbeb7796c)中看到commiter新增了 spl-token program id 檢查 -- 一個`check_program_account()` 函式來確保 Program ID 真的代表使用者預期呼叫的Program。

## Reference
- [CPI Guard實作提議原始討論串](https://github.com/solana-labs/solana-program-library/issues/3694)
- [PR: token-2022: implement CpiGuard](https://github.com/solana-labs/solana-program-library/pull/3712)
