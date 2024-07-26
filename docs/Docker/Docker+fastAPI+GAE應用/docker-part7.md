---
sidebar_position: 7
---

Docker + fastAPI 應用 - part7
------



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

