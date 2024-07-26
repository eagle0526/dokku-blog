---
sidebar_position: 8
---

Docker + fastAPI 應用 - part8
------


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

