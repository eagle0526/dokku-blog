---
title: cronjob-v2-sousou翻譯-part4
sidebar_label: "sousou-translation-part4"
description: sousou 的日文產品 title, description 翻譯
last_update:
  date: 2024-04-16
keywords:
  - chatGPT
  - sousou
sidebar_position: 4
---


0、前言
------
:::tip
**寫此篇文章的動機**  
最後一篇展示所有的程式和架構
:::


## part4 - 最終架構和程式

### 架構

```md
├── GPT_prompt.py
├── main.py
├── pyproject.toml
```

ps. `pyproject.toml` 就不貼了，跟一開始的一樣

### 詳細資訊

1. GPT_prompt.py - 此檔案放 `GPT API` 的設定 + prompt

```py title="GPT_prompt.py"
import json

from openai import OpenAI


def chatGPT(prompt):
    API_KEY = "輸入在 openAI 那邊拿到的 token"
    client = OpenAI(api_key=API_KEY)
    model = "gpt-3.5-turbo-0125"
    temperature = 0.2
    max_tokens = 3500

    chat_completion = client.chat.completions.create(
        model=model,
        temperature=temperature,
        max_tokens=max_tokens,
        messages=[
            {
                "role": "user",
                "content": prompt,
            },
            {
                "role": "system",
                "content": "You are an expert Japanese translator and are proficient in Traditional Chinese and Taiwanese culture",
            },
        ],
    )
    # chat_completion.json() 這個會先轉成字串，json.loads 這個才會轉成 dic
    json_data = json.loads(chat_completion.json())
    content = json_data["choices"][0]["message"]["content"]
    return content


def get_chinese_translated(product):
    translated_sentence = chatGPT(f"""
                            I will provide you with some Japanese product names and descriptions, format like this [('product_key', 'Japanese title', 'Japanese description'),('product_key', 'Japanese title', 'Japanese description')]. please help me to translated the content into Traditional Chinese that is suitable for Taiwanese people to understand, and the returned format like this [('product_key', 'Translated Chinese title', 'Translated Chinese description'),('product_key', 'Translated Chinese title', 'Translated Chinese description')].

                            In addition, please note that some Japanese Chinese characters involve proper nouns. Although the literal translation is not wrong, Taiwanese people may not understand them, including but not limited to the following two examples: please translate 濃紺 into 深藍色, 急須 into 茶壺. Please make your own analogy based on your understanding of Taiwanese and Japanese culture.

                            Finally, please note that Japanese katakana and hiragana do not need to appear in the translated content, because Taiwanese people cannot understand it.                                         
                                       
                            Need be translated sentence : {product}
                                """)
    return translated_sentence
```


2. main.py - 主要執行資料的 i/o 批量處理

```py title="main.py"
"""
1. 抓出新資料庫的資料
2. 用 https://db-feed.tagtoo.com.tw/translations 判斷有哪些產品已經翻譯好
3. 還沒翻譯好的丟進 GPT 翻譯
4. 最後把翻譯好的資料存進 新/舊 資料庫
5. 每三天會更新一次產品
"""

import logging
import time

import requests
from django.utils import timezone
from GPT_prompt import get_chinese_translated
from shared.modules.db.orm.models import NewDBProduct, OldDBProduct

logger = logging.getLogger(name="main.py")
logger.setLevel(logging.INFO)
logger.addHandler(logging.StreamHandler())


def read_newDB_and_update_translate_table():
    logger.info("Start updating old/new DB")
    start_time = time.perf_counter()
    product_counts = 0

    CH_products = NewDBProduct.objects.filter(
        ec_id=1878, lang="zh", expired_at__gt=timezone.now()
    ).order_by("-expired_at")

    logger.info(f"all chinese products: {CH_products.count()}")

    """
    判斷哪些資料已經翻譯好，沒翻譯好的資料在篩選出來
    """
    translated_keys = get_translated_products_by_api()
    not_yet_translated_products = [
        item for item in CH_products if item.key not in translated_keys
    ]
    logger.info(f"not yet translated products: {len(not_yet_translated_products)}")

    # 3 divided into piles, because gpt can't translate accurately one time if the qty too much
    batch_size = 3
    product_groups = [
        not_yet_translated_products[i : i + batch_size]
        for i in range(0, len(not_yet_translated_products), batch_size)
    ]

    for product in product_groups:
        keys = [item.key for item in product]
        JP_products = NewDBProduct.objects.filter(
            ec_id=1878, key__in=keys, lang="ja"
        ).values_list("key", "title", "description")
        product_counts += JP_products.count()

        processed_JP_products = [
            (key.replace("product:jp", "product"), title, description)
            for key, title, description in JP_products
        ]
        translated_sentence = get_chinese_translated(processed_JP_products)

        try:
            new_array = eval(translated_sentence)
            for item in new_array:
                if len(item) != 3:
                    logger.error("Invalid item in translated_sentence: {}".format(item))
                    continue
                key, translated_keys, translated_description = item
                for key, translated_title, translated_description in new_array:
                    # oldDB & newDB
                    update_old_db(key, translated_title, translated_description)
                    update_new_db(key, translated_title, translated_description)
        except Exception:
            logger.error(
                f"Update Error while evaluating translated_sentence: {translated_sentence}"
            )

    remain_products = len(not_yet_translated_products) - product_counts
    logger.info(
        f"update product counts: {product_counts}, the remaining products not yet been translated: {remain_products}, total cost time: {time.perf_counter() - start_time:0.2f}"
    )


def get_translated_products_by_api():
    try:
        url = "https://db-feed.tagtoo.com.tw/translations"
        params = {"ec_ids": "1878"}
        response = requests.get(url, params=params)
        translated_data = response.json()
        translated_keys = set(item["key"] for item in translated_data)
        return translated_keys
    except Exception as e:
        logger.error(
            "happen error when get https://db-feed.tagtoo.com.tw/translations: {}".format(
                e
            )
        )
        return False


def update_old_db(key, translated_title, translated_description):
    try:
        revised_key = "sousou:product:" + key
        final_product = OldDBProduct.objects.filter(
            advertiser_id=1878, product_key=revised_key
        ).first()

        final_product.extra["translated_title"] = translated_title
        final_product.extra["translated_description"] = translated_description
        final_product.save()

        return True
    except Exception as e:
        logger.error("An error occurred while updating the old database: {}".format(e))
        return False


def update_new_db(key, translated_title, translated_description):
    try:
        url = "https://db-feed.tagtoo.com.tw/translations"
        product_translation_data = {
            "ec_id": 1878,
            "key": key,
            "lang": "zh",
            "currency": "JPY",
            "translated_title": translated_title,
            "translated_description": translated_description,
        }

        response = requests.post(url, json=product_translation_data)
        if response.status_code == 200:
            new_translation = response.json()
            logger.info(
                f"Product successfully created new translation: {new_translation}"
            )
            return True
        else:
            logger.error(
                f"Product failed created new translation: {response.status_code}"
            )
            return False

    except Exception as e:
        logger.error("An error occurred while updating the new database: {}".format(e))
        return False


if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO)
    read_newDB_and_update_translate_table()
```


