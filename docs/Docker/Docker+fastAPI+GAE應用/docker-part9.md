---
sidebar_position: 9
---


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