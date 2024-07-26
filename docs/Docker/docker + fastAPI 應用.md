---
sidebar_position: 4
---


Docker + fastAPI 應用
------

## 初始架構

首先我們先在空的資料夾中，建立兩個兩個檔案：

```md
| FASTAPI_Docker
  |  Dockerfile
  |  main.py
```

## 第一版 dockerfile - init dockerfile
### dockerfile 內容

```dockerfile
# Dockerfile

FROM python:3.10-slim
```

一開始只先設定這樣，先看一下等我們 build image 後，裡面的內容會長什麼樣子：

### 建立 image + container

#### image
以下這個 `.` 表示去找當前路徑最接近的 Dockerfile，建立他的 image 檔案，假設今天你的整個檔案今天有很多個 `Dockerfile`，建議使用 `-f` 這個指令來讓系統知道你想要建立的 `Dockerfile` 是哪一個

```shell
$ docker build -t fastapi:latest .
```

#### container
* name 是幫現在建立的 container 命名
* fastapi:latest - 是指你要吃的 image 檔案名稱
* bash 是指當你把容器建立好後，要不要直接進入 bash 環境

```shell
$ docker run -it --name fastapi-docker fastapi:latest bash

-----
root@9065ad0857ce:/# ls
bin  boot  dev  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

root@9065ad0857ce:/# exit
```

:::tip
這邊可以特別說一下，進到剛剛建立好的 image 後，會發現裡面有很多個資料夾，這些資料夾就是初始你建立 docker 時會有的
:::

#### exec 進入容器

如果今天離開了容器，又想進入後，可以輸入以下指令，再進入 bash 環境一次，記得要先啟動 container 才能進入喔

```shell
$ docker container exec --interactive --tty fastapi-docker bash
```



## 新增 pyproject.toml 檔案

先新增 `pyproject` 檔案，這個檔案的作用就是下載 package 用的，就是 pip 的 requirement：

```md
| FASTAPI_Docker
  |  Dockerfile
  |  pyproject.toml  
  |  main.py
```

再來補上裡面的內容，順面介紹一下裡面的 package 都是做啥用的：

* python: python 的版本
* fastapi: fastapi 的版本
* SQLAlchemy: 可以讓 python 用於關聯式資料庫進行互動 (使用 ORM)
* alembic: 資料庫 migration 的工具，幫忙做版本控制
* pydantic: Pydantic 主要是拿來做資料的驗證與設定，可幫你驗證資料的 data type ，及是否符合規則 (像是對應欄位是否為 emil)。
* PyMySQL: 讓 python 可以連接資料庫
* uvicorn: 這是一個 ASGI 服務器，用於運行 FastAPI 應用

```toml
[tool.poetry]
name = "fastAPI"
version = "0.1.0"
description = "fastAPI"
authors = ["eagle <john0526@tagtoo.com>"]

[tool.poetry.dependencies]
python = "^3.9"
fastapi = "0.104.1"
SQLAlchemy = "2.0.30"
alembic = "1.13.1"
pydantic = "2.5.2"
PyMySQL = "1.1.0"
uvicorn = "0.23.2"

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
```

Ps. 記住 name, version 那些都要填寫，要不然我 build 不起來




## 第二版的 dockerfile - 安裝 package

接下來 `docker` 這樣修改，我會詳細描述每一步驟都在做啥：

```dockerfile
FROM python:3.9-slim

WORKDIR /srv

COPY pyproject.toml .

ENV PIP_DISABLE_PIP_VERSION_CHECK=1
ENV PIP_NO_CACHE_DIR=1
# for slim image
ENV POETRY_VIRTUALENVS_CREATE=0

RUN pip install -U poetry \
    && poetry install --only main \
    && rm -rf ~/.cache/pypoetry \
    && apt-get update \
    && apt-get install -y nano

COPY . .
```

#### FROM
* 首先先把 python:3.9 版本給拉下來

#### WORKDIR
* 接著我們直接指定我們要在哪個資料夾運行 (還記得我們前面有把 docker 檔案全部印出來，其中就有 /srv)

#### COPY
* 把 `pyproject.toml` 這個檔案複製一份進到 docker 裡面 (COPY 左邊參數是本地端檔案，右邊參數是 dockerfile 的檔案)

#### ENV
* 再來設定一些環境變數，要設定變數的原因，可以到 `tagtoo/cron-job-v2` 這個檔案位置看原因，就不再說一次

#### RUN
* 安裝 `poetry` 套件，這個套件是類似 pip，協助進行 package 管理，不過功能比 pip 還強大很多
* 相較於 `pip 的 requirement`， poetry 的就是 `pyproject.toml`，因此 `poetry install` 就是在安裝 `pyproject` 裡面寫好的 package
* rm -rf : 清理 pypoetry 裡面的緩存文件
* apt-get update: 確保在運行 `apt-get install` 命令之前，系統知道最新的 package version。這樣可以避免安裝過時或有潛在安全問題的 package。
* 安裝 nano 這個 package

#### COPY
* 最後在補一個 `COPY . .`，把整份檔案複製進 docker 裡面



### 重新執行 docker build 指令

記得要把前一個 image 給刪掉，前一個只是測試是否能 build image 測試用的：

* 建立好 image, container 後，進入 container 裡面，可以看到裡面有我們本地端所有的檔案

```shell
$ docker build -t fastapi:latest .
$ docker run -it --name fastapi-docker fastapi:latest bash

------
root@c49805e823d0:/srv# ls
Dockerfile  db-oprator  main.py  poetry.lock  pyproject.toml

```

* 並且可以輸入 `poetry show` 指令，查看目前有哪些 package 已經被 poetry 裝好
```shell
root@ 49805e823d0:/srv# poetry show
-----
alembic           1.13.1  A database migration tool for SQLAlchemy.
annotated-types   0.7.0   Reusable constraint types to use with typing.Annotated
anyio             3.7.1   High level compatibility layer for multiple asyn
...
...

```

PS. 這邊要特別注意，現在這種建立容器的方法，會讓你更新檔案後，就要重 build 一次，如果不想要每次都重 build，可以使用 `bind` 的方法



## 設定 fastapi 基礎檔案

目前的架構為這樣，現在已經可以建立基本的環境了 (dockerfile)，因此現在來把 fastapi 的檔案給加上去

```md
| FASTAPI_Docker
  |  Dockerfile
  |  pyproject.toml  
  |  main.py
```

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

## 第三版的 dockerfile - 設定 port 

這一段的目的是，讓我們啟用 docker 的時候，可以自動啟用 host，也就是可以自動打開 `port`

```dockerfile
FROM python:3.9-slim

WORKDIR /srv

COPY pyproject.toml .

ENV PIP_DISABLE_PIP_VERSION_CHECK=1
ENV PIP_NO_CACHE_DIR=1
# for slim image
ENV POETRY_VIRTUALENVS_CREATE=0

RUN pip install -U poetry \
    && poetry install --only main \
    && rm -rf ~/.cache/pypoetry \
    && apt-get update \
    && apt-get install -y nano

COPY . .

CMD ["uvicorn", "DBoperator.app:app", "--host", "0.0.0.0", "--port", "8000"]  #=> 加上這一段
```


### 重新執行 docker build 指令

由於修改了檔案內容，因此需要重新 `build image, container`

```shell
$ docker build -t fastapi:latest .
$ docker run -it --name fastapi-docker -p 8000:8000 fastapi:latest
```

這樣執行後，可以在網址輸入 `http://127.0.0.1:8000/docs#`，應該會出現剛剛在 `app.py` 設定好的 `endpoint`。  
     
BUT!!!! 這個會有一個問題，就是這時候如果你打這個路徑 `http://127.0.0.1:8000/get_products?ec_id=2990`，會發生錯誤，內容是你不能連結 `docker - MySQL`，出現像這樣的訊息：  

```shell
# ...省略
packages/pymysql/connections.py", line 358, in __init__
    self.connect()
  File "/usr/local/lib/python3.9/site-packages/pymysql/connections.py", line 711, in connect
    raise exc

sqlalchemy.exc.OperationalError: (pymysql.err.OperationalError) (2003, "Can't connect to MySQL server on 'localhost' ([Errno 99] Cannot assign requested address)")
```
     
     
因此接下來要解決這件事情。       



## 第四版的 dockerfile -> 要加上 docker-compose.yml

目前的整體檔案架構：
 
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

### 新增 docker-compose.yml

先說一下現在要做啥，我們現在就是要解決 `docker` 連不到 `本機端資料庫` 的問題，會有這個問題是因為 `docker` 的特性 - `容器隔離屬性` 和 `network` 設定的關係，所以才不能直接讓 `docker container` 連結到本機端的資料庫。    

因此來新增 `docker-compose.yml` 這個檔案：   

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
  |  docker-compose.yml         => 新增這個資料夾
```

#### docker-compose.yml 檔案內容
這邊介紹一下為什麼要這樣寫，因為我們現在要串接兩個服務，一個就是 `mysql 的 db`，一個就是我們的 `fastapi`，這樣就可以很輕鬆的讓我們的 `資料庫 和 fastapi` 的網路設定連接起來。    
ps. 在這個例子中，fastapi 服務依賴於 mysql 服務，並且可以使用服務名稱 mysql 來訪問 MySQL 數據庫  

```yaml
version: '3.8'

services:
  db:
    image: mysql:latest
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: test
      MYSQL_USER: newuser
      MYSQL_PASSWORD: newpassword
    ports:
      - "3306:3306"

  fastapi:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      - MYSQL_HOST=db
      - MYSQL_USER=newuser
      - MYSQL_PASSWORD=newpassword
      - MYSQL_PORT=3306
      - MYSQL_DB=test
```



### 重新執行 docker build 指令
接下來我們就來輸入以下指令，讓 `image` 重新建立起來：

```shell
$ docker-compose up --build
```

這個指令會依照你的 `docker-compose.yml` 檔案，建立 `image`。     


### 進入 endpoint 點

最後我們再一次輸入這個 `endpoint: http://127.0.0.1:8000/get_products?ec_id=2990`，這時候會發現，剛剛的 `connect error` 已經不見了，但！！！ㄅ這次會出現另外一個錯誤～像這樣：

```shell
# ...省略
packages/pymysql/err.py", line 143, in raise_mysql_exception
fastapi-1  |     raise errorclass(errno, errval)
fastapi-1  | sqlalchemy.exc.ProgrammingError: (pymysql.err.ProgrammingError) (1146, "Table 'test.product' doesn't exist")
fastapi-1  | [SQL: SELECT product.id AS product_id, product.ec_id AS product_ec_id, product.key AS product_key, product.title AS product_title, product.price AS product_price, product.currency AS product_currency, product.extra AS product_extra, product.availability AS product_availability, product.custom_labels AS product_custom_labels, product.publish_time AS product_publish_time, product.update_time AS product_update_time 
fastapi-1  | FROM product 
fastapi-1  | WHERE product.ec_id = %(ec_id_1)s]
fastapi-1  | [parameters: {'ec_id_1': '2990'}]
fastapi-1  | (Background on this error at: https://sqlalche.me/e/20/f405)
```


簡單說這個錯誤是什麼，這個錯誤就是我們 `docker` 裡面的 `db`，並沒有 `Product` 這個 `table`，因此我們接下來要多加一行程式，讓我們在建立 `image` 的時候，順便把 `Product` 這個 `table` 建立完成    


### 修改 app.py 檔案

新增的這行這行的作用就是建立所有指定的 `table` 欄位：

```py
# ...省略
from .database import SessionLocal, Base, primary_engine    # 修改這行

app = FastAPI() # 建立一個 Fast API application

# 創建 table
Base.metadata.create_all(bind=primary_engine)               # 新增這行


# ...省略
```



### 再次建立容器、進入 endpoint 點

記得要刪除原本的 `container 和 image` 喔～

* 建立 image, container
```shell
$ docker-compose up --build
```

* 進入 endpoint

```shell
curl -X 'GET' \
  'http://0.0.0.0:8000/get_products?ec_id=2990' \
  -H 'accept: application/json'

------
fastapi-1  | 2024-07-08 07:02:28,538 INFO sqlalchemy.engine.Engine SELECT product.id AS product_id, product.ec_id AS product_ec_id, product.`key` AS product_key, product.title AS product_title, product.price AS product_price, product.currency AS product_currency, product.extra AS product_extra, product.availability AS product_availability, product.custom_labels AS product_custom_labels, product.publish_time AS product_publish_time, product.update_time AS product_update_time 
fastapi-1  | FROM product 
fastapi-1  | WHERE product.ec_id = %(ec_id_1)s
fastapi-1  | 2024-07-08 07:02:28,538 INFO sqlalchemy.engine.Engine [generated in 0.00049s] {'ec_id_1': '2990'}
fastapi-1  | ********** results []
fastapi-1  | INFO:     192.168.247.1:26599 - "GET /get_products?ec_id=2990 HTTP/1.1" 200 OK
fastapi-1  | 2024-07-08 07:02:28,582 INFO sqlalchemy.engine.Engine ROLLBACK
```

這邊就可以看到已經沒有錯誤了，只是目前資料庫裡面沒有答案，因此印出來是空的！


### 修改 docker-compose.yml 的 volume 名稱

由於剛剛的建置方法，建置出來的 `volume 名稱`，會是一連串的亂碼，這樣會很難分辨現在使用的 `volume` 是哪一個，因此現在我們來修改一下名稱：

* 新增 `fasiapi-mysql:/var/lib/mysql`，左邊是 volume 名稱，右邊是 mysql 在容器內的位置
* 在文件的末尾聲明現在使用的 volume 

```yml
version: '3.8'

services:
  db:
    image: mysql:latest
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: test
      MYSQL_USER: newuser
      MYSQL_PASSWORD: newpassword
    ports:
      - "3306:3306"
    volumes:                                    # 新增這行
      - fasiapi-mysql:/var/lib/mysql            # 新增這行

  fastapi:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      - MYSQL_HOST=db
      - MYSQL_USER=newuser
      - MYSQL_PASSWORD=newpassword
      - MYSQL_PORT=3306
      - MYSQL_DB=test

volumes:                                        # 新增這行
  fasiapi-mysql:                                # 新增這行
```


* 這樣修改好後，再一次重新建立 `image, container, volume`：

```shell
$ docker-compose up --build 
```

* 建立好後，查看現在正在運行的 volume

```shell
$ docker volume ls

------
DRIVER    VOLUME NAME
local     fastapi_docker_fasiapi-mysql
```

可以發現我們的 volume 名稱不是亂碼了！！！


## 建立 docker 時匯入資料

但是這樣寫的話，我們一開始是沒有資料的，因此我們會需要啟用 docker 時，順便塞一些測試資料到 docker 裡面，因此我們來修改一些檔案：

* 目前的檔案架構 - 

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
  |  docker-compose.yml         => 新增這個資料夾
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



## 檔案丟上 GCP

到目前為止我們已經可以用 `docker` 打包所有資料，並且也可以正常打開 `port` 和 `建置基本資料庫`，因此現在我們來把資料丟上雲端運作

## 把本地 mysql 資料轉移到GCP SQL

### 第一步驟 - 建立 cloudSQL 的執行個體

參考文件： https://penueling.com/%E7%B7%9A%E4%B8%8A%E5%AD%B8%E7%BF%92/%E6%8A%8A%E6%9C%AC%E5%9C%B0-mysql-%E8%BD%89%E7%A7%BB%E5%88%B0gcp-sql/

* 上方的文件主要在做以下幾件事情

1. 進到 GCP -> sql 
2. `建立執行個體`，選擇 Mysql
3. 設定 cloudSQL 的 `ID` 和 `密碼`
4. 建立好執行個體後，就可以找到公開的 `IP`，這個 IP 就可以讓大家都連進該資料庫
5. 到連線設定 -> 公開ip -> 已授權網路，設定自己目前的 ip，這樣才可以讓你本機電腦連上去

:::tip 設定連線 ip
第五點非常重要，因為如果你不設定這個 ip，你會無法連進資料庫  
可以使用這個網址: https://www.whatismyip.com.tw/tw/  
查完後直接把該IP填進去，這樣才可以在該網路環境連進資料庫   
:::


### 第二步驟 - 匯出本地端資料庫 -> dump 檔案

建立好後，我們先來把目前儲存在本地端的資料庫，匯出成 `dump 檔案`

1. 首先打開終端機，並確定 `mysql cli` 是否有打開

```shell
$ brew services list            # 確定目前 mysql 是否有打開
$ brew services start mysql     # 打開 mysql cli
```


2. 進入指定的資料庫(進入本地資料庫)

```shell
$ mycli -u root -h localhost
```

3. 判斷指定的使用者，是否有權利輸出 dump 資料

* `SHOW GRANTS FOR '使用者名稱'@'localhost'`

```shell
$ MySQL root@localhost:(none)> SHOW GRANTS FOR 'newuser'@'localhost';

------
+-----------------------------------------------------------+
| Grants for newuser@localhost                              |
+-----------------------------------------------------------+
| GRANT USAGE ON *.* TO newuser@localhost                   |
| GRANT ALL PRIVILEGES ON test.* TO newuser@localhost       |
+-----------------------------------------------------------+
```

以上方的資料來看，目前這個 `使用者(newuser)`，只有使用這個資料庫的權限，並沒有 `PROCESS` 的權限，因此我們現在先來增加權限ㄋ


4. 新增使用者權限

```shell
$ MySQL root@localhost:(none)> GRANT PROCESS ON *.* TO 'newuser'@'localhost';
                             > FLUSH PRIVILEGES;                             
```

新增好後，我們再來確認一次使用者的權限修改是否有成功：

```shell
$ MySQL root@localhost:(none)> SHOW GRANTS FOR 'newuser'@'localhost';

------
+-----------------------------------------------------------+
| Grants for newuser@localhost                              |
+-----------------------------------------------------------+
| GRANT PROCESS ON *.* TO `newuser`@`localhost`             |
| GRANT ALL PRIVILEGES ON `test`.* TO `newuser`@`localhost` |
+-----------------------------------------------------------+
```

這邊就可以看到我們的權限已經變更成 `GRANT PROCESS` 了～代表我們已經可以匯出 `dump` 檔案


5. 匯出 dump 檔案

此時我們先來退出 `cli`，然後再執行匯出指令

*  mysqldump -u 'username' -p 'your_database_name' > backup.sql

```shell
$ MySQL root@localhost:(none)> exit
$ mysqldump -u newuser -p test > backup.sql

Enter password: # 這邊要輸入 `使用者的密碼`
```

6. 確認得到的 dump 檔案

* 先確認自己是在最上層
```shell
$ pwd

----
/Users/yee0526
```

* 印出所有檔案
可以看到我們剛剛匯出的檔案 `backup.sql`

```shell
$ ls

------
Applications        Library             Pictures            backup.sql          dumps
# ...其他省略
```


### 第三步驟 - dump 檔案 import 進 storage 和 cloudSQL

1. 創建 GCS Bucket

* `gsutil mb gs://my-backups`

```shell
$ gsutil mb gs://fastapi-sql
```

2. 上傳 dump 檔案到 GCS

* `gsutil cp backup.sql gs://my-backups/backup.sql`

```shell
$ gsutil cp backup.sql gs://fastapi-sql/backup.sql
```

ps. 要把  `backup.sql`  推上 GCS，記得要把先把該檔案放到同檔案的位置那邊，要不然會因為指令找不到該檔案，導致發生錯誤

3. 用指令新增 database

* `gcloud sql databases create 'database-name' --instance='instance-name'`

```shell
$ gcloud sql databases create test --instance=product
```



4. 將 SQL 檔案導入 cloudSQL
* `gcloud sql import sql my-instance gs://my-backups/backup.sql --database=my-database`

ps. 注意事項： `my-instance` 是你在 sql 那邊已經建立好的實體，而 `--database` 要放的參數則是你 database 的名字，如果今天 cloudSQL 裡面沒有這個 database 會無法匯入，因此可以手動加入或是用指令加入這個 database

```shell
$ gcloud sql import sql product gs://fastapi-sql/backup.sql --database=test
```

但是記得通常會遇到權限問題！！！但是目前我還找不到解決權限問題的方法，目前我輸入以下指令

```shell
$ gcloud sql import sql product gs://fastapi-sql/backup.sql --database=test

------
ERROR: (gcloud.sql.import.sql) HTTPError 403: The service account does not have the required permissions for the bucket.
```

會噴出權限不足的錯誤，但是我明明已經到 `iam` 那邊打開 `Cloud SQL 管理員` 和 `儲存空間物件管理員 (storage object admin)` 的權限，還是無法，之後再研究

ps. 後來和同事討論過後，應該是因為 `backup.sql` 這個檔案內部，沒有特別指出 `database = test`，也就是沒有指出該 `database的名稱`，因此直接改用進入 `GCP 的介面`，手動匯入資料就可以成功






## 連接 Cloud SQL

前面我們都已經把 database 全部都丟上雲端之後，現在嘗試在本地端連接 cloudSQL，進而在本地直接操作雲端資料庫

### 第一步驟 - 安裝 cloud-sql-proxy

* 參考連結: https://cloud.google.com/sql/docs/mysql/connect-auth-proxy#mac-m1
* 直接進 GCP 的文件下載該 package

```shell
$ curl -o cloud-sql-proxy https://storage.googleapis.com/cloud-sql-connectors/cloud-sql-proxy/v2.11.4/cloud-sql-proxy.darwin.arm64
$ chmod +x cloud-sql-proxy
```
ps1. 在這裡，`chmod` 是改變文件模式的命令，`+x` 表示添加執行權限，`cloud-sql-proxy` 是文件名。
ps2. 安裝好後，可以直接在終端機輸入 `ls`，應該可以在目錄層級的地方，找到 `cloud-sql-proxy`


### 第二步驟 - 建立 Service Account 並產生 key

* 參考連結: https://kejyuntw.gitbooks.io/google-cloud-platform-learning-notes/content/google-cloud-sql/proxy/google-cloud-sql-proxy-README.html
* 上面的連結有詳細的圖文解釋，如何建立帳號，並產生 key，這個 key 會自動存到你的電腦裡，可以到 `download` 資料夾裡面找到


### 第三步驟 - 透過 Cloud SQL Proxy 連線到資料庫

先準備以下三種資料：

1. key 的位置: `/Users/yee0526/Downloads/fastapi-428908-58a60a94c892.json`
2. Cloud SQL instance 連線名稱: `<專案名稱>:<區域名稱>:<Cloud SQL Instance 名稱>`，這個可以直接到 cloudSQL 找到 - `fastapi-428908:asia-east1:product`
3. 確認目前 3306 這個 port 是沒有人佔用的


#### 1. key 位置

這個很容易，就是要確認一下剛剛載下來 key 的絕對位置在哪


#### 2. Cloud SQL instance 名稱

這個也很容易，只要進到你帳號裡面的 `cloudSQL`，就可以找到你的連線名稱 - `fastapi-428908:asia-east1:product`


#### 3. 3306 port

這個要先執行以下的命令：
```shell
$ sudo lsof -i -P -n | grep LISTEN

------
mysqld     7040        test12345   33u  IPv4 0x517389216ek1uo26ae75e      0t0    TCP 127.0.0.1:3306 (LISTEN)
......忽略
```

如果有這種的 `port` 代表目前 `3306` 是有人在使用的，而我研究後發現，是因為目前我的 `mysql client` 還開著，所以才導致被佔用中：

```shell
$ brew services list 

------
mysql   started yee0526 ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
```

只要把這個關掉，就可以了：
```shell
$ brew services stop mysql 
```

### 輸入以指令連線 cloudSQL

* 先暫時設定變數 `GOOGLE_APPLICATION_CREDENTIALS` 為 key 的位置
* 執行以下指令：`./cloud-sql-proxy {instance name}`

```shell
$ export GOOGLE_APPLICATION_CREDENTIALS="/Users/yee0526/Downloads/fastapi-428908-58a60a94c892.json"
$ ./cloud-sql-proxy fastapi-428908:asia-east1:product

------
2024/07/12 15:29:09 Authorizing with Application Default Credentials
2024/07/12 15:29:10 [fastapi-428908:asia-east1:product] Listening on 127.0.0.1:3306
2024/07/12 15:29:10 The proxy has started successfully and is ready for new connections!
```

出現以下指令後，就成功連結了 `cloudSQL`!!



### 第四步驟 － 連接本地 mysql
前面已經本地端連上 `cloudSQL` 了，最後來連接本地的 `mysql`，這樣就可以本地直接操作 `cloudSQL` 的資料

* 參考連結: https://blog.cloud-ace.tw/database/cloud-sqlpart2-cloud-sql-proxy/#Cloud_SQL_Proxy_%E9%80%A3%E7%B7%9A%E6%AD%A5%E9%A9%9F%E4%B8%80%EF%BC%9A%E5%95%9F%E7%94%A8_Cloud_SQL_Admin_API

#### 進入 mysql

* 記住自己 cloudSQL 的使用者名稱、密碼 - `mysql -h 127.0.0.1 -P 3306 -u {使用者名稱} -p`
```shell
$ mysql -h 127.0.0.1 -P 3306 -u user -p

----
Enter password: 輸入使用者密碼
```

#### 測試 cloudSQL 的資料

* 查看 database

```shell
$ mysql> show databases;

------
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test               |
+--------------------+
```

* 進入 database

```shell
$ use test;

------
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
```

* 查看 table

```shell
$ mysql> show tables;

------
+-----------------+
| Tables_in_test  |
+-----------------+
| alembic_version |
| product         |
| website         |
+-----------------+
```



## 丟上 GAE 運作

cloudSQL 已經準備好了，現在可以來把本機的檔案 `deploy` 上去，不過在 `deploy` 之前，要先來改一下些檔案：


* 目前的檔案架構

```md
| FASTAPI_Docker
  |  -- DBoperator         
        | -- __init__.py        
        | -- app.py        
        | -- database.py   
        | -- models.py     
        | -- base.py
  |  Dockerfile
  |  pyproject.toml  
  |  main.py
  |  docker-compose.yml
```

### 第一步驟 - 新增 `requirements.txt`、`app.yaml` 兩個檔案

1. `requirements.txt`: 應該會很好奇我們明明不是已經有 `pyproject.toml` 這個檔案來安裝 `package`，不過因為 GAE 只吃 `requirements`，不吃 `poetry` 的設定檔案，因此要改成 `requirements`
2. `app.yaml`：這個檔案是 `GAE deploy` 要的設定檔，部署時會先執行裡面的指令

```md
| FASTAPI_Docker
  |  -- DBoperator         
        | -- __init__.py        
        | -- app.py        
        | -- database.py   
        | -- models.py     
        | -- base.py
  |  Dockerfile
  |  pyproject.toml  
  |  main.py
  |  docker-compose.yml
  |  requirements.txt            => 安裝 python package
  |  app.yaml                    => 部署到 GAE 要的檔案
```

#### requirements.txt

這邊就直接把在 `pyproject` 有安裝的 `package` 放上來就好了
```txt
fastapi==0.104.1
SQLAlchemy==2.0.30
alembic==1.13.1
pydantic==2.5.2
PyMySQL==1.1.0
uvicorn==0.23.2
```

#### app.yaml 

這邊介紹一下這些參數都是幹嘛用的：

1. runtime: python39: 使用的環境是 python 3.9 版本
2. entrypoint ~~: 先安裝 `requirements` 裡面的 package，接著啟動指定資料夾的 host
3. env_variables: 設定連接資料庫的環境變數

ps. 特別提一下 `MYSQL_HOST`，會是這個指令是因為要連接 `cloudSQL`，如果今天要連接 `cloudSQL`，格式為 - `MYSQL_HOST=/cloudsql/<YOUR-PROJECT-ID>:<REGION>:<INSTANCE-NAME>`

```yaml
runtime: python39
entrypoint: pip install -r requirements.txt && uvicorn DBoperator.app:app --host 0.0.0.0 --port $PORT

env_variables:
  MYSQL_HOST: "/cloudsql/fastapi-429704:asia-east1:fastapi-data"
  MYSQL_USER: "root"
  MYSQL_PASSWORD: "00000"
  MYSQL_DB: "test"
```



### 第二步驟 - 修改 database.py

* 把連結本地資料庫的參數改成連結 cloudSQL 的參數，由於我們在 `app.yaml` 檔案設定了這個環境變數 `MYSQL_HOST: /cloudsql/fastapi-429704:asia-east1:fastapi-data`，因此如果今天是要連到 `cloudSQL`，就會進入 `cloudSQL` 的連結路徑，反之就會進入本地端資料庫的路徑

```py
# ...省略
import os

MYSQL_HOST = os.getenv('MYSQL_HOST', 'localhost')
MYSQL_USER = os.getenv('MYSQL_USER', 'root')
MYSQL_PASSWORD = os.getenv('MYSQL_PASSWORD', '00000')
MYSQL_DB = os.getenv('MYSQL_DB', 'test')

if MYSQL_HOST.startswith('/cloudsql/'):
    # Running on App Engine
    SQLALCHEMY_DATABASE_URL = f"mysql+pymysql://{MYSQL_USER}:{MYSQL_PASSWORD}@/{MYSQL_DB}?unix_socket={MYSQL_HOST}"
else:
    # Local development
    MYSQL_PORT = os.getenv('MYSQL_PORT', '3306')
    SQLALCHEMY_DATABASE_URL = f"mysql+pymysql://{MYSQL_USER}:{MYSQL_PASSWORD}@{MYSQL_HOST}:{MYSQL_PORT}/{MYSQL_DB}"


# ...省略
```

### 第三步驟 - gcloud app deploy

都修改好後，就可以來把檔案 push 上雲端了，執行以下指令，途中會出現是否要你繼續 `deploy` 的指令，只要輸入 `Y` 就可以繼續部署了

```shell
$ gcloud app deploy

------
Services to deploy:

descriptor:                  [/Users/yee0526/Desktop/python/fastAPI_Docker/app.yaml]
source:                      [/Users/yee0526/Desktop/python/fastAPI_Docker]
target project:              [fastapi-429704]
target service:              [default]
target version:              [20240719t124352]
target url:                  [https://fastapi-429704.de.r.appspot.com]
target service account:      [fastapi-429704@appspot.gserviceaccount.com]

Do you want to continue (Y/n)?  Y
```


### 第四步驟 - gcloud app browse、gcloud app logs tail -s default

部署完畢後，如果沒有錯誤，就可以來執行這兩個指令，來看一下部署是否成功~    

* gcloud app browse: 打開部署完成的網站
```shell
$ gcloud app browse

------
Opening [https://fastapi-429704.de.r.appspot.com] in a new tab in your default browser.
```

* gcloud app logs tail -s default: 查看部署網站的 log，如果今天部署失敗，可以用這個指令看是哪裡出錯了

```shell
$ gcloud app logs tail -s default

------
Waiting for new log entries...
```


到這邊為止就成功部署完成拉！！！！！！


:::tip GAE Dockerfile
這邊要特別提下，由於 `GAE` 在標準環境下，不直接使用 `Dockefile`，而是使用剛剛我們建置的 `app.yaml` 來建置容器   
並把建置好後的環境丟入 `Artifact Registry` 裡面，因此可以到 `Artifact Registry` 找到一些檔案   
:::

## 本地端開發 - 連接 cloudSQL

不過今天如果想要在本地端連結到 cloudSQL 要怎麼做呢？我們來修改 `docker-compose`：

```yml
# docker-compose.yml
services:
  fastapi:
    build: .
    ports:
      - "8000:8000"
    environment:
      - MYSQL_HOST=35.201.232.244
      - MYSQL_USER=root
      - MYSQL_PASSWORD=00000
      - MYSQL_PORT=3306
      - MYSQL_DB=test
```


這樣修改完後，輸入 `docker-compose up --build`，就可以直接在本地連進 `cloudSQL資料庫`：
```shell
$ docker-compose up --build
```




## 雲端、本機資料庫連接設定 - version 1

由於前面的設定方式，在連接本地的資料庫和雲端的資料庫差得很多，因此我們要來修改個方式，讓我們改一個變數就可以選擇要連結 - `雲端` 還是 `本地`。  

:::tip 資料庫判斷讀取流程
1. GAE -> app.yaml 設定的變數 -> database.py 的判斷式 -> 進入 `MYSQL_HOST.startswith('/cloudsql/')`
2. local -> docker-compose 設定的變數 -> database.py 的判斷式 -> 偵測現在的環境變數是 `cloudSQL` 還是 `本地端資料庫`
:::


### 目前的資料庫連接設定


#### database 判斷連接哪個資料庫

* database.py

先前設定的 `database.py` 檔案，這邊會判斷要讀取雲端還是本地的資料庫，如果今天是在 `GAE` 上，並且要讀取 `cloudSQL`，我們在 `app.yaml` 那邊有設定環境變數，因此會由 `MYSQL_HOST` 判斷現在要連 `雲端資料庫`，反之如果今天連到的是 `本地資料庫`，就會吃原本寫在 `database.py` 的環境變數


```py
# DBoperator/database.py

MYSQL_HOST = os.getenv('MYSQL_HOST', 'localhost')
MYSQL_USER = os.getenv('MYSQL_USER', 'root')
MYSQL_PASSWORD = os.getenv('MYSQL_PASSWORD', '00000')
MYSQL_DB = os.getenv('MYSQL_DB', 'test')


if MYSQL_HOST.startswith('/cloudsql/'):
    # Running on App Engine
    SQLALCHEMY_DATABASE_URL = f"mysql+pymysql://{MYSQL_USER}:{MYSQL_PASSWORD}@/{MYSQL_DB}?unix_socket={MYSQL_HOST}"
else:
    # Local development
    MYSQL_PORT = os.getenv('MYSQL_PORT', '3306')
    SQLALCHEMY_DATABASE_URL = f"mysql+pymysql://{MYSQL_USER}:{MYSQL_PASSWORD}@{MYSQL_HOST}:{MYSQL_PORT}/{MYSQL_DB}"

# 省略
```




#### local 連接本地資料庫

* docker-compose.py

```yaml
services:
  db:
    image: mysql:latest
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: test
      MYSQL_USER: newuser
      MYSQL_PASSWORD: newpassword
    ports:
      - "3306:3306"
    volumes:
      - fasiapi-mysql:/var/lib/mysql

  fastapi:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      - MYSQL_HOST=db
      - MYSQL_USER=newuser
      - MYSQL_PASSWORD=newpassword
      - MYSQL_PORT=3306
      - MYSQL_DB=test

volumes:
  fasiapi-mysql:
```



#### local 連接 cloudSQL 資料庫

* docker-compose.py

```yaml

# cloudSQL
services:
  fastapi:
    build: .
    ports:
      - "8000:8000"
    environment:
      - MYSQL_HOST=35.201.232.244
      - MYSQL_USER=root
      - MYSQL_PASSWORD=00000
      - MYSQL_PORT=3306
      - MYSQL_DB=test
```


### 重構 local 連接 - 本地, cloudSQL 資料庫

由於前面那樣寫，當今天要切換連接本地或是雲端資料庫會非常麻煩，因此我們現在來改成只要修改一個變數名稱，就可以切換要連接的資料庫


#### .env 環境變數檔案

* 新增一個 .env 檔案
```md
| FASTAPI_Docker
  |  -- DBoperator         
        | -- __init__.py        
        | -- app.py        
        | -- database.py   
        | -- models.py     
        | -- base.py
  |  Dockerfile
  |  pyproject.toml  
  |  main.py
  |  docker-compose.yml
  |  requirements.txt
  |  app.yaml
  |  .env                       => 新增一個環境變數檔案
```

* .env 檔案內容 - 只要切換 `DB_MODE` 就可以選擇要連接哪一個資料庫

```md
# 設置為 'local' 或 'cloud'
DB_MODE=local
# DB_MODE=cloud

# Local DB settings
LOCAL_MYSQL_HOST=db
LOCAL_MYSQL_USER=newuser
LOCAL_MYSQL_PASSWORD=newpassword
LOCAL_MYSQL_PORT=3306
LOCAL_MYSQL_DB=test

# Cloud SQL settings
CLOUD_MYSQL_HOST=35.201.232.244
CLOUD_MYSQL_USER=root
CLOUD_MYSQL_PASSWORD=00000
CLOUD_MYSQL_PORT=3306
CLOUD_MYSQL_DB=test
```


#### docker-compose.yml 檔案修改

* 這邊要修改成讓 fastapi 除了資料庫讀取 `db`，也要吃進 `DB_MODE` 變數，這樣我們等等可以用 `DB_MODE` 的值影響要連進哪一個資料庫
* 還要吃進剛剛的 `.env` 檔案，裡面有等等會使用到的環境變數
```yaml
version: '3.8'

services:
  db:
    image: mysql:latest
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: test
      MYSQL_USER: newuser
      MYSQL_PASSWORD: newpassword
    ports:
      - "3306:3306"
    volumes:
      - fastapi-mysql:/var/lib/mysql

  fastapi:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      - DB_MODE=${DB_MODE}
    env_file:
      - .env

volumes:
  fastapi-mysql:
```


#### database.py 檔案修改

* 這邊要修改成，判斷現在的 `DB_MODE` 變數值是啥，來判斷要連進 `cloud` 或 `local`

```py
# 省略

DB_MODE = os.getenv('DB_MODE', 'local')

if DB_MODE == 'local':
    MYSQL_HOST = os.getenv('LOCAL_MYSQL_HOST', 'localhost')
    MYSQL_USER = os.getenv('LOCAL_MYSQL_USER', 'root')
    MYSQL_PASSWORD = os.getenv('LOCAL_MYSQL_PASSWORD', '00000')
    MYSQL_DB = os.getenv('LOCAL_MYSQL_DB', 'test')
    MYSQL_PORT = os.getenv('LOCAL_MYSQL_PORT', '3306')
else:
    MYSQL_HOST = os.getenv('CLOUD_MYSQL_HOST', 'localhost')
    MYSQL_USER = os.getenv('CLOUD_MYSQL_USER', 'root')
    MYSQL_PASSWORD = os.getenv('CLOUD_MYSQL_PASSWORD', '00000')
    MYSQL_DB = os.getenv('CLOUD_MYSQL_DB', 'test')
    MYSQL_PORT = os.getenv('CLOUD_MYSQL_PORT', '3306')

if MYSQL_HOST.startswith('/cloudsql/'):

    print("cloud ================")
    # Running on App Engine
    SQLALCHEMY_DATABASE_URL = f"mysql+pymysql://{MYSQL_USER}:{MYSQL_PASSWORD}@/{MYSQL_DB}?unix_socket={MYSQL_HOST}"
else:

    print(f"LOCAL ====== connect {DB_MODE} ==========")
    # Local development
    MYSQL_PORT = os.getenv('MYSQL_PORT', '3306')
    SQLALCHEMY_DATABASE_URL = f"mysql+pymysql://{MYSQL_USER}:{MYSQL_PASSWORD}@{MYSQL_HOST}:{MYSQL_PORT}/{MYSQL_DB}"


# 省略
```




### 修改完後切換連接 local or cloudSQL 資料庫的方式

#### 本地
```shell
$ export DB_MODE=local
$ docker-compose up --build
```

#### cloudSQL
```shell
$ export DB_MODE=cloud
$ docker-compose up --build
```

:::tip
用 `DB_MODE` 變數切換就可以選擇要連接到哪個資料庫了
:::



## 雲端、本機資料庫連接設定 - version 2


### .env

* 這邊跟前面基本一樣，不過改一下變數名稱 - `USE_CLOUD_SQL`: 用它來控制現在要連接雲端還是本地資料庫，如果要連接雲端輸入 `true`，反之 `false`

```md
USE_CLOUD_SQL=true
CLOUD_SQL_HOST=35.201.232.244
CLOUD_SQL_USER=root
CLOUD_SQL_PASSWORD=00000
CLOUD_SQL_DB=test

LOCAL_DB_HOST=db
LOCAL_DB_USER=newuser
LOCAL_DB_PASSWORD=newpassword
LOCAL_DB_NAME=test
```



### database.py 修改

* 變數 `GAE_APPLICATION`: 由於這個變數要 deploy 上 GAE 的時候，GAE 會自動把變數設定為 GAE 項目的 ID，因此可以用這個變數來當作現在是不是 GAE 的環境
* 變數 `USE_CLOUD_SQL`: 此變數我們會設定在 .env 檔案中，等等用這個變數來判斷我們在本地端時，要連接的是本地端資料庫還是雲端資料庫
* 這一段做的就是: 先判斷我們現在是本地端還是 GAE 端，再來判斷如果是本地端，我們要連接的本地端的資料庫，還是雲端資料庫

```py
# DBoperator/database.py

# 省略
is_gae = os.getenv('GAE_APPLICATION', None)

if is_gae:
    MYSQL_HOST = os.getenv('MYSQL_HOST')
    MYSQL_USER = os.getenv('MYSQL_USER')
    MYSQL_PASSWORD = os.getenv('MYSQL_PASSWORD')
    MYSQL_DB = os.getenv('MYSQL_DB')
    
    if MYSQL_HOST.startswith('/cloudsql/'):
        SQLALCHEMY_DATABASE_URL = f"mysql+pymysql://{MYSQL_USER}:{MYSQL_PASSWORD}@/{MYSQL_DB}?unix_socket={MYSQL_HOST}"  
else:
    USE_CLOUD_SQL = os.getenv('USE_CLOUD_SQL', 'false').lower() == 'true'

    if USE_CLOUD_SQL:
        MYSQL_HOST = os.getenv('CLOUD_SQL_HOST')
        MYSQL_USER = os.getenv('CLOUD_SQL_USER')
        MYSQL_PASSWORD = os.getenv('CLOUD_SQL_PASSWORD')
        MYSQL_DB = os.getenv('CLOUD_SQL_DB')
    else:
        MYSQL_HOST = os.getenv('LOCAL_DB_HOST', 'db')
        MYSQL_USER = os.getenv('LOCAL_DB_USER', 'newuser')
        MYSQL_PASSWORD = os.getenv('LOCAL_DB_PASSWORD', 'newpassword')
        MYSQL_DB = os.getenv('LOCAL_DB_NAME', 'test')

    MYSQL_PORT = os.getenv('MYSQL_PORT', '3306')
    SQLALCHEMY_DATABASE_URL = f"mysql+pymysql://{MYSQL_USER}:{MYSQL_PASSWORD}@{MYSQL_HOST}:{MYSQL_PORT}/{MYSQL_DB}"            


primary_engine = create_engine(SQLALCHEMY_DATABASE_URL, echo = True)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=primary_engine)

# 省略
```


### docker-compose.yml

* 這邊跟之前相比，我們全部都把環境變數加上去，這樣未來如果要換資料庫，或者是資安問題避免別人看到關鍵變數，都會比較方便

```yml
version: '3.8'

services:
  db:
    image: mysql:latest
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: ${LOCAL_DB_NAME}
      MYSQL_USER: ${LOCAL_DB_USER}
      MYSQL_PASSWORD: ${LOCAL_DB_PASSWORD}
    ports:
      - "3306:3306"
    volumes:
      - fastapi-mysql:/var/lib/mysql

  fastapi:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      - USE_CLOUD_SQL=${USE_CLOUD_SQL}
      - CLOUD_SQL_HOST=${CLOUD_SQL_HOST}
      - CLOUD_SQL_USER=${CLOUD_SQL_USER}
      - CLOUD_SQL_PASSWORD=${CLOUD_SQL_PASSWORD}
      - CLOUD_SQL_DB=${CLOUD_SQL_DB}
      - LOCAL_DB_HOST=${LOCAL_DB_HOST}
      - LOCAL_DB_USER=${LOCAL_DB_USER}
      - LOCAL_DB_PASSWORD=${LOCAL_DB_PASSWORD}
      - LOCAL_DB_NAME=${LOCAL_DB_NAME}

volumes:
  fastapi-mysql:
```

這樣修改完後，以後只要修改 `.env檔案` 裡面的環境變數 `USE_CLOUD_SQL`，就可以切換現在到底要連接的是本地資料庫還是雲端資料庫。  
    


