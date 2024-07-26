---
sidebar_position: 5
---




Docker + fastAPI 應用 - part5
------

## 第四版的 dockerfile -> 要加上 docker-compose.yml

目前的整體檔案架構：
 
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

* db 服務 - 拉 mysql 的 image 檔案，並且創建時需要先設定 `MYSQL_ROOT_PASSWORD` 參數，不然無法創建，再來設定 `資料庫的使用者環境變數`: `MYSQL_DATABASE`、`MYSQL_USER`、`MYSQL_PASSWORD`
* fastapi 服務
 - build 參數: 指定 Docker 構建上下文為當前目錄。這表示在當前目錄下必須有一個 Dockerfile，用來構建 fastapi 服務的鏡像
 - depends_on 參數: 指定 fastapi 服務依賴於 db 服務。這意味著 fastapi 服務將在 db 服務啟動後再啟動
 - environment: 這邊的環境變數就要設定剛剛 db 已經設定好的參數，這樣才可以進入 `db` 資料庫中

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
      - MYSQL_DB=test
      - MYSQL_USER=newuser
      - MYSQL_PASSWORD=newpassword
      - MYSQL_PORT=3306
```

### 重新執行 docker build 指令
接下來我們就來輸入以下指令，讓 `image` 重新建立起來：

```shell
$ docker-compose up --build
```

這個指令會依照你的 `docker-compose.yml` 檔案，建立 `image`。     


### 進入 endpoint 點

最後我們再一次輸入這個 `endpoint: http://127.0.0.1:8000/get_products?ec_id=2990`，這時候會發現，剛剛的 `connect error` 已經不見了，但！！！這次會出現另外一個錯誤～像這樣：

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

由於剛剛的建置方法，建置出來的 `volume 名稱`，會是一連串的亂碼，這樣會很難分辨現在使用的 `volume` 是哪一個，因此現在我們 `來修改一下名稱 + 創建持久化儲存數據的 volume`：

* 新增 `fasiapi-mysql:/var/lib/mysql`，左邊是 volume 名稱，右邊是 mysql 在容器內的位置: `在 service 中定義 volume（如在 db 服務中的 volumes 配置）是指定該卷在容器內的具體掛載位置。`
* 在文件的末尾聲明現在使用的 volume: `在文件最底部定義 volume，是全局定義該 volume，使其成為 Docker Compose 文件的一部分，讓所有服務都可以引用和使用它。`

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

:::tip
兩者結合起來工作，確保卷被正確創建並掛載到指定的容器內目錄：   

在 db 服務中定義卷的掛載點，確保 MySQL 數據文件存儲在該卷中。  
在文件最底部定義卷，讓 Docker Compose 知道這個卷的存在並進行管理。   
因此，這兩部分的配置是互補的，確保卷被正確創建並用於持久化存儲數據。   
:::




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