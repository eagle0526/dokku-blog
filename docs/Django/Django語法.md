---
sidebar_position: 2
---

## Django 操控資料庫的語法ㄋ


### 標題包含某個字串
```py
products = OldDBProduct.objects.filter(
    advertiser_id=EC_ID,
    expire__gte=timezone.now(),
    title__contains='雙11',
)
```


### 資料庫中，有哪些 ID 跟陣列中的值一樣

```py
pks_has_expired = [42269670, 44180464]
products = OldDBProduct.objects.filter(
    advertiser_id=EC_ID, 
    pk__in=pks_has_expired
)
```



### values_list 用法

從資料庫取出指定的資料後，會把指定的兩個資料格式，兩兩包成一個 tuple，最後用一個陣列包住全部

```py
products = OldDBProduct.objects.filter(
    advertiser_id=EC_ID,
    expire__gte=timezone.now()
)
pks_has_expired = []
data_sets = products.values_list('pk', 'link')
print(data_sets, "============")
```

```shell
root@c254923ebbbf:/srv# python main.py

------
<BulkUpdateOrCreateQuerySet [(11111, 'https://google.com'), (22222, 'https://facebook.com'), (33333, 'https://nice.com')]> ============
```







