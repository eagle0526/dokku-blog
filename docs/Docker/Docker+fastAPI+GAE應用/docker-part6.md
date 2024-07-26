---
sidebar_position: 6
---

Docker + fastAPI 應用 - part6
------

## 建立 docker 時匯入資料

但是這樣寫的話，我們一開始是沒有資料的，因此我們會需要啟用 docker 時，順便塞一些測試資料到 docker 裡面，因此我們來修改一些檔案：

* 目前的檔案架構

```md
| FASTAPI_Docker
  |  -- DBoperator         
        | -- __init__.py        
        | -- app.py        
        | -- database.py   
        | -- models.py     
  |  Dockerfile
  |  pyproject.toml  
  |  main.py
  |  docker-compose.yml
```

* 新增一個資料夾 - base.py

```md
| FASTAPI_Docker
  |  -- DBoperator         
        | -- __init__.py        
        | -- app.py        
        | -- database.py   
        | -- models.py     
        | -- base.py             => 新增這個資料夾          
  |  Dockerfile
  |  pyproject.toml  
  |  main.py
  |  docker-compose.yml
```

###  新增 base.py 和修改 database.py、models.py、app.py 

#### 1. 先新增 base.py

要額外增加這個檔案的原因，是因為我們要把某個變數拉到這個檔案來，避免發生 `circular import`！

```py
# FASTAPI_Docker/DBoperator/base.py
from sqlalchemy.orm import declarative_base

Base = declarative_base()
```

#### 2. 修改 database.py

再來我們來修改 database.py 這個檔案，我們要在這個檔案新增一段：可以讓我們生成一些測試資料

ps. 這邊要記得把原本的 `Base = declarative_base()` 刪掉喔，一開始我們是寫在這個檔案的
```py
# FASTAPI_Docker/DBoperator/database.py


# ...省略

from .base import Base
from .models import Product

def init_db():
    Base.metadata.create_all(bind=primary_engine)
    db = SessionLocal()

    try:
        if not db.query(Product).first():
            initial_products = [
                Product(ec_id=2990, key='2990_key1', title='Product 1', price=100.0, currency='USD', extra={}, availability='in stock', custom_labels={}, publish_time='2023-01-01 00:00:00', update_time='2023-01-01 00:00:00'),
                Product(ec_id=2990, key='2990_key2', title='Product 2', price=500.0, currency='TWD', extra={}, availability='in stock', custom_labels={}, publish_time='2023-01-01 00:00:00', update_time='2023-01-01 00:00:00'),
                Product(ec_id=3151, key='3151_key1', title='Product 1', price=200.0, currency='USD', extra={}, availability='in stock', custom_labels={}, publish_time='2023-01-01 00:00:00', update_time='2023-01-01 00:00:00'),
                Product(ec_id=100, key='100_key1', title='Product 1', price=300.0, currency='JPY', extra={}, availability='in stock', custom_labels={}, publish_time='2023-01-01 00:00:00', update_time='2023-01-01 00:00:00')                
            ]
            db.add_all(initial_products)
            db.commit()
    finally:
        db.commit()
```

#### 3. 修改 models.py

這邊就比較簡單，只需要修改匯入的資料夾就好，從原本的 `from .database` 改成 `from .base`

```py
from .base import Base

# 省略
```


#### 4. 修改 app.py

最後我們把來修改主應用程式，利用 `fastapi` 的 `@app.on_event("startup")`，在應用程式啟動前，要先運行的動作

ps. 參考連結: `https://www.cnblogs.com/mazhiyong/p/13372006.html`
```py
# ... 省略
from .database import SessionLocal, init_db

@app.on_event("startup")
def on_startup():
    init_db()


# 省略
```



### 重新 build image 和 container


```shell
$ docker-compose up --build
```

這樣建立好後，就可以發現建立好的 `container` 裡面會有建立好的資料
