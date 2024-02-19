---
sidebar_position: 2
---

0、前言
------
:::tip
**寫此篇文章的動機**  
這一篇紀錄 cronjob-v2 搬家的流程
:::


## checkProductExpire - async

不過用前一個 async 方法，會遇到另外一個問題，就是如果同一個 ip 對一間客戶同一時間打太多 request，他們的網頁的 status code 會回傳 429。   

因此現在我們要來試看看批量請求，能不能解決這個問題




```py
#! /usr/bin/env python
# -*- coding: utf-8 -*-


from django.utils import timezone
from shared.modules.db.orm.models import NewDBProduct, OldDBProduct, Availability
import logging
import time
import aiohttp
import asyncio
import xml.etree.ElementTree as ET
from asgiref.sync import sync_to_async
import requests


async def fetch_url_status(session, url):
    async with session.get(url) as response:
        return url, response.status

async def batch_request(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url_status(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
        return results
    
def test():
    url = "https://api.tagtoo.com.tw/v2/batch_request?link=https://www.tfmmall.com/products/28245&link=https://www.tfmmall.com/products/27585sale?gad_source=1&link=https://www.tfmmall.com/products/57617&link=https://www.tfmmall.com/products/suqqu-eyeshadow-brush-f-large-2022&link=https://www.tfmmall.com/products/57606?gad_source=1&link=https://www.tfmmall.com/products/35047"
    
    reUrls = re.findall(r'link=(.*?)&', url)

    print(reUrls, "======")

    results = asyncio.run(batch_request(reUrls))

    for url, status in results:
        print(f"URL: {url}, Status Code: {status}")

```




poetry 插件檔案
```toml
[tool.poetry]
name = "checkout-expect-product"
version = "0.1.0"
description = "check product page status code, if status is not 200, it is considered expired"
authors = ["John <john0526@tagtoo.com>"]

[tool.poetry.dependencies]
python = "^3.9"
requests = "^2.31.0"
beautifulsoup4 = "^4.12.2"
cron-jobs-v2-shared = {path = "./shared/"}
aiohttp = "^3.9.3"
asgiref = "^3.7.2"
asyncio-throttle = "^1.0.2"


[tool.poetry.dev-dependencies]
ipython = "^8.16.1"

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
```













