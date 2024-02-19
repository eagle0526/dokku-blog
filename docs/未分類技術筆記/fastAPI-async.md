---
sidebar_position: 2
---

0、前言
------
:::tip
**寫此篇文章的動機**  
這裡整理如何使用 async 
:::


## async 使用

1. 我這邊列出了 10 個連結
2. 接著同時對 10 間廠商進行 request，查看網頁的 status (session.get(url))
3. 這時候順便調查每個網址的 request 時間


```py
from typing import Optional
from fastapi import FastAPI

import requests
import bs4
import pandas as pd
import urllib.parse
import time

async def fetch(url, session):
    start_time = time.time()
    async with session.get(url) as response:
        end_time = time.time()
        return url, response.status, end_time - start_time
    
@app.get("/get_info/{PID}")
async def get_info(PID: str):

    total_time = 0.0

    url_list = ["https://www.facebook.com/", 
                "https://www.google.com/", 
                "https://www.youtube.com/", 
                "https://www.tfmmall.com/products/03051", 
                "https://www.tfmmall.com/products/03775", 
                "https://www.tfmmall.com/products/28152e0084", 
                "https://www.tfmmall.com/products/51079",
                "https://www.tfmmall.com/products/32037", 
                "https://www.tfmmall.com/products/36593qu083", 
                "https://www.tfmmall.com/products/33586", 
                "https://www.tfmmall.com/products/35580",
                "https://www.tfmmall.com/products/32541",
                "https://www.tfmmall.com/products/32532",
                "https://www.tfmmall.com/products/01889"]

    print("step1", "**********")
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(url, session) for url in url_list]
        results = await asyncio.gather(*tasks)

        print("step2", "**********")
        for url, status_code, elapsed_time in results:
            print(f"URL: {url}, Status Code: {status_code}, Time: {elapsed_time:.2f} seconds")
            total_time += elapsed_time                        

        print(f"Total time: {total_time:.2f} seconds")        

    return {"user": PID}
```





















