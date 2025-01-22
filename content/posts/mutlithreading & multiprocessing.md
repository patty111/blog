---
title: "Mutlithreading & Multiprocessing"
date: 2024-10-16T05:55:57+08:00
tags:
 - python
 - OS
draft: false
---

### Concurrency vs Parallelism
中文可以翻譯維 “並發” 與 “並行”
Concurrency 是指一個系統同時可以處理多個任務，任務間可以相互切換，但是不是同時進行。Parallelism 是指一個系統同時可以處理多個任務，任務間可以同時進行。而常見的 async programming 或 multi-threading 都是屬於 concurrency，很好的解決了 blocking 的問題。

舉一個例子在 js 中傳送 request 是 I/O 類型的任務，整個 process 會被 block 住直到收到 response。利用 async/await 就可以在等待 response 的同時，繼續執行其他的任務。  
Parallelism 的部分就舉 FastAPI deployment 為例。FastAPI 本身是支援 async programming 的，但是在 deployment 的時候，通常會用 gunicorn 來啟動多個 uvicorn worker。每一個worker都是獨立的 process，達到 parallelism 的效果。

