---
sidebar_position: 11
---

## part11 - 設定 update product 的 endpoint

剛剛的 create 完成，現在來設定 update 的路徑


## 在 product_router.py 新增 endpoint 和在 product_handler.py 新增方法



### 1. product_router.py

* `db: Session = Depends(get_db)`      -> 這一段是讓你可以操作資料庫
* `products: list[ProductBaseIn]`      -> 這一段是 query 要填寫的參數，由於我們想設定的方式是，可以一次大量更新商品，所以設定成 list
* `update_all_product(db, products)`   -> 這是等等要創建的方法，query 輸入的資料，會由這個函數進去處理

```py
# FASTAPI/sql_alchemy/product_router.py

from fastapi import APIRouter, status, Depends, Query, HTTPException
from product_handler import handle_products, create_product, update_all_products
from schemas import ProductOut
# ... 省略

# update product endpoint
@product_router.put("", response_model=list[ProductOut])
async def update_products(products: list[ProductBaseIn], db: Session = Depends(get_db)):
    update_products = update_all_products(db, products=products)

    if not update_products:
      raise HTTPException(
        status_code = status.HTTP_404_NOT_FOUND,
        detail=f'None of products were updated'
      )
    return update_products
```



### 2. product_handler.py - update_all_products function

這一步驟先建立一個空的 `array`，並把剛剛從 `query` 傳進來的 `list 資料` 跑迴圈，接下來再用另外一個 function 處理這些資料

```py
# FASTAPI/sql_alchemy/product_handler.py 

# ...省略

def update_all_products(db= Session, products= list[ProductBaseIn]):
    updated_products = []
    try:
        for product in products:
            result = update_specify_product(db, product)

            if result:
                updated_products.append(result)
    
    except Exception as e:
        db.rollback()
        print("Exception", e)
    
    return updated_products
```


### 3. product_handler.py - update_specify_product function

介紹一下這一段資料在做什麼事情：

1. 經由剛剛 `update_all_products` 跑迴圈，一筆一筆送過來後，我們先把 query 來的資料丟進 `format_products` 來統一格式
2. 更新 `update_time` 設定為現在的時間
3. 設定好等等要更新的資料庫語法 - stmt 就是把 query 進來的資料，轉成等等可以被資料庫更新的語法
4. 如果今天有同一種商品，有兩種幣值以上，再更新該商品時，就會觸發 `product.currency`
5. 最後把整筆資料進行更新
6. 更新完後，抓出剛剛更新的那筆資料，並回傳


```py
# FASTAPI/sql_alchemy/product_handler.py 

# ...省略

from sqlalchemy import select, update

def update_specify_product(db=Session, product= ProductBaseIn):
    product_dict = format_products(product)
    product_dict['update_time'] = datetime.now()
    stmt = (
      update(Product).where(Product.ec_id == product.ec_id).where(Product.key == product.key)
    )

    if product.currency:
        stmt = stmt.where(Product.currency == product.currency)
          
    stmt = stmt.values(**jsonable_encoder(product_dict))

    db.execute(stmt)
    db.commit()

    fetch_result = (
      db.query(Product)
       .filter(Product.ec_id == product.ec_id)
       .filter(Product.key == product.key)
    ).first()

    return fetch_result
```


:::tip `product.currency` 這一段可能會有人不明白，我寫詳細一點
這個是設定給，如果今天有兩個同樣 `ec_id` 和 同樣 `key` 但是 `不同幣值的產品` 來做更新的，以下為範例：   
1. ec_id: 100, key: product_100, currency: USD   
2. ec_id: 100, key: product_100, currency: JPY      
   
這邊有兩個不同幣值，但 ec_id 和 key 都相同的產品，這時候如果你在更新產品時，固定住 currency 為 USD 或是 JPY - 就可以更新到對應的產品   
:::

### 4. dataflow 資料流

一樣來看一下整體資料流   

#### 步驟一

我們輸入 `put` 指令，並且陣列裡面有兩個產品資料，我一次更新兩樣產品：

```shell
curl -X 'PUT' \
  'http://0.0.0.0:8000/products' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '[
  {
    "ec_id": 101,
    "key": "product_991",
    "title": "double fourth 101",
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
  },
{
  "ec_id": 100,
  "key": "product_999",
  "title": "Sample Product fourth update 999",
  "price": 999,
  "currency": "JPY",
  "extra": "string",
  "availability": "In Stock",
  "custom_labels": {
    "label_0": "string",
    "label_1": "string",
    "label_2": "string",
    "label_3": "string",
    "label_4": "string"
  }
}

]'
```

#### 步驟二

接著就會進到這個 `endpoint` 的設定裡面

```python
@product_router.put("", response_model=list[ProductOut])
async def update_products(products: list[ProductBaseIn], db: Session = Depends(get_db)):
    update_products = update_all_products(db, products=products)

    if not update_products:
        raise HTTPException(
            status_code = status.HTTP_404_NOT_FOUND,
            detail=f"None of products were updated"
        )
    
    return update_products
```


#### 步驟三

首先會進到 `update_all_products` 這個 `function` 裡面，由於我們陣列裡面有兩個，因此讓他們兩個去跑迴圈：

```py
def update_all_products(db: Session, products: list[ProductBaseIn]):
    updated_products = []

    try:
        for product in products:
            result = update_specify_product(db, product)

            if result:
                updated_products.append(result)

    except Exception as e:
        db.rollback()
        print("Exception", e)

    return updated_products
```



#### 步驟四

進入迴圈後，就到這個函式裡面了 `update_specify_product`，並且我有把 `stmt` 印出來，可以看得出來就是轉換成 `SQL` 更新的語法

```py
def update_specify_product(db: Session, product: ProductBaseIn):
    product_dict = format_products(product)
    product_dict['update_time'] = datetime.now()
    stmt = (
        update(Product).where(Product.ec_id == product.ec_id).where(Product.key == product.key)
    )

    print(stmt, "========== stmt")
    # UPDATE product SET id=:id, ec_id=:ec_id, key=:key, title=:title, price=:price, currency=:currency, extra=:extra, availability=:availability, custom_labels=:custom_labels, publish_time=:publish_time, update_time=:update_time WHERE product.ec_id = :ec_id_1 AND product.key = :key_1 ========== stmt


    if product.currency:
        stmt = stmt.where(Product.currency == product.currency)

    stmt = stmt.values(**jsonable_encoder(product_dict))
    print(stmt, "============ jsonable_encoder")
    # UPDATE product SET ec_id=:ec_id, key=:key, title=:title, price=:price, currency=:currency, extra=:extra, availability=:availability, custom_labels=:custom_labels WHERE product.ec_id = :ec_id_1 AND product.key = :key_1 AND product.currency = :currency_1 ============ jsonable_encoder


    db.execute(stmt)
    db.commit()

    fetch_result = (
        db.query(Product)
        .filter(Product.ec_id == product.ec_id)
        .filter(Product.key == product.key)
        .first()
    )

    return fetch_result
```

### 5. 執行 main.py 

fastAPI 有兩種方式可以直接測試 `put` 的方法！

#### 4-1 第一種: http://0.0.0.0:8000/docs#/

進到 `fastAPI` 特別提供的 docs 連結，找到你設定的 `put endpoint`，並直接輸入你想要新增的資料就可以了

#### 4-2 第二種: curl

在終端機輸入以下指令，可以做到 `post` 的效果

執行 `curl put` 指令:  

```shell
curl -X 'PUT' \
  'http://0.0.0.0:8000/products' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '[
  {
    "ec_id": 101,
    "key": "product_991",
    "title": "double fourth 101",
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
  },
{
  "ec_id": 100,
  "key": "product_999",
  "title": "Sample Product fourth update 999",
  "price": 999,
  "currency": "JPY",
  "extra": "string",
  "availability": "In Stock",
  "custom_labels": {
    "label_0": "string",
    "label_1": "string",
    "label_2": "string",
    "label_3": "string",
    "label_4": "string"
  }
}

]'
```
     
