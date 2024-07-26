---
sidebar_position: 9
---


## 本地端開發 - 連接 cloudSQL

我們原本 `docker` 的設定是連接到本地端的資料庫，不過今天如果想要在本地端連結到 cloudSQL 要怎麼做呢？我們來修改 `docker-compose`：

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