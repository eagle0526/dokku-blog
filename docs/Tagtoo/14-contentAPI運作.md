---
title: tagtoo/contentAPI
sidebar_label: "14. [tagtoo] contentAPI in GMC"
description: GMC 的 contentAPI 是怎麼運作的
last_update:
  date: 2024-08-27
keywords:
  - contentAPI
  - cron-job-v2
  - GoogleSheet
tags:
  - cron-job-v2
sidebar_position: 14
---



## ContentAPI

如果今天你已經對GMC很熟了，那應該會知道有一個匯入產品的方法，叫做 ContentAPI，他可以透過打 google 指定 API 的方法，直接把一堆產品打進 GMC 的帳戶內，因此現在來介紹一下 tagtoo 的 contentAPI 是怎麼運作的


### cron-job-v2

在 `cron-job-v2` 這個專案裡面，可以找到這個資料夾 - `gmc-content-api` 主要就是在做管理 contentAPI 的任務:

* 可以在 enums.py 這個資料夾，找到主要打的 api
```py
# cron-job-v2/src/cronjobs/gmc-content-api/enums.py 

# domain 'legacy-api.tagtoo.com.tw' is connected to old db
OLD_DB_API = "https://legacy-api.tagtoo.com.tw/v1/feed/GMC/{EC_ID}"
NEW_DB_API = "https://api.tagtoo.com.tw/v1/feed/GMC/{EC_ID}"
```

* 並且查看 utils.py，會發現目前都是用 `OLD_DB_API` 此 api，目前新 API 還沒啟用
```py
# cron-job-v2/src/cronjobs/gmc-content-api/utils.py

def get_products_through_api(ec_id, conditions, specials):
    if ec_id in (0,):
        url = NEW_DB_API.format(EC_ID=ec_id)
    else:
        url = OLD_DB_API.format(EC_ID=ec_id)
    resp = requests.post(
        url=url, json={"conditions": conditions, "specials": specials}, timeout=300
    )
    return resp.json()
```

不過這邊會發現有兩個奇怪的參數: `conditions` 和 `specials`，這時候就要提到另一個 GoogleSheet 資料夾的


### Google Sheet
* GMC: https://docs.google.com/spreadsheets/d/1hKSe_Kj-4WENHJnf2mcd5cqCty9NIpUs2d69O0a0hgc/edit?gid=974154889#gid=974154889
* 點進此資料夾後，會發現裡面有好多不同的參數，`gtm-content-api` 主要就會吃這幾項參數來發送 API，可以看到最左側的參數 `EC ID`，就是目前有在發送ID的客戶，如果今天想要停掉該 api，直接把 Method 調成 delete 就好


