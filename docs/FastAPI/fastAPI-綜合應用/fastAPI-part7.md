---
sidebar_position: 7
---

## part7 - 建立 product 相關的 router

我們前面新增好第二個 table，現在我們想要新增一些 endpoint，讓我們可以藉由這些 router，抓取到我們想要的 Product

## 修改 Product table

### 1. 新增 ec_id 欄位

新增這個欄位的原因，是因為希望未來可以在某個 endpoint 的參數上，輸入 ec_id，就可以把所屬的所有商品抓出來

```py
# sql_alchemy/sql_alchemy.py

# ...省略

class Product(Base):
    __tablename__ = "product"
    id = Column(Integer, primary_key=True, autoincrement=True)
    ec_id = Column(Integer)                                         => 新增這個
    key = Column(String(100))
    title = Column(String(100))
    price = Column(Float)
    currency = Column(String(100))
    extra = Column(JSON)
    availability = Column(String(50))
    custom_labels = Column(JSON)
    publish_time = Column(DATETIME)
    update_time = Column(DATETIME)

# ...省略    

```

### 2. migration 指令


### 3. 生成測試資料

生成好欄位之後，我們直接用更新的方式更新就好，不用整個全部重建

```py
# sql_alchemy/build_data.py


# ... 省略

def update_product_data():
    session = create_session()

    # 查詢所有紀錄
    Products = session.query(Product).all()
    for product in Products:
        product.ec_id = random.choice([100, 2990, 3151, 1929])        

    session.commit()
    session.close()

if __name__ == "__main__":
    # ... 省略
    # 更新 product 資料
    update_product_data()   
```




### 5. 執行程式

修改好後，在執行這個檔案：
```shell
$ python sql_alchemy/build_data.py 
```
Ps. 這樣就可以把 4 個數字塞進所有產品的 ec_id 欄位之中




## 建立 Product 相關的 endpoint


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


### 2. 新增 product_router.py 檔案


```md
| FASTAPI
  |  -- sql_alchemy
        | -- __init__.py  
        | -- query.py
        | -- sql_alchemy.py
        | -- build_data.py
        | -- database.py
        | -- legacy_router.py
        | -- product_router.py              => 新增這個   
        | -- main.py             
  |  -- virtualenv
  |  app.py
```

### 3. product_router.py 檔案內容

#### 1. 新增 product_router.py 檔案

```md
| FASTAPI
  |  -- lib
        | -- src
                | -- shared
                        | -- __init__.py    
  |  -- sql_alchemy
        | -- __init__.py  
        | -- query.py
        | -- sql_alchemy.py
        | -- build_data.py
        | -- database.py
        | -- legacy_router.py
        | -- main.py         
        | -- product_router.py                  => 新增這個檔案
  |  -- virtualenv
  |  app.py
```

#### 2. product_router.py 檔案內容


* 基礎資料先引入和建立

```py
# FASTAPI/sql_alchemy/product_router.py

from database import SessionLocal
from fastapi import APIRouter, status, Depends, Query
from sqlalchemy.orm import Session
from sql_alchemy import Product


def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()


product_router = APIRouter(
    prefix='/products',
    tags=['products']
)


# 等等設定 endpoint 的地方
```

* main.py 資料夾引入剛剛建立的 router

```py
# FASTAPI/sql_alchemy/main.py

from fastapi import FastAPI, Request, status, Depends
import uvicorn
from database import SessionLocal
from legacy_router import router as legacy_router
from product_router import product_router                   => 新增這一行

app = FastAPI()

# Depends
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/")
async def root():
    return {"message": "Database operator"}

app.include_router(legacy_router)
app.include_router(product_router)                          => 新增這一行


if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```


#### 2-1 測試一: 用 ec_id 抓出所有指定的 ec

```py
@product_router.get("/test")
async def get_products(
    ec_id: int | None = None, 
    db: Session = Depends(get_db)
    ):
    results = (
        db.query(Product)
        .filter(
            Product.ec_id == ec_id
        ).all()
    )

    print("********** results", results)
    return results
```

建立好後，直接啟動 server:  `python sql_alchemy/main.py`，並且輸入 `http://0.0.0.0:8000/products/test?ec_id=2990`，就可以抓出所有 `ec_id = 2990` 的資料了



#### 2-2 測試二: 用 availability - str 抓出相關 ec

```py
@product_router.get("/stock")
async def get_products( 
    availability: str | None = None,
    db: Session = Depends(get_db),
    ):

    results = (
        db.query(Product).filter(Product.availability == availability)
    ).all()

    print("========= results", results)
    return results
```

前面已經啟動好 server，這邊就直接輸入路徑就好 - `http://0.0.0.0:8000/products/stock?availability=In%20Stock`，這樣就可以抓出所有 `availability = In Stock` 的資料。    
Ps. 會是這樣格式的原因，是因為我在資料庫的存的格式是這樣 - Preorder、In Stock、Out of Stock， `%20` 這個字串代表的是空格


#### 測試三: 一次設定兩個 query 參數，多重邏輯設定

```py
@product_router.get("/query")
async def get_products(
    ec_id: int | None = None,
    availability: str | None = None,
    db: Session = Depends(get_db)
):
    q = db.query(Product)

    if ec_id:
        q = q.filter(Product.ec_id == ec_id)
    
    if availability:
        q = q.filter(Product.availability == availability)

    products = q.all()

    return products
```

這邊就直接輸入路徑就好 - `http://0.0.0.0:8000/products/query?ec_id=2990&availability=Preorder`，這樣輸入我們就可以得到 `ec_id = 2990` + `availability=Preorder` 的資料


#### 測試四: 把所有 query 參數全部加進來

這邊有幾個參數要注意一下像是:
(1) `p_date_gte`、`p_date_lte`、`u_date_gte`、`u_date_lte`: 
```md
這些在輸入網址的時候，會像以下
- http://0.0.0.0:8000/products/all_query?p_date_gte=2024-05-26 -> 時間格式的 query
- http://0.0.0.0:8000/products/all_query?p_date_gte=2024-05-26%2013:00:00 -> 如果想要加上時間，由於中間要給 %20 這代表空格，因為資料庫裡面時間格式存的是像這樣 - 2024-05-27 15:49:13
```

(2) `limit` :
limit 要搭配這一段 - `products = q.limit(limit).all()`，這樣才可以在你新增 limit 的 query 時，才可以觸發數量限制

```py
@product_router.get("/all_query")
async def get_products(
    ec_id: int | None = None,
    key: str | None = None,    
    availability: str | None = None,   
    p_date_gte: str | None = None,
    p_date_lte: str | None = None,
    u_date_gte: str | None = None,
    u_date_lte: str | None = None,
    limit: int = 30000,
    db: Session = Depends(get_db),
):
    q = db.query(Product)

    if ec_id:
        q = q.filter(Product.ec_id == ec_id)
    
    if key:
        q = q.filter(Product.key == key)
    
    if currency:
        q = q.filter(Product.currency == currency)

    if availability:
        q = q.filter(Product.availability == availability)
    
    if p_date_gte:
        q = q.filter(Product.publish_time >= p_date_gte)
    
    if p_date_lte:
        q = q.filter(Product.publish_time <= p_date_lte)

    if u_date_gte:
        q = q.filter(Product.update_time >= u_date_gte)
    
    if u_date_lte:
        q = q.filter(Product.update_time <= u_date_lte)
    
    
    products = q.limit(limit).all()    

    return products
```






