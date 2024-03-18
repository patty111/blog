---
title: "Simple Introduction to CORS"
date: 2024-03-19T02:17:27+08:00
tags:
 - web
draft: false
---
最近在接一個外包的案子的時候遇到了一個error message  
> Access to fetch at ‘https://some-web-service/' from origin ‘http://my-frontend-page' has been blocked by CORS policy: No ‘Access-Control-Allow-Origin’ header is present on the requested resource. If an opaque response serves your needs, set the request’s mode to ‘no-cors’ to fetch the resource with CORS disabled.  

先簡單介紹我的web app架構：由fastapi & react 組成前後端，使用者在前端登入的時候會需要透過一個第三方服務進行類似OAuth的驗證。之前維護的工程師把trigger OAuth的 request 寫在前端送出，形成了這個錯誤。  
### What is CORS?
CORS (Cross Origin Resource Sharing) 是一種用http header使正在瀏覽網站的user-agent可以存取其他來源(origin, 網域)的方法。所謂的同源不單只domain name要一樣，port number也要相同。  
舉例來說，`localhost:3000` 和 `localhost:3001` 就是不同的origin.  

CORS存在的目的是為了防止一些惡意腳本訪問它不應該取得的資源，它會阻擋一切非簡單請求。簡單請求需要符合以下所有條件:  
- one of GET, POST, HEAD methods
- Content-Type value 只能為 text/plain, multipart/form-data、application/x-www-form-urlencoded 之一
- Request Header 不可以存在自定義 Header

如用 GET 取得html, image 或是文字資料就不會被CORS policy 阻擋。
但如 Header 使用 application/json 夾帶json資料就會被擋下來。

正常來說前端發送request到後端的流程如下：  
1. 使用者在前端網頁發送了請求到後端server
2. 如果CORS policy存在，瀏覽器會先送一個preflight OPTIONS請求，去檢查request是否被允許的，如果server回傳正確的CORS header (如Access-Control-Allow-Origin)，瀏覽器就會讓request執行。如果沒有CORS policy檢查，惡意腳本就可以隨意發送request去取得想要的資源，甚至使用者敏感資料，或是代替使用者執行各種操作請求。

![cors-header-example](../../images/CORS-header-example.png)

### How to Resolve?
一般來說只能透過後端解決，要把前端的網域開在一個白名單中，讓他們成為 allow origins。  
以fastapi為例，它提供了一個CORSMiddleware：

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```
上面範例開放全部的網域都可以存取後端資源，但這樣CORS就沒有存在的意義了，稍微修改一下他的scope和origins:  
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

origins = [
    "http://localhost:3000", 
    "https://my-frontend-page"
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["GET", "POST", "OPTIONS", "PUT", "PATCH"],
    allow_headers=["*"],
)
```

以我碰到的專案來說，我們架設的FastAPI server已經將frontend設定為allow origin，是可以存取資源的，但是用來OAuth的第三方server並沒有whitelist我們的前端網域，我們也不可能去修改第三方server的whitelist。  
解法就是把原本直接由前端送出的請求先轉送到FastAPI server，後端再送request去做OAuth，取得token後送回前端。 
![workflow](../../images)