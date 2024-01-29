---
title: 2162 - 修改 facebook UTM 參數
sidebar_label: "2. [UTM] 修改 facebook UTM 參數"
description: 修改 facebook UTM 參數
last_update:
  date: 2023-12-25
keywords:
  - tagtoo
  - FB UTM
sidebar_position: 2
---

### 詳細描述
"客戶希望能調整feed上的連結
相望調整連結：https://eshop.ttl.com.tw/b2b_cplist.aspx?catid=1&utm_source=FB_T&utm_medium=CPC&utm_campaign=BIOAD

調整產品之內容編號：
ttl:product:23608
ttl:product:23611
ttl:product:20549
ttl:product:23617
ttl:product:23615
ttl:product:23609
ttl:product:23607
ttl:product:23606
ttl:product:23613
"


### 解決方法
到 `api/configs/cpa_config.py` 這個資料夾中，就可以找到過往是如何 `修改/新增UTM` 的方法：

```py
# api/configs/cpa_config.py

# ...省略
def cpa_ttl(o, pb, **kwargs):
    specific_products_id = (
        'id=23608', 
        'id=23611', 
        'id=20549', 
        'id=23617',
        'id=23615',
        'id=23609',
        'id=23607',
        'id=23606',
        'id=23613',
    )
    url = kwargs.get('url', '')
    if pb == 71:
        for pid in specific_products_id:
            if pid in url:
                o.append_param({
                    'utm_source': 'FB_T',
                    'utm_medium': 'CPC',
                    'utm_campaign': 'BIOAD',
                })
    return o

cpa_builder.register(2162, cpa_ttl, 'eshop.ttl.com.tw')    
```

### PR、需求編號

[api]: https://github.com/Tagtoo/api/pull/1849