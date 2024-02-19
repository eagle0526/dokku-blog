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

現在用非同步的方式，一開始先用 rss，可以得到目前資料庫所有的產品，接著用 `fetch_url_status` 方法，打 rss 中所有產品的連結，確認是否 `status = 200`。
如果不是200，就把該產品的 `id` ，丟進 `pks_has_expired` 陣列裡面

最後用 `sync_to_async` 這個插件，讓我們可以把原本同步的資料庫，變成異步執行，這樣就可以修改資料庫裡面的資料了



```py
from django.utils import timezone
from shared.modules.db.orm.models import NewDBProduct, OldDBProduct, Availability
import logging
import time
import aiohttp
import asyncio
import xml.etree.ElementTree as ET
from asgiref.sync import sync_to_async
import requests


async def fetch_url_status(url, session):    
    async with session.get(url) as response:        
        return url, response.status

async def read_api(EC_ID):    
    # api_url = (f"https://api.tagtoo.com.tw/v2/feed/facebook/{EC_ID}")
    api_url = (f"https://api.tagtoo.com.tw/v2/feed/facebook/{EC_ID}?begin_value=0&scope=10")
    namespaces = {"g": "http://base.google.com/ns/1.0"}
    response = requests.get(api_url)    
    root = ET.fromstring(response.content)    
    products = root.findall(".//item")

    data_sets = []
    pks_has_expired = []        
    for product in products:
        id = product.find("./g:id", namespaces).text
        link = product.find("./g:link", namespaces).text
        product_set = (id, link)
        data_sets.append(product_set)        
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url_status(url, session) for _, url in data_sets]
        results = await asyncio.gather(*tasks)        
        for (id, url), (url, status) in zip(data_sets, results):            
            if status != 200:                
                pks_has_expired.append(id)
            print(f"URL: {url}, Status Code: {status}")
            await asyncio.sleep(2)

    await update_old_db(EC_ID, pks_has_expired)
    await update_new_db(EC_ID, pks_has_expired)


@sync_to_async
def update_old_db(EC_ID, pks_has_expired):    
    if pks_has_expired:
        three_days_ago = timezone.now() - timezone.timedelta(days=3)        
        products = OldDBProduct.objects.filter(advertiser_id=EC_ID, product_key__in=pks_has_expired)        
        # products.update(live=False, expire=three_days_ago)        
        print(products, "**old**")

@sync_to_async
def update_new_db(EC_ID, pks_has_expired):    
    if pks_has_expired:            
        three_days_ago = timezone.now() - timezone.timedelta(days=3)        
        pks_has_expired_revised = [item.split(":")[-1] for item in pks_has_expired]
        products = NewDBProduct.objects.filter(ec_id=EC_ID, key__in=pks_has_expired_revised)        
        # products.update(availability=Availability.OUT_OF_STOCK, expired_at=three_days_ago)    
        print(products, "**new**")


if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO)

    EC_IDs = (
        1784,
    )
    for EC_ID in EC_IDs:
        asyncio.run(read_api(EC_ID))
```



















