---
sidebar_position: 1
---

## mac 安裝 mysql-client
參考文章：https://myapollo.com.tw/blog/install-mysql-using-homebrew/

### brew 指令

假設大家先前都已經安裝好 `homebrew`，所以我們直接來安裝 mysql-client

1. 安裝 mysql
```shell
$ brew install mysql mycli
```

Ps. homebrew 是推薦使用 `brew install mysql`，但是我一開始只安裝這個，但是一直無效，後來改成 `brew install mysql mycli` 這個就成功了


2. 啟動 brew server

先輸入以下指令，看目前有哪些服務：
```shell
$ brew services list

----
Name    Status User File
mysql   none        
redis   none        
unbound none    
```

再來啟動 mysql

```shell
$ brew services start mysql
```

4. 最後輸入指令，連結本地 mysql
```shell
$ mycli -u root -h localhost

------
MySQL root@localhost:(none)>
```

這樣就可以開始輸入指令了


## MySQL基礎語法

### 查看目前所有的 database

```shell
MySQL root@localhost:(none)> show databases

------
+--------------------+
| Database           |
+--------------------+
| mysql              |
+--------------------+
```
ps. 指令輸入後， `terminal` 會進到另一個地方，按 q 就可離開

### 新增 database

新增一個 database，這個 database 叫做 `test`
```shell
MySQL root@localhost:(none)> create database test
```

我們這邊新增另外一個 database，等等來做比較
```shell
MySQL root@localhost:(none)> create database test2
```

新增完後，我們在列出 `database` 來看一下
```shell
$ MySQL root@localhost:(none)> show databases

------
+--------------------+
| Database           |
+--------------------+
| mysql              |
| test               |
| test2              |
+--------------------+
```


### 選擇使用哪個 database

我現在想要連到 test 這個 `database`

```shell
MySQL root@localhost:(none)> use test

------
You are now connected to database "test" as user "root"
Time: 0.006s
MySQL root@localhost:test>
```

ps. 成功連接後，`localhost` 後面就會從 `none` 轉變成你現在 `database` 的名稱

### 新增 table

進到指定的 database 後就可以創建 table，現在來創建一個 `name = MyClass` 的表單

```shell
MySQL root@localhost:test> create table MyClass(id int(4) not null primary key auto_increment, name char(20) not null);
Query OK, 0 rows affected
```

這樣就可以新增一個 table 了！


### 確認這個 table 的內詳細內容

```shell
$ MySQL root@localhost:test> describe `MyClass`

------
+-------+----------+------+-----+---------+----------------+
| Field | Type     | Null | Key | Default | Extra          |
+-------+----------+------+-----+---------+----------------+
| id    | int      | NO   | PRI | <null>  | auto_increment |
| name  | char(20) | NO   |     | <null>  |                |
+-------+----------+------+-----+---------+----------------+
```

### 確認目前 database 有哪些 table

```shell
MySQL root@localhost:test> show tables
1 row in set

------
+----------------+
| Tables_in_test |
+----------------+
| MyClass        |
+----------------+
```

### 新增單筆資料

```shell
MySQL root@localhost:test> insert into `MyClass` (id, name) values (4, 'david')

------
Query OK, 1 row affected
```

### 查看剛剛的輸入的資料

```shell
MySQL root@localhost:test> select * from `MyClass`
1 rows in set

------
+----+-------+
| id | name  |
+----+-------+
| 4  | david |
+----+-------+


```

### 批量輸入資料

如果今天想要一次輸入多筆資料，可以像這樣輸入

```shell
MySQL root@localhost:test> insert into MyClass values(1,'Tom'),(2,'Joan'), (3,'Wang');

------
Query OK, 3 rows affected
Time: 0.023s
```

再看一次這次輸入的結果：
```shell
MySQL root@localhost:test> select * from `MyClass`
3 rows in set

------
+----+------+
| id | name |
+----+------+
| 1  | Tom  |
| 2  | Joan |
| 3  | Wang |
+----+------+
```


### 刪除 table

我們先來看一下現在有哪些 table

```shell
MySQL root@localhost:test> show tables
1 row in set

------
+----------------+
| Tables_in_test |
+----------------+
| MyClass        |
+----------------+
```

我們來刪掉 users 這個 table

```shell
MySQL root@localhost:test> drop table `MyClass`
You're about to run a destructive command.
```

再查看一下，確定被刪掉了

```shell
MySQL root@localhost:test> show tables

------
+----------------+
| Tables_in_test |
+----------------+
+----------------+
```


### 刪掉 database

先確認目前有的 database

```shell
MySQL root@localhost:test> show databases

------
+--------------------+
| Database           |
+--------------------+
| mysql              |
| test               |
| test2              |
+--------------------+

```

刪掉 good1 的 database

```shell
MySQL root@localhost:test> drop database test2
You're about to run a destructive command.
Do you want to proceed? (y/n): y
Your call!
Query OK, 0 rows affected
Time: 0.028s
```

再次確認是否刪除

```shell
MySQL root@localhost:test> show databases

------
+--------------------+
| Database           |
+--------------------+
| mysql              |
| test               |
+--------------------+
```


### 離開 mySql 環境

```shell
MySQL root@localhost:test> exit
Goodbye!
```

### 停止 mysql

先檢查有再啟動的 brew 有哪些

```shell
$ brew services list 

------
Name    Status 
mysql   started
redis   none            
unbound none   
```

停止 mysql 服務
```shell
$ brew services stop mysql                                                                                            1 ↵  
------

Stopping `mysql`... (might take a while)
==> Successfully stopped `mysql` (label: homebrew.mxcl.mysql)
```



參考連結 : https://allaboutdataanalysis.medium.com/mysql%E5%B8%B8%E7%94%A8%E6%8C%87%E4%BB%A4%E7%A2%BC-0cf63b71d6c3




### 遠端 MySql 連接

參考連結 : https://handle.idv.tw/mysql%E6%8C%87%E4%BB%A4%E5%A2%9E%E5%8A%A0%E5%B8%B3%E8%99%9F/

1. 先登入MySQL
```shell
$ mysql -u root -p
```
2. 然後為新帳號建立資料庫, 以下會以 “newdatabase” 為例子:
```shell
$ mysql>  CREATE DATABASE 'newdatabase';
Query OK, 1 row affected (0.00 sec)
```
3. 建立 “newuser” 帳號, 密碼是 “newpassword”, 主機是 “localhost”:
```shell
$ mysql> CREATE USER 'newuser'@'localhost' IDENTIFIED BY 'newpassword';
Query OK, 0 rows affected (0.00 sec)
```

若要不指定存取來源，可以把 localhost 設定為 ％，類似這樣。

```shell
$ mysql> CREATE USER 'newuser'@'%' IDENTIFIED BY 'newpassword';
Query OK, 0 rows affected (0.00 sec)

```
4. 給予新帳號 “newuser” 權限讀寫新資料庫 “newdatabase”

```shell
$ mysql> GRANT ALL PRIVILEGES ON newdatabase.* TO 'newuser'@'localhost';
Query OK, 0 rows affected (0.00 sec)
```
5. 更新權限狀態
```shell
$ mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)
```


如果今天 `local` 端不知道自己的 user 名稱或是 password 是什麼，可以參考上面的連結，創造一個新的使用者，這樣你在本地端連接資料庫的時候，就可以用項以下的連結控制資料庫


```md
mysql+pymysql://<username>:<password>@<host>:<port>/<database_name>
mysql+pymysql://newuser:newpassword@localhost:3306/test
```


Ps. MySql 預設的 port 是 3306