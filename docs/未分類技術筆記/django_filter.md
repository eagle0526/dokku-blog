---
sidebar_position: 2
---




0、前言
------
:::tip
**寫此篇文章的動機**  
紀錄 django filter 的一些用法
:::



## 修改資料庫 json 欄位的值


```py
def update_product():
    product = OldDBProduct.objects.filter(
        advertiser_id = 1878,
        product_key = 'sousou:product:3175429'
    ).first()
    if product:
        # 讀取 extra 欄位的值
        extra_data = product.extra

        # 修改 translated_description 的值
        if 'translated_description' in extra_data:            
            extra_data['translated_description'] = '・京型友禪傳統職人手工染色。 ・面料採用三重縣津市的傳統工藝品『伊勢木棉』。 ・伊勢木棉手巾，越洗觸感越柔軟，質感更佳。 ・柔軟親膚，亦適合小朋友使用。'
        

        # 更新到資料庫中
        product.extra = extra_data
        product.save()
```


## 刪除資料庫 json 欄位的值

```py
def update_product():
    product = OldDBProduct.objects.filter(
        advertiser_id = 1878,
        product_key = 'sousou:product:3175429'
    ).first()
    if product:
        
        # 刪除 json 鍵值得寫法
        if 'description' in extra_data:
            del extra_data['description']

        # 更新到資料庫中
        product.extra = extra_data
        product.save()
```



## 篩選出資料庫 json 欄位是否有 translated_title 的 key 值

順便更新 expire < 今天，把所有的 expire 改成三天後

```py
from django.db.models import CharField
from django.db.models.functions import Cast

    products = OldDBProduct.objects.filter(
        advertiser_id=1878,
        expire__lt=timezone.now(),  # expire 小於今天
    ).annotate(
        extra_str=Cast('extra', CharField()),  # 將 JSONField 轉換為 CharField
    ).filter(
        extra_str__contains='"translated_title"'
    ).order_by("-expire")

    print(products.count())
    three_days_later = timezone.now() + timezone.timedelta(days=3)
    print(three_days_later, "*****")
    for item in products[:2]:

        item.update(expire=three_days_later)

        print(item.product_key)
        print(item.expire)
        print(item.extra)
```
