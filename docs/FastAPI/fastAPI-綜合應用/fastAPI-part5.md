---
sidebar_position: 5
---

## part5 - 用新增 alembic 套件，新增 table 的資料

由於之後可能會需要其他的欄位，還有資料，因此我們先來安裝 `alembic 套件`，讓我們先來新增幾個欄位熱身

## 使用 alembic package

### 1. 資料夾架構

一樣先來看目前的資料夾架構

```md
| FASTAPI
  |  -- sql_alchemy
        | -- __init__.py  
        | -- query.py
        | -- sql_alchemy.py
        | -- build_data.py
        | -- database.py
        | -- legacy_router.py
        | -- main.py             
  |  -- virtualenv
  |  app.py
```

Ps. app.py 和 query.py 裡面都是空的




### 2. 修改 Website 的 model 設定


```py
# sql_alchemy/sql_alchemy.py

from sqlalchemy import Integer, String, DATETIME, TEXT, Float

class Website(Base):
    __tablename__ = "website"
    id = Column(Integer, primary_key=True, autoincrement=True)
    title = Column(String(100))
    content = Column(TEXT)
    picture = Column(TEXT)  # Chrome 網址長度上限
    category = Column(String(100))
    youtube = Column(TEXT)  # Chrome 網址長度上限
    slideshare = Column(TEXT)  # Chrome 網址長度上限
    publish_time = Column(DATETIME)
    update_time = Column(DATETIME)
    price = Column(Float)                              # => 新增這個欄位


# ...省略
```


### 3. 安裝 alembic

```shell
$ pip install alembic
```

### 4. 初始化 Alembic

```shell
$ alembic init alembic

======
  Creating directory '/Users/yee0526/Desktop/python/fastAPI/alembic' ...  done
  Creating directory '/Users/yee0526/Desktop/python/fastAPI/alembic/versions' ...  done
  Generating /Users/yee0526/Desktop/python/fastAPI/alembic/script.py.mako ...  done
  Generating /Users/yee0526/Desktop/python/fastAPI/alembic/env.py ...  done
  Generating /Users/yee0526/Desktop/python/fastAPI/alembic/README ...  done
  Generating /Users/yee0526/Desktop/python/fastAPI/alembic.ini ...  done
  Please edit configuration/connection/logging settings in '/Users/yee0526/Desktop/python/fastAPI/alembic.ini' before proceeding.
```
這個指令會創造這些檔案，最主要的就是 `alembic.ini` 和 `env.py` 這兩個


### 5. 修改 alembic.ini 文件，設置數據庫 URL：

```ini
# 在 alembic.ini 文件中找到這一行，並設置為你的數據庫 URL
sqlalchemy.url = mysql+pymysql://newuser:newpassword@localhost:3306/test
```

### 6. 編輯 alembic 資料夾裡面的 env.py 檔案，告訴 Alembic 你的 model：

```py
# alembic/env.py

# ...省略

from sql_alchemy.sql_alchemy import Website
target_metadata = Website.metadata

# ...省略
```



### 7. 修改欄位的指令 - migration

#### 7-1 創建 migration file
```shell
$ alembic revision --autogenerate -m "Add price column to website table"
```
Ps. 這個指令會在 `alembic/versions` 這個資料夾裡面，多產出一個檔案，也就是會紀錄你 table 變化的過程

#### 7-2 實際執行

```shell
$ alembic upgrade head
```



### 8. 把 price 資料加進去

現在我們已經有 price 的欄位了，不過這些資料都沒有 price 的 data，因此現在在 `build_data.py` 這個檔案裡面，我們來新增一個 function，亂數產生 price 並把這些資料塞進去


```py
# sql_alchemy/build_data.py
import random

# ...省略

import random
def update_data():
    session = create_session()
    
    # 查詢所有紀錄
    Websites = session.query(Website).all()

    for website in Websites:
        website.price = random.uniform(0, 100)

    # 提交更新
    session.commit()
    session.close()

if __name__ == "__main__":
    # 從 0 建立 data
    # build_data()

    # 更新資料
    update_data()
```

修改好後，在執行這個檔案：
```shell
$ python sql_alchemy/build_data.py 
```
最後看 table，就可以發現一開始匯入的資料，都有 price 了






## 直接更新整個資料庫

如果今天你不想要單純更新原有資料庫的資料，想要把整個資料庫刪掉，並且重新把有 price 資料的 data 匯入，可以這樣做：

```py
# sql_alchemy/build_data.py


def build_data():
    url = "https://sme.moeasmea.gov.tw/startup/upload/opendata/gov_infopack_opendata.json"
    res = requests.get(url)

    datas = json.loads(res.text)

    # 呼叫剛剛創建的 session -> 讓我們可以使用 orm 
    session = create_session()
    # 清空表格
    session.query(Website).delete()
    session.commit()


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
            "update_time": datetime.strptime(item["修改時間"], "%Y%m%d%H%M%S"),
            "price": random.uniform(0, 100)
        }


        # 把一大堆資料，統一塞進 Website 這個 table 裏面
        session.add(Website(**data_obj))
        session.commit()
        session.close()

# ...省略

if __name__ == "__main__":
    # 從 0 建立 data
    build_data()
```


重點在匯入資料庫前，先把 table 整個清空一次，之後在新增就好


