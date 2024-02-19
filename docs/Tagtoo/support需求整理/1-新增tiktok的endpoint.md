---
title: 163 - 新增 endpoint
sidebar_label: "1. [AddEndPoint] 新增 endpoint"
description: 新增 endpoint
last_update:
  date: 2024-01-29
keywords:
  - tagtoo
  - endpoint
sidebar_position: 1
---


Ps. 這一份文件描述是錯的，不要看


### 詳細描述     
由於之前展開的子類產品 endpoint 只有做 facebook 的，今天 op 要求 tiktok 的 feed 也要展開子類產品，因此要來新增一個 endpoint:


### 解決方法

```py
# api/apps/feed/views.py

class TiktokVariant(Tiktok):
    """
    Feed for TikTok Dynamic Product Ads.
    """        
    def get(self, advertiser_id):
        logging.info(advertiser_id)
        advertiser, products = self.get_advertiser_and_products(advertiser_id)
        if not advertiser:
            return self.return_404()
        else:
            products = extend_product_variants(products)            
            filename = 'tiktok-{}-{}.csv'.format(
                advertiser['name'],
                datetime.datetime.today(),
            )

        mapping, csv_products = self.csv_items_formatting(products, advertiser_id)
        return self.items_to_csv(filename, mapping.keys(), csv_products)

```

```py
# index.py
import webapp2
app = webapp2.WSGIApplication([
  # ...省略
    (r'/v2/feed/tiktok/(\d+)', 'apps.feed.views.TiktokVariant'),
  # ...省略
])
```


這個路徑的主要寫法，只要去參考 facebook v3 是怎麼寫的就好，照他的格式就可以把商品給展開

### PR、需求編號
[api] : https://github.com/Tagtoo/api/pull/1754