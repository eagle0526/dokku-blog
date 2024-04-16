---
sidebar_position: 2
---

0、前言
------
:::tip
**寫此篇文章的動機**  
這一篇紀錄串接 chatGPT 和 翻譯資料庫中的日文產品並重新會匯入
:::


## sousou-product-translate - part3 - 使用 批量抓取 + bulk_update_or_create_context

### 需求
(1) 前面那個寫法會造成太多的 I/O，也就是因為翻譯一個產品就跑一次迴圈，會造成效率大減，因此我們使用 `bulk_update_or_create_context` 方法一次批量更新


* 修改的地方

1. 一開始在抓日文商品的時候，就先 5 個 5 個分成一堆
2. 使用 `bulk_update_or_create_context`


### 實際流程

1. 先抓出所有中文商品 - 並且有幾項篩選條件   
(1) 要篩選出 expire > now    
(2) 要抓出沒有翻譯過的商品 (extra 欄位裡面沒有 translated_title 欄位)    
(3) 根據 expire 排序，時間最遠的越前面 (先更新最新被瀏覽的商品)  

2. 把剛剛抓出的中文商品，用 `product_key` 轉成陣列
3. 把中文 key 全部轉成日文商品的 key
4. 把那些陣列分成 5 個 5 個為一堆 - `product_groups`
5. `product_groups` 拿去跑迴圈 (一組就是5個商品)
6. 用 `orm` 的 `product_key__in` 語法，一次抓出 5 個對應的日文商品 (前面我們已經把中文商品的 key 轉成日文商品的 key)
7. 把取出的日文資料包含，`product_key, title, description` 都轉成陣列，並且再次把 `日文 key 換成 中文 key` - `processed_JP_products`
8. 現在 `processed_JP_products` 有 `中文的 key 和日文的 title, description`，現在我們一次把 5 個產品和丟進 chatGPT 讓他翻譯
9. 拿回來的翻譯是一個字串，使用 `eval` 轉回陣列
10. 最後使用 `bulk_update_or_create_context` 存資料進資料庫



### 第一版本 - 先把抓出來的產品 5 個 5 放一堆 + 最後一筆一筆跑迴圈存進資料庫
```py
def new_bulk_method_translated():
    # 現在要先抓出，中文 + 還沒翻譯完成 + 還沒過期的商品
    logger.info("Start updating old DB")
    start_time = time.perf_counter()
    product_counts = 0

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
    
    # 抓出全部中文商品
    # 把中文商品五個五個分成一組
    # 找出對應的英文商品
    CH_data_sets = CH_products.values_list('product_key')
    
    # 把 key 整理成中文的 key
    CH_revised_data_sets = [(key[0].replace('product:', 'product:jp:')) for key in CH_data_sets]
    
    # 5 個 5 個分成一堆
    batch_size = 5
    product_groups = [CH_revised_data_sets[i:i+batch_size] for i in range(0, len(CH_revised_data_sets), batch_size)]    

    # 直接用這一堆去找到中文對應的日文產品
    for item in product_groups[:100]:
        print("******************************************")
        JP_products = OldDBProduct.objects.filter(
            advertiser_id=1878,
            product_key__in = item
        ).values_list('product_key', 'title', 'description')
        product_counts += JP_products.count()
        
        processed_JP_products = [(key.replace('product:jp', 'product'), title, description) for key, title, description in JP_products]
        print(processed_JP_products, "processed_JP_products*******")
        translated_sentence = chatGPT(f"""
                                        I will provide you with some Japanese product names and descriptions, format like this [('product_key', 'Japanese title', 'Japanese description'),('product_key', 'Japanese title', 'Japanese description')]. please help me to translated the content into Traditional Chinese that is suitable for Taiwanese people to understand, and the returned format like this [('product_key', 'Translated Chinese title', 'Translated Chinese description'),('product_key', 'Translated Chinese title', 'Translated Chinese description')].

                                        In addition, please note that some Japanese Chinese characters involve proper nouns. Although the literal translation is not wrong, Taiwanese people may not understand them, including but not limited to the following two examples: please translate 濃紺 into 深藍色, 急須 into 茶壺. Please make your own analogy based on your understanding of Taiwanese and Japanese culture.

                                        Finally, please note that Japanese katakana and hiragana do not need to appear in the translated content, because Taiwanese people cannot understand it.                                         
                                       
                                        Need be translated sentence : {processed_JP_products}
                                      """)        
        
        print(translated_sentence, "*****")

        try:
            # GPT 吐給你的是一個 string 格式，因此要記得轉成 array
            new_array = eval(translated_sentence)
        except Exception as e:
            logger.error('Old DB Update Error while evaluating translated_sentence: {}'.format(e))    
        
                
        for key, translated_title, translated_description in new_array:
            final_product = OldDBProduct.objects.filter(
                advertiser_id=1878,
                product_key = key
            ).first()

            final_product.extra['translated_title'] = translated_title
            final_product.extra['translated_description'] = translated_description            
            final_product.save()


    logger.info(f"update product counts: {product_counts}, total cost time: {time.perf_counter() - start_time:0.2f}")
```


### 第二版本 - 使用 bulk_update_or_create_context 批量存進資料庫

```py
def old_db_bulk_translated():
    # 現在要先抓出，中文 + 還沒翻譯完成 + 還沒過期的商品
    logger.info("Start updating old and new DB")
    start_time = time.perf_counter()    
    product_counts = 0
    NOW = timezone.now()
    THREE_DAYS_LATER = NOW + timezone.timedelta(days=3)
    BATCH_SIZE = 5

    CH_products = OldDBProduct.objects.filter(
        advertiser_id=1878,
        link__icontains = 'CHT',
        expire__gt=timezone.now(),
    ).annotate(
        extra_str=Cast('extra', CharField()),  # 將 JSONField 轉換為 CharField
    ).exclude(
        extra_str__contains='"translated_title"'    
    ).order_by("-expire")


    logger.info(f"Have {CH_products.count()} products which expire and not yet translated")


    CH_data_sets = CH_products.values_list('product_key')    
    CH_revised_data_sets = [(key[0].replace('product:', 'product:jp:')) for key in CH_data_sets]        
    product_groups = [CH_revised_data_sets[i:i+BATCH_SIZE] for i in range(0, len(CH_revised_data_sets), BATCH_SIZE)]    

    
    for item in product_groups:
        print("******************************************")        
        JP_products = OldDBProduct.objects.filter(
            advertiser_id=1878,
            product_key__in = item
        ).values_list('product_key', 'title', 'description')
        product_counts += JP_products.count()
        
        processed_JP_products = [(key.replace('product:jp', 'product'), title, description) for key, title, description in JP_products]        
        print(processed_JP_products, "processed_JP_products*******")
        translated_sentence = chatGPT(f"""
                                        I will provide you with some Japanese product names and descriptions, format like this [('product_key', 'Japanese title', 'Japanese description'),('product_key', 'Japanese title', 'Japanese description')]. please help me to translated the content into Traditional Chinese that is suitable for Taiwanese people to understand, and the returned format like this [('product_key', 'Translated Chinese title', 'Translated Chinese description'),('product_key', 'Translated Chinese title', 'Translated Chinese description')].

                                        In addition, please note that some Japanese Chinese characters involve proper nouns. Although the literal translation is not wrong, Taiwanese people may not understand them, including but not limited to the following two examples: please translate 濃紺 into 深藍色, 急須 into 茶壺. Please make your own analogy based on your understanding of Taiwanese and Japanese culture.

                                        Finally, please note that Japanese katakana and hiragana do not need to appear in the translated content, because Taiwanese people cannot understand it.                                         
                                       
                                        Need be translated sentence : {processed_JP_products}
                                      """)
        

        print(translated_sentence, "translated_sentence*******")

        try:
            new_array = eval(translated_sentence)

            with OldDBProduct.objects.bulk_update_or_create_context(
                update_fields = ['expire', 'updated', 'extra'],
                match_field = 'product_key',
                batch_size = BATCH_SIZE
            ) as bulkit_old, NewDBProduct.objects.bulk_update_or_create_context(
                update_fields = ['expired_at', 'extra', 'updated_at'],
                match_field = ['key', 'key_fmt'],
                batch_size = BATCH_SIZE,
            ) as bulkit_new:
                for item in new_array:
                    new_key = item[0].split("sousou:product:")[1]

                    bulkit_old.queue(OldDBProduct(
                        product_key = item[0],
                        extra = dict(
                            preorder = "",
                            customer_product_id = "",
                            mpn = "",
                            gender = "",
                            material = "",
                            custom_label_0 =  "",
                            custom_label_1 = "",
                            custom_label_3 =  "",
                            custom_label_2 =  "",
                            custom_label_4 = "",
                            overseas = "",
                            GTIN = "",                            
                            age_group = "",
                            extension_images =  {},
                            translated_title = item[1],
                            translated_description = item[2],
                        ),
                        expire = THREE_DAYS_LATER,
                        updated = NOW
                    ))
                    
                    bulkit_new.queue(NewDBProduct(                        
                        key = new_key,
                        key_fmt = 'sousou:product:{key}',
                        extra = dict(
                            translated_title = item[1],
                            translated_description = item[2],
                        ),
                        expired_at = THREE_DAYS_LATER,
                        updated_at = NOW                        
                    ))

                    print(bulkit_new, "******bulkit_new")
                    print(bulkit_old, "******bulkit_old")
                
        except Exception as e:
            logger.error('Old DB Update Error while evaluating translated_sentence: {}'.format(e))    
        
                
    logger.info(f"update product counts: {product_counts}, total cost time: {time.perf_counter() - start_time:0.2f}")
```



### 此專案學到的事情

1. gptAPI 的使用
2. gpt prompt 的使用，要如何才能提高翻譯的精準度
3. bulk_update_or_create_context 的使用，如果今天商品量太大，一筆一筆跑迴圈會造成速度太慢





