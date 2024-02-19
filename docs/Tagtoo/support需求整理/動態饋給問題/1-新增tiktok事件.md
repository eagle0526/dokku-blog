---
title: 715 - 動態饋給問題
sidebar_label: "1. [reviseFeed] 715 - 標題亂碼修正"
description: 動態饋給問題
last_update:
  date: 2023-12-06
keywords:
  - tagtoo
  - 動態饋給問題
sidebar_position: 1
---



### 詳細描述     
"我們 FEED 抓到的商品名名稱有亂碼，有可能是因為這個品項的商品名稱有「±」這個符號的關係，導致系統無法辨識，但想請問這個問題有辦法解決嗎～

商品正常名稱：高麗菜(每粒約1kg±10%)
我們 FEED 抓到的商品名稱：高麗菜(每粒約1kgï¿½10%)
目錄編號：718345755010011
商品編號：2200300100101
網站商品頁面：https://online.carrefour.com.tw/zh/2200300100101.html"


### 解決方法

這個原因是因為編碼問題，資料庫的格式無法存取該符號，因此存到資料庫裡面會變亂碼，解決方式也很簡單，直接到以下資料夾，把該客戶的產品標題中，有這個符號的產品標題修改就好

```python
# apps/feed/query_products.py

# ...省略
def set_item_title_for_different_ec(advertiser_id, item, publisher_id):
    # carrefour
    if str(advertiser_id) == '715':    
        item['title'] = item['title'].replace(u'ï¿½', u'±')


# 順便修改 description 中的符號
def set_item_description_for_different_ec(advertiser_id, publisher_id, item):        
    if str(advertiser_id) == '715':
        # 促銷資訊字數過多會造成line 產品拒登
        if publisher_id == 474:
            item['description'] = ''
        # ± 此符號會顯示亂碼 
        else:
            item['description'] = item['description'].replace(u'ï¿½', u'±')  
```


### PR、需求編號
[api] : https://github.com/Tagtoo/api/pull/1837
