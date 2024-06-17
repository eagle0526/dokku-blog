---
sidebar_position: 3
---

## part3 - api router 讀取出資料庫的資料


## 1. 資料夾架構

我們在 part2的時候，最後的資料庫樣式是這樣
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

Ps. app.py 和 query.py 裡面都是空的



## 2. 新增 database.py 檔案

這個檔案的目的，跟 `sql_alchemy.py` 有一點點類似，不過因為 `sql_alchemy.py` 主要是用來新增 DB 用的，現在新增 `database.py` 這個，主要是後續 `router` 想要使用資料庫的檔案都可以用他，因此才拆分開來。  

Ps. 不過想要寫在一起也是可以的   

```md
| FASTAPI
  |  -- sql_alchemy
        | -- __init__.py  
        | -- query.py
        | -- sql_alchemy.py
        | -- build_data.py
        | -- database.py
  |  -- virtualenv
  |  app.py
```


### 2-1 - database.py 內容介紹

基本上很多內容都跟 `sql_alchemy.py` 很像，就不多做既哨

```py
import logging
# 連接資料庫
from sqlalchemy import create_engine
# 建造初始的 model 資料表 和 讓資料庫可以使用 orm query
from sqlalchemy.orm import declarative_base, sessionmaker
# 讓函式重複跑
from tenacity import retry, stop_after_attempt, wait_fixed, retry_unless_exception_type
# 錯誤顯示訊息
from sqlalchemy.exc import OperationalError

# engine_url = "mysql+pymysql://newuser:newpassword@localhost:3306/test"

# 以下是連接資料庫的資訊
MYSQL_HOST = 'localhost'
MYSQL_USER = 'newuser'
MYSQL_PASSWORD = 'newpassword'
MYSQL_PORT = 3306
MYSQL_DB = 'test'

SQLALCHEMY_DATABASE_URL = f"mysql+pymysql://{MYSQL_USER}:{MYSQL_PASSWORD}@{MYSQL_HOST}:{MYSQL_PORT}/{MYSQL_DB}"
primary_engine = create_engine(SQLALCHEMY_DATABASE_URL, echo=True)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=primary_engine)

Base = declarative_base()
```

## 3. legacy_router.py 檔案內容

再來就可以來寫 api 的檔案內容了，先來新增檔案
```md
| FASTAPI
  |  -- sql_alchemy
        | -- __init__.py  
        | -- query.py
        | -- sql_alchemy.py
        | -- build_data.py
        | -- database.py
        | -- legacy_router.py       => 新增這個檔案
  |  -- virtualenv
  |  app.py
```

### 3-1 引入等等會用到的東西

* SessionLocal 是前面在 `database.py` 建立好的 function，我們可以用它來跟資料庫溝通
* FastAPI, APIRouter, status, Depends 這些都是 api 相關的函式，比較特別的是 Depends，他可以讓我們在啟用函式之前，先行啟用指定的函式，不懂可以去看 `fastAPI-Dependencies` 這個檔案
* Session 這個會根據用法，有不同的意義，我們等等會用在連結資料庫
* Website 是我們的 table 名稱

```py
# legacy_router.py

from database import SessionLocal
from fastapi import FastAPI, APIRouter, status, Depends
from sqlalchemy.orm import Session
from sql_alchemy import Website
```

### 3-2 設定好基礎路徑

這裡的用法也可以到 `fastAPI-routes-setting` 這個檔案找到用法

```py
app = FastAPI() # 建立一個 Fast API application

router = APIRouter(
    prefix='/legacy',
    tags=["legacy"]
)
```

### 3-3 設定資料庫 Dependency

在 FastAPI 中，依賴項用於在路由處理函式中處理需要先行執行的邏輯，例如建立資料庫連線、驗證身份等等。    
     
具體來說，這個 get_db 函式用於獲取資料庫的會話（session）。在函式內部，它使用 SessionLocal() 函式來建立資料庫的會話物件，然後使用 Python 的 yield 關鍵字將該會話物件傳遞給呼叫者。   
這允許在路由處理函式中使用 get_db 作為參數，並在該函式內部使用該資料庫會話。   
   
try 和 finally 塊確保在函式執行完畢後，無論成功與否都會關閉資料庫的會話。這樣可以確保不會因為資源洩漏而導致資料庫連線未正確關閉。  
```py
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### 3-4 設定完整路徑

* 設定好 `/get_products` 路徑
* 設定參數，id 和 db，並且用 Depends，讓此函式觸發前，先讀取進資料庫
* 並且因為我們使用 `sqlalchemy`，下面我們就可以用 orm 的方法，取出我想要的資料庫資料
* 最後 `app.include_router(router)` 讓 app 路徑可以吃進 `APIRouter` 設定的路徑

```py
@router.get("/get_products", status_code=status.HTTP_200_OK)
async def get_products(id, db: Session = Depends(get_db)):
    results = (
        db.query(Website)
        .filter(
            Website.id == id
        ).all()
    )

    print("********** results", results)
    return {"status_code": status.HTTP_200_OK, "products": results}


app.include_router(router)
```

### 3-5 實際測試

最後我們路徑是長這樣：

```md
http://127.0.0.1:8000/legacy/get_products
```

但是因為我有加上 `id` 參數，所以實際是要打這個：

```md
http://127.0.0.1:8000/legacy/get_products?id=2
```

在瀏覽器輸入這一串，就可以得到資料庫裡面，id 是 2 的資料了！





### 3-6 最終完整檔案
```py
# FASTAPI/sql_alchemy/legacy_router.py

from database import SessionLocal
from fastapi import FastAPI, APIRouter, status, Depends
from sqlalchemy.orm import Session

from pprint import pprint
from sql_alchemy import Website

app = FastAPI() # 建立一個 Fast API application

router = APIRouter(
    prefix='/legacy',
    tags=["legacy"]
)

# Dependency
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()


@router.get("/get_products", status_code=status.HTTP_200_OK)
async def get_products(id, db: Session = Depends(get_db)):
    results = (
        db.query(Website)
        .filter(
            Website.id == id
        ).all()
    )

    print("********** results", results)
    return {"status_code": status.HTTP_200_OK, "products": results}

app.include_router(router)
```







