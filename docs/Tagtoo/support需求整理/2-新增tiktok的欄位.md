---
title: 163 - 新增 tiktok feed 欄位
sidebar_label: "2. [AddField] 增 tiktok feed 欄位"
description: 增 tiktok feed 欄位
last_update:
  date: 2024-01-29
keywords:
  - tagtoo
  - feed
sidebar_position: 2
---



### 詳細描述     
目前 163 tiktok 渠道用的 feed 是以下三個：
```md
> Tagtoo_S3_目錄_01 - https://api.tagtoo.com.tw/v1/feed/tiktok/163?begin_value=0&scope=5000
> Tagtoo_S3_目錄_02 - https://api.tagtoo.com.tw/v1/feed/tiktok/163?begin_value=5001&scope=5000
> Tagtoo_S3_目錄_03 - https://api.tagtoo.com.tw/v1/feed/tiktok/163?begin_value=10001&scope=5000
```

不過目前的渠道，沒有 product_type 這個欄位，如果要幫這個渠道添加欄位，要怎麼新增呢？


### 解決方法

#### (1) 打開 api 檔案

1. 搜尋 index.py 這個檔案，這邊有所有的 api 路徑
2. 在 index.py 裡面搜尋 tiktok ， 會找到一個 api 的路徑

```py
# Feed related
(r'/v1/feed/tiktok/(\d+)', 'apps.feed.views.Tiktok'),
```

看到這個路徑就可以知道關於 tiktok 路徑的設定寫在哪邊

3. 在檔案搜尋輸入 view.py ，找到 feed/views.py 這個檔案

4. 在這個檔案搜尋 tiktok ，就可以找到 class Tiktok：

照下面的設定就可以新增指定欄位！
```py
class Tiktok(FeedHandler, CsvHandler):
    """
    Feed for TikTok.
    """
    publisher_id = 475
    field_mapping_table = OrderedDict((
        (u"sku_id",         u"product_key"),
        (u"title",          u"title"),
        (u"description",    u"description"),
        (u"image_link",     u"image_url"),
        (u"link",           u"ad_click_link"),
        (u"availability",   u"availability"),
        (u"condition",      u"condition"),
        (u"price",          u"store_price"),
        (u"sale_price",     u"price"),
        (u"brand",          u"brand"),
        (u"item_group_id",  u"item_group_id"),
        (u"custom_label_0", u"custom_label_0"),
        (u"custom_label_1", u"custom_label_1"),
        (u"custom_label_2", u"custom_label_2"),
        (u"custom_label_3", u"custom_label_3"),
        (u"custom_label_4", u"custom_label_4"),
        (u"product_type", u"category_path"),          # 新的欄位增加在這邊
    ))
```

### PR、需求編號
