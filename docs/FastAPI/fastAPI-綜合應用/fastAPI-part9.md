---
sidebar_position: 9
---

## part9 - 設定 create product endpoint

part8 我們設定好結合 `pydantic` 的 endpoint，讓我們可以利用 api 查看資料庫的商品，並且是經由 `pydantic` 偵測過的型態，現在我們來寫一個 `create product` 的 `endpoint`。


## 在 product_router.py 新增 endpoint 和在 product_handler.py 新增方法


### 1. product_router.py

* `db: Session = Depends(get_db)` -> 這一段是讓你可以操作資料庫
* `product: ProductBaseIn`        -> 這一段是 query 要填寫的參數，先由 pydantic 規定好格式
* `create_product(db, product)`   -> 這是等等要創建的方法，並且最後會直接傳資料過去

```py
# FASTAPI/sql_alchemy/product_router.py

from product_handler import handle_products, create_product

# ... 省略

# create product endpoint
@product_router.post("", response_model=ProductBaseIn)
async def update_products(product: ProductBaseIn, db: Session = Depends(get_db)):
    result_product = create_product(db, product)
    return result_product
```


### 2. product_handler.py

* 這邊我們要使用 `pydantic` 的規範格式，因此要引入 `ProductBase` 的 `class`
* step 1: 使用者 `post query` 資料進來後，先把一堆的字串整理成 `dict` 格式
* step 2: 等等更新的時候，可以知道產品的 `ec_id 和 key`
* step 3: 先確認這次更新的產品，是否有存在資料庫，如果有，就用 `setattr` 更新產品的內容，如果沒有就直接新增一個到資料庫裡面

```py
# FASTAPI/sql_alchemy/product_handler.py

import sys
import os
from sqlalchemy.orm import Session
from sql_alchemy import Website, Product
import logging
from datetime import timedelta
from pydantic_extra_types.currency_code import ISO4217
from lib.src.shared import Availability
from fastapi.encoders import jsonable_encoder

current_dir = os.path.abspath(os.path.dirname(__file__))
target_dir = os.path.abspath(os.path.join(current_dir, os.pardir))
sys.path.append(target_dir)

from lib.src.shared import ProductBase as ProductBaseIn


# step 1
def format_products(product: ProductBaseIn):    
    product_dict = product.dict()
    return product_dict


# step 2
def get_product_id_info(product: ProductBaseIn):
    return { "ec_id": product.ec_id, "key": product.key }


# step 3
def create_product(db: Session, product: ProductBaseIn):
    product_dict = format_products(product)
    ec_id = product_dict["ec_id"]
    key = product_dict["key"]

    existing_product = (
        db.query(Product).filter(ec_id=ec_id, key=key).first()
    )

    if existing_product:
        print((f"updating product: {get_product_id_info(product)}"))

        for key, value in product_dict.items():
            setattr(existing_product, key, value)
        
        db.commit()
        print(f"product updated: {get_product_id_info(product)}")
        return existing_product
    else:
        print(f"creating product {get_product_id_info(product)}")
        db_product = Product(**jsonable_encoder(product_dict))

        db.add(db_product)
        db.commit()
        db.refresh(db_product)
        print(f"product created: {get_product_id_info(product)}")

        return db_product
```


### 3. dataflow 資料流

上面那樣寫可能會有一點不懂，現在把每一步驟的資料流寫出來: 

#### 步驟一

直接在終端機上，執行想要輸入的產品內容

```shell
curl -X 'POST' \
  'http://0.0.0.0:8000/products' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "ec_id": 100,
  "key": "product_002",
  "title": "Sample Product 10",
  "price": 1,
  "currency": "USD",
  "extra": "string",
  "availability": "In Stock",
  "custom_labels": {
    "label_0": "string",
    "label_1": "string",
    "label_2": "string",
    "label_3": "string",
    "label_4": "string"
  }
}'
```


#### 步驟二

接著就會進到這個 `endpoint` 的設定裡面

```python
@product_router.post("", response_model=ProductBaseIn)
async def update_products(product: ProductBaseIn, db: Session = Depends(get_db)):
    result_product = create_product(db, product)
```

#### 步驟三

首先會進到 `create_product` 這個 `function` 裡面
```py
def create_product(db: Session, product: ProductBaseIn):
    product_dict = format_products(product)                 => 這
```

接著因為我們需要把 `query` 進來的一堆資料，轉成 `dict`，因此接著進到 `format_products`，這邊我們把進來的資料和格式化後的資料都印出來，這樣非常清楚

```py
def format_products(product: ProductBaseIn):    
    print(product, "******* product *******")
    # ec_id=100 key='product_002' title='Sample Product 10' price=1.0 currency='USD' extra='string' availability=<Availability.IN_STOCK: 'In Stock'> custom_labels=CustomLabels(label_0='string', label_1='string', label_2='string', label_3='string', label_4='string') ******* product *******
    product_dict = product.dict()

    print(product_dict, "========= product_dict =========")
    # {'ec_id': 1878, 'key': 'product_212', 'title': 'Sample Product 212', 'price': 1020.0, 'currency': 'AED', 'extra': 'string', 'availability': <Availability.IN_STOCK: 'In Stock'>, 'custom_labels': {'label_0': 'string', 'label_1': 'string', 'label_2': 'string', 'label_3': 'string', 'label_4': 'string'}}    
    return product_dict
```



#### 步驟四

跑完 `format_products` 後，可以得到整理過後的 `dict`，我們先把 `ec_id 和 key` 抓出來，要用這兩個值來分辨，目前資料庫裡面存的產品是否存在 - `existing_product`。  
    
如果存在，用 `setattr` 這個 `function` 更新原本產品的值，如果不存在，就直接新增進資料。  

Ps. 在新增或更新期間，我們會用這個函式 `get_product_id_info` 來偵測被新增或更新產品的資訊

```py
def create_product(db: Session, product: ProductBaseIn):    
    product_dict = format_products(product)                         => 現在在這邊

    ec_id = product_dict['ec_id']
    key = product_dict['key']

    existing_product = (
        db.query(Product).filter_by(ec_id=ec_id, key=key).first()
    )

    if existing_product:
        print((f"updating product: {get_product_id_info(product)}"))

        for key, value in product_dict.items():
            setattr(existing_product, key, value)
    
        db.commit()

        print(f"product updated: {get_product_id_info(product)}")
        return existing_product
    else:
        print(f"creating product {get_product_id_info(product)}")
        # creating product: {'ec_id': 100, 'key': 'product_002'}

        db_product = Product(**jsonable_encoder(product_dict))

        db.add(db_product)
        db.commit()
        db.refresh(db_product)

        print(f"product created: {get_product_id_info(product)}")
        # product created: {'ec_id': 100, 'key': 'product_002'}

        return db_product
```




### 4. 執行 main.py 

fastAPI 有兩種方式可以直接測試 `post` 的方法！

#### 4-1 第一種: http://0.0.0.0:8000/docs#/

進到 `fastAPI` 特別提供的 docs 連結，找到你設定的 `post endpoint`，並直接輸入你想要新增的資料就可以了

#### 4-2 第二種: curl

在終端機輸入以下指令，可以做到 `post` 的效果

執行 `curl post` 指令:  
```shell
curl -X 'POST' \
  'http://0.0.0.0:8000/products' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "ec_id": 100,
  "key": "product_002",
  "title": "Sample Product 10",
  "price": 1,
  "currency": "USD",
  "extra": "string",
  "availability": "In Stock",
  "custom_labels": {
    "label_0": "string",
    "label_1": "string",
    "label_2": "string",
    "label_3": "string",
    "label_4": "string"
  }
}'
```



