---
sidebar_position: 10
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
