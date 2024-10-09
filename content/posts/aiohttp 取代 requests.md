---
title: "AioHttp 取代 Requests"
date: 2023-11-17T04:47:56+08:00
tags:
 - python
 - backend
 - web
draft: false
---
```python
async def main():
    async with aiohttp.ClientSession() as session:
        collected_data = []
        depth = 0
        max_depth = os.getenv('MAX_DEPTH')
        INITIAL_ADDRESSES = [#some addresses here]

        tasks = [process_transactions(session, address, API_KEY, depth, max_depth, collected_data) for address in INITIAL_ADDRESSES]
        await asyncio.gather(*tasks)

        save_to_csv(collected_data)
```