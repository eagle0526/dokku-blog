---
sidebar_position: 10
---

## part10 - 設定 create product 的 publish_time, update_time 屬性

前面我們成功設定好了 `create 的 endpoint`，也確實可以直接透過該 endpoint 創造新的產品，但是會發現我們的產品少了兩個屬性，就是 `publish_time, update_time`，我們現在來新增這一項


## 新增 create endpoint 的 publish_time, update_time 屬性


### 1. 新增 schemas.py 資料夾

現在整個資料夾架構是這樣：

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
        | -- product_handler.py
        | -- product_router.py
  |  -- virtualenv
  |  app.py
```


我們來新增一個 `schemas.py` 檔案:

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
        | -- product_handler.py
        | -- product_router.py
        | -- schemas.py                     => 新增這個資料夾        
  |  -- virtualenv
  |  app.py
```

這個檔案的目的是要來新增有把時間型態加上去的 `response_model` 的。


### 2. schemas.py 檔案內容

這個檔案主要就是繼承 `ProductBase`，並把兩個需要更新的時間段加上去

```py
from datetime import datetime
import sys
import os

# 獲取當前文件所在的絕對路徑
current_dir = os.path.abspath(os.path.dirname(__file__))                    # /Users/yee0526/Desktop/python/fastAPI/sql_alchemy
target_dir = os.path.abspath(os.path.join(current_dir, os.pardir))          # /Users/yee0526/Desktop/python/fastAPI

# 把當前的副資料夾加到 python 模塊搜索路徑中
sys.path.append(target_dir)


from pydantic import BaseModel
from lib.src.shared import ProductBase

class ProductOut(ProductBase):
    publish_time: datetime
    update_time: datetime

    class ConfigDict:
        from_attributes = True
```





### 3. 修改 create 的 response_model

把 `response_model` 從原本的 `ProductBaseIn` 改成 `ProductOut`，這樣改後，進入資料庫前的型態驗證，就會加上時間的判斷
```py
# sql_alchemy/product_router.py

# ...省略

from schemas import 

@product_router.post("", response_model=ProductOut)
async def update_products(product: ProductBaseIn, db: Session = Depends(get_db)):
    result_product = create_product(db, product)
    return result_product
```

### 4. 修改 create_product 函式的內容

剛剛把 `response_model` 修改後，我們要來修改 `create_product` 這個函式，我們要把時間的設定塞進資料庫裡面

```py
# sql_alchemy/product_handler.py
from datetime import timedelta, datetime

def create_product(db: Session, product: ProductBaseIn):    
    product_dict = format_products(product)

    ec_id = product_dict['ec_id']
    key = product_dict['key']

    existing_product = (
        db.query(Product).filter_by(ec_id=ec_id, key=key).first()
    )

    current_time = datetime.now()                                       # => 加這行

    if existing_product:
        print((f"updating product: {get_product_id_info(product)}"))

        for key, value in product_dict.items():
            setattr(existing_product, key, value)

        existing_product.update_time = current_time                     # => 加這行
        db.commit()

        print(f"product updated: {get_product_id_info(product)}")
        return existing_product
    else:
        print(f"creating product {get_product_id_info(product)}")
        
        db_product = Product(**jsonable_encoder(product_dict))
        
        db_product.publish_time = current_time                          # => 加這行
        db_product.update_time = current_time                           # => 加這行

        db.add(db_product)
        db.commit()
        db.refresh(db_product)
        print(f"product created: {get_product_id_info(product)}")
        return db_product

```




### 5. 執行 main.py 

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
     
     
這次當你這樣輸入這些資料時，`publish_time` 和 `update_time` 會一起產生！
