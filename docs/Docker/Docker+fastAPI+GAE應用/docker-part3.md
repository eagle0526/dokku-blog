---
sidebar_position: 3
---




Docker + fastAPI 應用 - part3
------

目前的架構為這樣，現在已經可以建立基本的環境了 (dockerfile)，因此現在來把 fastapi 的檔案給加上去     

```md
| FASTAPI_Docker
  |  Dockerfile
  |  pyproject.toml  
  |  main.py
```

## 設定 fastapi 基礎檔案



### 新增 fastapi 會用到的檔案

```md
| FASTAPI_Docker
  |  -- DBoperator              => 新增這個資料夾
        | -- __init__.py        
        | -- app.py             => fastapi 主程式
        | -- database.py        => 連接資料庫
        | -- models.py          => 設定 table
  |  Dockerfile
  |  pyproject.toml  
  |  main.py
```


### database.py

首先新增建立這個檔案，這個檔案的主要目的就是用來連接資料庫的！所有參數的介紹可以去 - fastAPI 的綜合應用那邊看

```py
import logging
from sqlalchemy import create_engine
from sqlalchemy.orm import declarative_base, sessionmaker

# 以下是連接資料庫的資訊
MYSQL_HOST = 'localhost'
MYSQL_USER = 'newuser'
MYSQL_PASSWORD = 'newpassword'
MYSQL_PORT = 3306
MYSQL_DB = 'test'

SQLALCHEMY_DATABASE_URL = f"mysql+pymysql://{MYSQL_USER}:{MYSQL_PASSWORD}@{MYSQL_HOST}:{MYSQL_PORT}/{MYSQL_DB}"

primary_engine = create_engine(SQLALCHEMY_DATABASE_URL, echo = True)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=primary_engine)

Base = declarative_base()
```

### models.py

這檔案就是要設定跟資料庫裡面的 table 一樣欄位架構！

Ps. 這邊要特別提到的就是 `from .database import Base`，由于之後我們要用 docker 打包，因此我們需要用相對路徑 `(.database)`來 import
```py
from .database import Base

from sqlalchemy import (
    Boolean,
    Column,
    Float,
    Integer,
    String,
    DATETIME,
    UniqueConstraint,
    Index,
    JSON,
)


class Product(Base):
    __tablename__ = "product"
    id = Column(Integer, primary_key=True, autoincrement=True)
    ec_id = Column(Integer)
    key = Column(String(100))
    title = Column(String(100))
    price = Column(Float)
    currency = Column(String(100))
    extra = Column(JSON)
    availability = Column(String(50))
    custom_labels = Column(JSON)
    publish_time = Column(DATETIME)
    update_time = Column(DATETIME)    
```


### app.py

這邊的內容主要就是啟用 `fastapi` 的相關設定，所有不懂的參數一樣都可以到 - fastAPI 的綜合應用那邊看

ps. 這邊 `import package` 一樣要記得使用相對路徑

```py
from fastapi import FastAPI, APIRouter, status, Depends
from sqlalchemy.orm import Session
from .database import SessionLocal
from .models import Product

app = FastAPI() # 建立一個 Fast API application

# Dependency
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/")
async def root():
    return {"message": "Database operator"}


@app.get("/get_products", status_code=status.HTTP_200_OK)
async def get_products(ec_id, db: Session = Depends(get_db)):
    results = (
        db.query(Product)
         .filter(
             Product.ec_id == ec_id
         ).all()
    )

    print("********** results", results)
    return {"status_code": status.HTTP_200_OK, "products": results}
```

這樣設定好後，我們就已經把 fastAPI 基本要設定的修改好了，不過要啟動他的話，我們接著要來修改 `dockerfile` 的設定