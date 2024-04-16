---
sidebar_position: 2
---

0、前言
------
:::tip
**寫此篇文章的動機**  
這一篇紀錄串接 chatGPT 和 翻譯資料庫中的日文產品並重新會匯入
:::


## sousou-product-translate - part2 - 改進翻譯 + 修改抓取順序

### 需求
(1) `part1` 的寫法是先找出日文商品，再找出對應的中文商品，最後再翻譯，但是這樣會有一點問題，就是今天客戶要求我只要 `找出特定中文商品`，這樣原本的寫法就會有問題，因此在 `part2` 會改進一些寫法      
(2) 再加上原本的 prompt 的翻譯品質太差，經過修改建議後，改成更精準的 prompt


* 修改的地方

1. 使用的翻譯 model version - 這個版本便宜非常非常多
2. GPT 的 api 參數 - messages，多加上這一段 `"role": "system","content": "You are an expert Japanese translator and are proficient in Traditional Chinese and Taiwanese culture"`，這個可以讓翻譯更精準 (讓 GPT 明白自己現在是什麼角色)
3. 翻譯的 prompt 改得更加精準
4. I/O 的流程修改
5. 加上時間計算 - `time.perf_counter()`


### 實際流程

1. 先抓出所有中文商品 - 並且有幾項篩選條件   
(1) 要篩選出 expire > now    
(2) 要抓出沒有翻譯過的商品 (extra 欄位裡面沒有 translated_title 欄位)    
(3) 根據 expire 排序，時間最遠的越前面 (先更新最新被瀏覽的商品)  

2. 用剛剛的抓出的中文產品，用 `product_key` 抓出對應的日文產品，並把日文產品的 `title, description` 整理好


3. 如果中文商品存在的話，把日文商品的標題和描述丟進 chatGPT 中一起翻譯
4. 把 GPT 翻譯好的中文丟進中文產品的 extra 欄位




```py
import logging
from shared.modules.db.orm.models import NewDBProduct, OldDBProduct
from django.utils import timezone
from openai import OpenAI
import json
import re
import json
import time
from django.db.models import CharField
from django.db.models.functions import Cast

def chatGPT(prompt):
    API_KEY = 'api-key'
    client = OpenAI(api_key=API_KEY)
    model = "gpt-3.5-turbo-0125"
    # model = "gpt-3.5-turbo-16k" -> 原本使用的 model 超級貴
    temperature = 0.2
    max_tokens = 3500
    
    chat_completion = client.chat.completions.create(
        model=model,
        temperature=temperature,
        max_tokens=max_tokens,
        messages=[
            {
                "role": "system","content": "You are an expert Japanese translator and are proficient in Traditional Chinese and Taiwanese culture",
                "role": "user","content": prompt,
            }
        ],        
    )
    # chat_completion.json() 這個會先轉成字串，json.loads 這個才會轉成 dic
    json_data = json.loads(chat_completion.json())
    content = json_data['choices'][0]['message']['content']
    return content


def new_method_translated():
    # 現在要先抓出 - 中文 + 還沒翻譯完成 + 還沒過期的商品
    logger.info("Start updating old DB")
    start_time = time.perf_counter()
    CH_products = OldDBProduct.objects.filter(
        advertiser_id=1878,
        link__icontains = 'CHT',
        expire__gt=timezone.now(),
    ).annotate(
        extra_str=Cast('extra', CharField()),  # 將 JSONField 轉換為 CharField
    ).exclude(
        extra_str__contains='"translated_title"'
    ).order_by("-expire")

    print(CH_products.count())
    products_list = []
    # 拿中文 product_key 抓出日文 title, description
    for product in CH_products[3:8]:        
        JP_product_key = product.product_key.replace("product:", "product:jp:")
        jp_title, jp_description = OldDBProduct.objects.filter(
                                        advertiser_id=1878, product_key=JP_product_key
                                    ).values_list('title', 'description').first()
        # 把修改過的 product_key 和 日文 title, description 放進新的陣列中
        products_list.append([product.product_key, jp_title, jp_description])
    
    
    for product in products_list:
        print("************************")
        print("product_key", product[0])
        jp_sentence = f'標題- {product[1]}|描述- {product[2]}'
        print(jp_sentence, "**jp_sentence**")
        translated_sentence = chatGPT(f"""
                                        我會提供你一些日文商品名稱和描述，格式像這樣，標題-'日文原文商品標題' | 描述-'日文原文商品描述'，請幫我翻譯成適合台灣人理解的繁體中文內容，並且回傳的格式像這樣，標題- '翻譯後的中文標題' | 描述- '翻譯後的中文描述'。另外，要請你注意有些日文漢字涉及專有名詞，雖然直譯也沒有錯，但台灣人可能不理解，包括但不限於以下兩個例子：濃紺請翻譯成深藍色，急須請翻譯成茶壺，請按照你對台日文化的理解自行類推。最後，請注意翻譯的內容中不需要出現日文的片假名和平假名，因為台灣人也看不懂。 
                                        要翻譯的句子：{jp_sentence}
                                      """)

        print(translated_sentence, "**translated_sentence**")

        
        pattern = r'標題- (.+?) \| 描述- (.+)'
        matches = re.match(pattern, translated_sentence, re.DOTALL)
        if (matches):
            translated_title = matches.group(1).strip()
            translated_description = matches.group(2).strip()

            print("translated_title: ", translated_title)
            print("translated_description: ", translated_description)

            CH_product_update = OldDBProduct.objects.filter(
                advertiser_id=1878,
                product_key = product[0]
            ).first()

            extra_data = CH_product_update.extra if CH_product_update.extra else {}
            extra_data['translated_title'] = translated_title
            extra_data['translated_description'] = translated_description
            CH_product_update.save()

        else: 
            print("noooooo")

    logger.info(f"cost time {time.perf_counter() - start_time:0.2f}")
```


### GPT PROMPT 建議

1. 可以跟 GPT 說你的角色是什麼，這樣他會更明白他要怎麼做 - ex. 你是一個日文翻譯專家
2. 給幾個簡單的翻譯例子，這樣他會更明白怎麼翻譯 - ex. 濃紺請翻譯成深藍色
3. 可以用英文 prompt，他會給更精確的回應






