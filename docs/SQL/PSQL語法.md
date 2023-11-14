---
sidebar_position: 1
---

## PSQL基礎語法

前情提要，在終端機打SQL指令，可以直接換行寫下一段程式碼，他判斷一段語法結束是看 `;分號`，所以你可以一直換行
最後用分號結尾就可以

### 開始指令

進到PSQL裡面 - 直接打 `psql`

### 查看目前所有的 database

```shell
test1234=# \l

------
                                                        List of databases
          Name           |  Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges   
-------------------------+----------+----------+-------------+-------------+------------+-----------------+-----------------------
 GitActions_development  | test1234  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | en-US      | icu             | 
 GitActions_test         | test1234  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | en-US      | icu             | 
```

### 新增 database

新增一個database 
```shell
test1234=# CREATE DATABASE good
```

我們這邊新增另外一個 database，等等來做比較
```shell
test1234=# CREATE DATABASE good1
```


### 選擇使用哪個 database

```shell
test1234=# \c good

------
You are now connected to database "good" as user "test1234".
```


### 新增 table

先到進到 `PSQL` 中某個 `database`，我們就在 good 這個 database 中新增

```shell
good=# CREATE TABLE users (
        id SERIAL PRIMARY KEY,
        username VARCHAR,
        age INTEGER,
        height INTEGER,
        weight INTEGER 
       );
```


這樣就可以新增一個 table 了！

### 確認目前 database 有哪些 table

```shell
good=# \z

------
                                  Access privileges
 Schema |     Name     |   Type   | Access privileges | Column privileges | Policies 
--------+--------------+----------+-------------------+-------------------+----------
 public | users        | table    |                   |                   | 
(1 rows)
```

### 新增一筆資料

```shell
good=# INSERT INTO users (username, age, height, weight)
VALUES ('Jimmy', 18, 180, 75);

------
INSERT 0 1
```

### 查看剛剛的輸入的資料

```shell
good=# SELECT * FROM users

------
 id | username | age | height | weight 
----+----------+-----+--------+--------
  1 | jimmy    |  18 |    180 |     75
(1 row)
```

### 批量輸入資料

如果今天想要一次輸入多筆資料，可以像這樣輸入

```shell
good=# INSERT INTO users (username, age, height, weight)
VALUES
  ('Alice', 25, 160, 55),
  ('Bob', 30, 175, 80),
  ('Eva', 22, 165, 62),
  ('Charlie', 28, 185, 90);

------
INSERT 0 4
```

再看一次這次輸入的結果：
```shell
good=# SELECT * FROM users;

------
 id | username | age | height | weight 
----+----------+-----+--------+--------
  1 | jimmy    |  18 |    180 |     75
  2 | Alice    |  25 |    160 |     55
  3 | Bob      |  30 |    175 |     80
  4 | Eva      |  22 |    165 |     62
  5 | Charlie  |  28 |    185 |     90
(5 rows)
```


### 刪除 table

我們先來看一下現在有哪些 table

```shell
good=# \z

------
                                  Access privileges
 Schema |     Name     |   Type   | Access privileges | Column privileges | Policies 
--------+--------------+----------+-------------------+-------------------+----------
 public | users        | table    |                   |                   | 
 public | users_id_seq | sequence |                   |                   | 
(2 rows)
```

我們來刪掉 users 這個 table

```shell
good=# DROP TABLE users;
```

再查看一下，確定被刪掉了

```shell
good=# \z

------
                            Access privileges
 Schema | Name | Type | Access privileges | Column privileges | Policies 
--------+------+------+-------------------+-------------------+----------
(0 rows)
```


### 刪掉 database

先確認目前有的 database

```shell
good=# \l

------
                                                        List of databases
          Name           |  Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges   
-------------------------+----------+----------+-------------+-------------+------------+-----------------+-----------------------
 GitActions_development  | yee0526  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | en-US      | icu             | 
 GitActions_test         | yee0526  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | en-US      | icu             | 
 good                    | yee0526  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | en-US      | icu             | 
 good1                   | yee0526  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | en-US      | icu             | 
```

刪掉 good1 的 database

```shell
good=# DROP DATABASE good1;
```

再次確認是否刪除

```shell
good=# \l

------
                                                        List of databases
          Name           |  Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges   
-------------------------+----------+----------+-------------+-------------+------------+-----------------+-----------------------
 GitActions_development  | yee0526  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | en-US      | icu             | 
 GitActions_test         | yee0526  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | en-US      | icu             | 
 good                    | yee0526  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | en-US      | icu             | 
```


### 離開 PSQL 環境

```shell
good=# \q
```




參考連結 : https://www.runoob.com/postgresql/postgresql-select-database.html