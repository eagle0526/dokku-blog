---
sidebar_position: 9
---

## 雲端、本機資料庫連接設定 - version 2

`version1` 我們還需要只用指令才能切換要連進哪個資料庫，但這樣還是有點麻煩，這個版本我們只要修改 `USE_CLOUD_SQL` 的值，就可以決定要使用哪個 `資料庫`!


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
    



### 修改完後切換連接 local or cloudSQL 資料庫的方式

#### 本地

* .env 檔案內容

```md
USE_CLOUD_SQL=false
# 省略
```

* 接著建立 container
```shell
$ docker-compose up --build
```

#### cloudSQL


```md
USE_CLOUD_SQL=true
# 省略
```
* 接著建立 container
```shell
$ docker-compose up --build
```

:::tip
用 `DB_MODE` 變數切換就可以選擇要連接到哪個資料庫了
:::