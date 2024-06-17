---
sidebar_position: 2
---

## part2 - 資料庫匯入資料

在 part1 的時候，要記得執行 `python sql_alchemy/sql_alchemy.py` 喔，要記得先創建 `table`，在這一步驟才能匯入資料

## 1. 資料夾架構

我們在 part1 的時候，最後的資料庫樣式是這樣
```md
| FASTAPI
  |  -- sql_alchemy
        | -- __init__.py  
        | -- query.py
        | -- sql_alchemy.py
  |  -- virtualenv
  |  app.py
```
Ps. app.py 和 query.py 裡面都是空的



## 2.新增 build_data.py 檔案

現在我們先來用 `SQLAlchemy` 創建 DB，並且用政府提供的一些資料，把資料塞進去資料庫，等等我們可以來用
```md
| FASTAPI
  |  -- sql_alchemy
        | -- __init__.py  
        | -- query.py
        | -- sql_alchemy.py
        | -- build_data.py        
  |  -- virtualenv
  |  app.py
```


### 2-1 - 匯入要用的套件

* create_session 是剛剛在 sql_alchemy.py 檔案那邊新增的函式，作用是我們可以用他跟資料庫互動
* Website 是我們新增的 table 名稱

```py
from sql_alchemy import create_session
from sql_alchemy import Website
import requests
import json
from datetime import datetime
```

### 2-2 使用政府提供的資料匯入
```py
# 資料來源，內容是 json 檔案
url = "https://sme.moeasmea.gov.tw/startup/upload/opendata/gov_infopack_opendata.json"
res = requests.get(url)

datas = json.loads(res.text)

for item in datas:
    # print(item)
    data_obj = {
        "title": item["標題"],
        "content": item["內容"],
        "picture": item["主圖"],
        "category": item["分類"],
        "youtube": item["youtube嵌入代碼"],
        "slideshare": item["slideshare嵌入代碼"],
        "publish_time": datetime.strptime(item["建立時間"], "%Y%m%d%H%M%S"),
        "update_time": datetime.strptime(item["修改時間"], "%Y%m%d%H%M%S")
    }
    # 呼叫剛剛創建的 session -> 讓我們可以使用 orm 
    session = create_session()

    # 把一大堆資料，統一塞進 Website 這個 table 裏面
    session.add(Website(**data_obj))
    session.commit()
    session.close()
```


### 最終完整檔案
```py
from sql_alchemy.sql_alchemy import create_session
from sql_alchemy.sql_alchemy import Website
import requests
import json
from datetime import datetime


url = "https://sme.moeasmea.gov.tw/startup/upload/opendata/gov_infopack_opendata.json"
res = requests.get(url)


# print(res)
datas = json.loads(res.text)


for item in datas:
    # print(item)
    data_obj = {
        "title": item["標題"],
        "content": item["內容"],
        "picture": item["主圖"],
        "category": item["分類"],
        "youtube": item["youtube嵌入代碼"],
        "slideshare": item["slideshare嵌入代碼"],
        "publish_time": datetime.strptime(item["建立時間"], "%Y%m%d%H%M%S"),
        "update_time": datetime.strptime(item["修改時間"], "%Y%m%d%H%M%S")
    }
    # 呼叫剛剛創建的 session -> 讓我們可以使用 orm 
    session = create_session()

    # 把一大堆資料，統一塞進 Website 這個 table 裏面
    session.add(Website(**data_obj))
    session.commit()
    session.close()

```







