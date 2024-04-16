---
sidebar_position: 2
---

0、前言
------
:::tip
**寫此篇文章的動機**  
這一篇紀錄串接 chatGPT 和 翻譯資料庫中的日文產品並重新會匯入
:::


## sousou-product-translate - part1 - 先讓程式動起來

### 需求
幫 SOUSOU 做中文商品資料翻譯，技術面想先確認如何確認某個商品翻譯過了？   
討論出的機制：中文商品進來先去查有無對應的日文商品，有的話便將該商品的日文文案透過 ChatGPT 翻譯，並覆蓋原中文商品內容    
若已有中文的商品則不更新，僅新增那些還不存在的，爾後每三天跑排程全部更新一次     
title + description 研究看看能不能包在一包發，以節省 API call 次     


### 實際流程

1. 先抓出日文商品
2. 把日文商品的 `product_key, title, description` 用 `list` 整理好
3. 所有商品開始跑迴圈，並把 `日文商品的 product_key` 改成 `中文商品的 product_key`
4. 用剛剛修改的中文商品 `product_key` 跑 ORM 抓出對應的中文商品
5. 如果中文商品存在的話，把日文商品的標題和描述丟進 chatGPT 中一起翻譯
6. 把 GPT 翻譯好的中文丟進中文產品的 extra 欄位


### 以下是最初版的寫法，會有很多問題
```py
import logging
import requests
from shared.modules.db.orm.models import NewDBProduct, OldDBProduct
from openai import OpenAI
import json
import re

def chatGPT(prompt):
    API_KEY = 'api_key'
    client = OpenAI(api_key=API_KEY)
    model = "gpt-3.5-turbo-16k"
    temperature = 0.2
    max_tokens = 5000
    chat_completion = client.chat.completions.create(
        messages=[
            {
                "role": "user",
                "content": prompt,
            }
        ],
        model=model,
        temperature=temperature,
        max_tokens=max_tokens,
    )
    # chat_completion.json() 這個會先轉成字串，json.loads 這個才會轉成 dic
    json_data = json.loads(chat_completion.json())
    content = json_data['choices'][0]['message']['content']
    return content




def old_database_translate():
    JP_products = OldDBProduct.objects.filter(
        advertiser_id = 1878,
        product_key__icontains = 'jp'
    )

    JP_data_sets = JP_products.values_list('product_key', 'title', 'description')
    # JP_data_sets 會有所有的日文商品，因此先用 2 個商品跑迴圈試試
    for item in JP_data_sets[1:2]:
        CH_product_key = item[0].replace("jp:", "")        
        CH_products = OldDBProduct.objects.filter(
            advertiser_id = 1878,
            product_key = CH_product_key
        )

        if CH_products.exists():
            translated_sentence = chatGPT(f"這是該產品的標題: {item[1]}，這是該產品的描述: {item[2]}。 請將標題和描述從日文翻譯成繁體中文，傳回給我的 '標題' 和 '描述' 請用 | 幫我分開")
            sentence_without_spaces = re.sub(r'\s+', "", translated_sentence)            
            pattern = r"標題：(.+?)描述：(.+)"
            matches = re.match(pattern, sentence_without_spaces, re.DOTALL)
            translated_title = matches.group(1).strip()
            translated_description = matches.group(2).strip()

            for CH_product in CH_products:
                extra_data = CH_product.extra if CH_product.extra else {}
                extra_data['translated_title'] = translated_title
                extra_data['translated_description'] = translated_description
                CH_product.save()

```




### poetry 插件檔案
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













