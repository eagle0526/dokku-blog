---
sidebar_position: 12
---

## part12 - 設定 delete product 的 endpoint

最後我們把 delete 的 endpoint 補上，就大功告成了！


## 在 schemas.py 新增 queryFormat, 在 product_router.py 新增 endpoint, 在 product_handler.py 新增方法


### 1. schemas.py

由於刪除資料需要的 query 參數，不需要像前面的產品一樣，需要這麼豐富的資料，因此我們另外開一個 class 來設定 delete 需要的 query 參數。

ps. 刪除資料需要的參數，包含 `ec_id, key, currency`
```py
# FASTAPI/sql_alchemy/schemas.py

# 省略

class ProductUniqueConstraint(BaseModel):
    ec_id: int
    key: str
    currency: str

    class ConfigDict:
        from_attributes = True
```



### 2. product_router.py

* `db: Session = Depends(get_db)`      -> 這一段是讓你可以操作資料庫
* `products: list[ProductBaseIn]`      -> 這一段是 query 要填寫的參數，由於我們想設定的方式是，可以一次大量更新商品，所以設定成 list
* `update_all_product(db, products)`   -> 這是等等要創建的方法，query 輸入的資料，會由這個函數進去處理

```py
# FASTAPI/sql_alchemy/product_router.py

from fastapi import APIRouter, status, Depends, Query, HTTPException
from product_handler import handle_products, create_product, update_all_products, delete_all_products
from schemas import ProductOut, ProductUniqueConstraint
# ... 省略

# delete product endpoint
@product_router.delete("")
async def delete_products(uc_products: list[ProductUniqueConstraint], db: Session = Depends(get_db)):
    delete_products = delete_all_products(db, uc_products)

    print("product deleted")
    return delete_products
```

### 3. product_handler.py - delete_all_products function

這一段是當 endpoint 的參數傳進來時，先進到這個裡面，會先跑一個迴圈，接著進到 `delete_specify_product` 這個 function 中

```py
# FASTAPI/sql_alchemy/product_handler.py 

# ...省略

def delete_all_products(db= Session, products= list[ProductUniqueConstraint]):
    deleted_products = []
    try:
        for product in products:
            result = delete_specify_product(db, product.ec_id, product.key, product.currency)
            
            deleted_products.append(result)

    except Exception as e:
        db.rollback()
        print("Exception", e)
    
    return deleted_products
```


### 4. product_handler.py - delete_specify_product function

介紹一下這一段資料在做什麼事情：

1. 這先很簡單，接受從剛剛回圈過來的每一筆資料
2. 使用 delete 的語法，並設定 sql 刪除的格式
3. 最後執行刪除指令


```py
# FASTAPI/sql_alchemy/product_handler.py 

# ...省略

from sqlalchemy import select, update, delete

def delete_specify_product(db=Session, ec_id: int, key: str, currency: None):
    print(
        f"deleting product ec_id: {ec_id}, key: {key}, currency: {currency}"
    )    

    stmt = (
      delete(Product).where(Product.ec_id = ec_id).where(Product.key == key)
    )    

    if currency:
        stmt = stmt.where(Product.currency == currency)

    print(stmt, "================= stmt")

    result = db.execute(stmt)
    db.commit()

    return result
```


### 5. dataflow 資料流

一樣來看一下整體資料流   

#### 步驟一

我們輸入 `delete` 指令，並且陣列裡面有兩個產品資料，我一次更新兩樣產品：

```shell
curl -X 'DELETE' \
  'http://0.0.0.0:8000/products' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '[
  {
    "ec_id": 1878,
    "key": "product_212",
    "currency": "AED"
  }
]'
```

#### 步驟二

接著就會進到這個 `endpoint` 的設定裡面

```python
@product_router.delete("")
async def delete_products(
    uc_products: list[ProductUniqueConstraint],
    db: Session = Depends(get_db),
):
    product = delete_all_products(db, uc_products)
    print("product deleted")

    return product
```


#### 步驟三

首先會進到 `delete_all_products` 這個 `function` 裡面

```py
def delete_all_products(db: Session, products: list[ProductUniqueConstraint]):
    deleted_results = []

    try:
        for product in products:
            result = delete_specify_product(db, product.ec_id, product.key, product.currency)
            deleted_results.append(result)
    except Exception as e:
        db.rollback()
        print("Exception", e)

    return deleted_results

```



#### 步驟四

進入迴圈後，就到這個函式裡面了 `update_specify_product`，並且我有把 `stmt` 印出來，可以看得出來就是轉換成 `SQL` 更新的語法

```py
def delete_specify_product(db: Session, ec_id: int, key: str, currency=None):

    print(
        f"deleting product ec_id: {ec_id}, key: {key}, currency: {currency}"
    )
    # deleting product ec_id: 1878, key: product_212, currency: AED

    stmt = (
        delete(Product).where(Product.ec_id == ec_id).where(Product.key == key)
    )

    if currency:
        stmt = stmt.where(Product.currency == currency)

    print(stmt, "================= stmt")
    # DELETE FROM product WHERE product.ec_id = :ec_id_1 AND product.key = :key_1 AND product.currency = :currency_1 ================= stmt
        
    result = db.execute(stmt)
    db.commit()

    return result
```

### 5. 執行 main.py 

fastAPI 有兩種方式可以直接測試 `delete` 的方法！

#### 4-1 第一種: http://0.0.0.0:8000/docs#/

進到 `fastAPI` 特別提供的 docs 連結，找到你設定的 `delete endpoint`，並直接輸入你想要新增的資料就可以了

#### 4-2 第二種: curl

在終端機輸入以下指令，可以做到 `post` 的效果

執行 `curl put` 指令:  

```shell
curl -X 'DELETE' \
  'http://0.0.0.0:8000/products' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '[
  {
    "ec_id": 1878,
    "key": "product_212",
    "currency": "AED"
  }
]'
```
     
