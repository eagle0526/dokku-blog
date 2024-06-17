---
sidebar_position: 8
---

## part8 - 用 router 搭配 pydantic，藉此融合出一個 endpoint

前面我們設定好好幾個 Product 的 endpoint，而且也把好多參數都加上去了，現在我們要來搭配 `pydantic`，讓 queryString 輸入的時候可以有規則的被限制




## 建立 pydantic 資料夾


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
        | -- product_router.py
        | -- main.py             
  |  -- virtualenv
  |  app.py
```


### 2. 新增 lib/src/shared/__init__.py 檔案

* 多新增一系列 `pydantic` 的相關資料夾和檔案

```md
| FASTAPI
  |  -- lib                                     => 新增下面這幾個檔案
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
        | -- product_router.py
        | -- main.py             
  |  -- virtualenv
  |  app.py
```


### 3. 安裝 Pydantic package

```shell
$ pip install pydantic
```

### 4. __init__.py 檔案內容

以下都是用 Pydantic 來對每個欄位做屬性驗證，這樣設定好後，之後在呼叫 api 時，每個參數給個值就必須跟這邊設定的一樣才能成功呼叫

```py
# enum 的 package
from enum import Enum
# 當你不確定欄位的類型時，可以使用 any，他可以兼容所有的屬性，會忽略類型不匹配的問題
from typing import Any
# 可以去看 fastAPI-Pydantic-model 這個檔案的介紹
from pydantic import BaseModel, Field, HttpUrl
# 幣值的 package，使用 ISO4217 類型可以驗證和限制輸入的貨幣代碼，使其符合 ISO 4217 標準。例如，USD 代表美元，EUR 代表歐元
from pydantic_extra_types.currency_code import ISO4217

class Availability(str, Enum):
    IN_STOCK = "In Stock"
    OUT_OF_STOCK = "Out Of Stock"
    PREORDER = "Preorder"

class CustomLabels(BaseModel):
    label_0: str | None = None
    label_1: str | None = None
    label_2: str | None = None
    label_3: str | None = None
    label_4: str | None = None    

class ProductBase(BaseModel):
    ec_id: int
    key: str
    title: str
    price: float = Field(gt=0.0)
    currency: ISO4217
    extra: Any | None = None
    availability: Availability = Availability.IN_STOCK
    custom_labels: CustomLabels | None = None
```


## 建立新的 endpoint 給 pydantic 用


### 1. 新增在 product_router.py 這個檔案

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
        | -- product_router.py                  => 新增在這個檔案裡面
  |  -- virtualenv
  |  app.py
```

### 2. product_router.py 檔案內容修改

雖然在 part7，我們已經把 router 的前置設定都寫好了，但是由於要把 `pydantic` 的 `class` 引入，這邊要先來做一些處理，才可以 `import`：

#### 2-1 import ProductBase

由於檔案層級的不一樣，所以我們要先移動到最上層，再 `import ProductBase` 才行
```py
import sys
import os

current_dir = os.path.abspath(os.path.dirname(__file__))                # ..../fastAPI/sql_alchemy
target_dir = os.path.abspath(os.path.join(current_dir, os.pardir))      # ..../fastAPI

# 把當前的父資料夾加到 python 模塊搜索路徑中
sys.path.append(target_dir)

from lib.src.shared import ProductBase as ProductIn
from lib.src.shared import Availability


# ...忽略其他 import package

```

#### 2-2 寫 endpoint

這邊可以看到跟前面最大的不同，就是在寫 router 的時候，直接給他 `response_model` 並且值是用 `pydantic` 設定好的 `ProductBase`，這樣設定後，如果你今天在對資料庫進行查找的時候，只要跟 `pydantic` 規定的類型不一樣，就會噴出錯誤。     

```py
@product_router.get("", response_model=list[ProductBaseIn])
async def get_products(
    ec_ids: str | None = None,    
    key: str | None = None,
    currency: ISO4217 | None = None,
    availability: Availability = Availability.IN_STOCK,
    p_date_gte: str | None = None,
    p_date_lte: str | None = None,
    u_date_gte: str | None = None,
    u_date_lte: str | None = None,
    limit: int = 30000,
    db: Session = Depends(get_db),
):
    
    q = db.query(Product)

    if ec_ids:
        ec_id_list = ec_ids.split(",")
        q = q.filter(Product.ec_id.in_(ec_id_list))

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

##### ec_ids: str | None = None 介紹

這邊還有一個很特別的地方，就是原本我設定的 `ec_id 參數` 設定是這樣：`ec_id: int | None = None`，現在改成 `ec_ids: str | None = None` 這樣，並且在多設定了一段：
```py
if ec_ids:
    ec_id_list = ec_ids.split(",")
    q = q.filter(Product.ec_id.in_(ec_id_list))
```
這樣當我在輸入路徑的時候，就可以同時 `query` 出好幾家的商品，像這樣：`http://0.0.0.0:8000/products?ec_ids=2990,3151`，這樣我就可以同時抓出 `2990 和 3151` 的產品！


##### pydantic + router 數據流

不過這時候有人應該會有疑問，明明一開始在 pydantic 那邊設定的 class， `ec_id` 設定的型態是 `int`，為什麼這邊的 `ec_ids` 可以設定成 `str`？    
原因是從查詢參數中獲取的 `ec_ids` 字符串會在拆分成列表時進行轉換，因此 `查詢參數` 本身作為字符串進來，但在進行查詢時會被轉換為整數列表，這與您的 `Pydantic` 模型定義是兼容的。   
     
具體的數據流是：     
     
1. 查詢參數 ec_ids 作為字符串進來。
2. 將 ec_ids 拆分成一個包含 ID 的字符串列表。
3. 在查詢過程中，將這些字符串 ID 轉換為整數來進行查詢。
4. 查詢結果返回時，每個 Product 對象的 ec_id 都是整數，這符合您的 ProductBase 定義。



#### 2-3 抽象化 get_products 這一大段 router 的設定

由於 get_products 寫的有點臭長，我們把他拆開到另外一個 function 裡面，首先來新增另外一個檔案 - product_handler.py


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
        | -- product_handler.py                  => 新增在這個檔案裡面
        | -- product_router.p
  |  -- virtualenv
  |  app.py
```

* 新增好後，我們來修改這個 `product_handler.py` 檔案:


```py
# FASTAPI/sql_alchemy/product_handler.py

from sqlalchemy.orm import Session
from sql_alchemy import Website, Product
from pydantic_extra_types.currency_code import ISO4217
from lib.src.shared import Availability

def handle_products(
    ec_ids: list[str] | None,
    key: str | None,
    currency: ISO4217 | None,
    availability: Availability,
    p_date_gte: str | None,
    p_date_lte: str | None,
    u_date_gte: str | None,
    u_date_lte: str | None,
    limit: int,
    db:Session,
):
    q = db.query(Product)

    if ec_ids and len(ec_ids):
        ec_id_list = ec_ids.split(",")
        q = q.filter(Product.ec_id.in_(ec_id_list))
    
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

* 再來修改原本那一段超長的路徑設定

```py
# FASTAPI/sql_alchemy/product_router.py

@product_router.get("", response_model=list[ProductBaseIn])
async def get_products(
    ec_ids: str | None = None,    
    key: str | None = None,
    currency: ISO4217 | None = None,
    availability: Availability = Availability.IN_STOCK,
    p_date_gte: str | None = None,
    p_date_lte: str | None = None,
    u_date_gte: str | None = None,
    u_date_lte: str | None = None,
    limit: int = 30000,
    db: Session = Depends(get_db),
):
    products = handle_products(
        ec_ids=ec_ids,
        key=key,
        currency=currency,
        availability=availability,
        p_date_gte=p_date_gte,
        p_date_lte=p_date_lte,
        u_date_gte=u_date_gte,
        u_date_lte=u_date_lte,
        limit=limit,
        db=db,
    )

    return products
```

這樣我們的路徑處理就非常漂亮了！！

### 3. 執行 main.py 

執行 main.py 資料後
```shell
$ python sql_alchemy/main.py 
```

輸入指定的 path，我想要抓出 ec_id 是 2990, 3151 的所有產品:
```md
http://0.0.0.0:8000/products?ec_ids=2990,3151
```






