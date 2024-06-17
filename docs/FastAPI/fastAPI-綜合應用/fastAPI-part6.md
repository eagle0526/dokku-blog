---
sidebar_position: 6
---

## part6 - 創建第二個 table

在 part6 我們才剛幫 Website 新增一個欄位，現在我們直接來新增第二個 table

## 新增 table

### 1. 先設定 table 的細節

```py
# sql_alchemy/sql_alchemy.py

# ...省略

class Product(Base):
    __tablename__ = "product"
    id = Column(Integer, primary_key=True, autoincrement=True)
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

### 2. 更新 env.py 檔案

修改這邊是為了要讓 `alembic` 知道，現在你的 `table` 總共有幾個要控制，而因為現在有 2 個 table，所以我們直接指定 `Base` 這個資料。    


```py
# alembic/env.py

from sql_alchemy.sql_alchemy import Base
target_metadata = Base.metadata
```



還記得 `Base` 是啥嗎？就是這個：   
```py
# sql_alchemy/sql_alchemy.py
Base = declarative_base()
```


### 3. migration 指令

#### 3-1 先生成 migration 檔案
```shell
$ alembic revision --autogenerate -m "Add Product table"
```

#### 3-2 實際產出

```shell
$ alembic upgrade head
```



### 4. 生成測試資料

我們現在來生成 10 筆假資料，並且塞到 table 中：

```py
# sql_alchemy/build_data.py


# ... 省略

def build_product_data():
    session = create_session()

    session.query(Product).delete()
    session.commit()    

    def generate_random_product():

        return Product(
            key=f"product_{random.randint(1, 1000)}",
            currency=random.choice(["USD", "EUR", "JPY", "TWD"]),
            title=f"Sample Product {random.randint(1, 100)}",
            price=random.randint(0, 100),
            extra={"color": random.choice(["red", "green", "blue"]), "size": random.choice(["S", "M", "L"])},
            availability=random.choice(["In Stock", "Out Of Stock", "Preorder"]),
            custom_labels={f"label_{i}": None for i in range(5)},
            publish_time= datetime.now(),
            update_time= datetime.now(),
        )

    for _ in range(10):
        product = generate_random_product()
        session.add(product)

    session.commit()
    session.close()

if __name__ == "__main__":
    # 從 0 建立 data
    # build_website_data()

    # 更新資料
    # update_website_data()

    build_product_data()    
```

Ps. 這邊我有稍微修改一些變數的名稱喔




### 5. 執行程式

修改好後，在執行這個檔案：
```shell
$ python sql_alchemy/build_data.py 
```
最後看 table，就可以發現多了一個 table，並且這個 table 裡面都有資料









