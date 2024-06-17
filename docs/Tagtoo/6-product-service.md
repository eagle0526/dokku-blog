---
title: tagtoo/product-service
sidebar_label: "6. [tagtoo] product-service-server"
description: product-service server 是關於 api 和 dashboard-api 結合的專案
last_update:
  date: 2024-06-17
keywords:
  - product-service  
tags:
  - product-service  
sidebar_position: 6
---


## 本機開發 - 流程大綱

1. git clone https://github.com/Tagtoo/product-service.git
2. 修改 `docker-compose.yml` 中 `services → supplier-api → environment → DB_OPERATOR_HOST`，直接改成 `DB_OPERATOR_HOST: db-feed.tagtoo.com.tw` 才會有商品資訊
3. 在 `services/supplier/libs/feed/feed/ecs/` 創建 `_107` 資料夾
4. 跑 `docker-compose up supplier-api` 指令
5. `vscode` 開另外個 `terminal` 進入 docker 容器裡面 - `docker exec -it product-service-supplier-api-1 /bin/bash`
6. 安裝 `pdm package` - `pip install pdm`
7. 更新套件的版本依賴 - `pdm install`
8. 本地開發 - `http://localhost:8002/feed?ecid=2990&publisher=fb`
9. 正式上線版 - `https://supplier-feed.tagtoo.com.tw/feed?ecid=107&publisher=facebook&currency=TWD&lang=zh&skip=0&limit=10000`


## 本機開發 - 詳細步驟

### 步驟一：先把 product-service 的 repo clone 下來
```shell
$ git clone ttps://github.com/Tagtoo/product-service.git
```

### 步驟二：修改 docker-compose.yml

這個步驟的目的，是要讓我們在本地端可以讀到資料庫，依照以下位置找到要修改的東西 -> `services → supplier-api → environment → DB_OPERATOR_HOST`

```docker
<!-- product-service/docker-compose.yml -->

<!-- ...省略 -->

    environment:
      # DB_OPERATOR_HOST: db-api:8000                   -> 這一行註解掉
      DB_OPERATOR_HOST: db-feed.tagtoo.com.tw           -> 這一行打開
```

### 步驟三：新增指定 ec 的資料夾

在 `services/supplier/libs/feed/feed/ecs/` 創建 `_107` 資料夾

Ps. 依照你現在是什麼 `ec`，就取名你的資料夾



### 步驟四：跑 docker-compose 指令

這個指令會新增 image 和 container 

```shell
$ docker-compose up supplier-api
```




### 步驟五：vscode 新增 terminal + 進入剛剛創建好的 container

這個指令的目的是，我們現在要先進到 docker 裡面，用 `pdm` 解決依賴項的關係，由於我們在第三步驟新增的 `資料夾`，跟 `pyproject.toml` 不同層級，因此我們要來更新他
```shell
$ docker exec -it product-service-supplier-api-1 /bin/bash
```



### 步驟六：安裝 pdm package


```shell
$ root@d1128268cc8c:/srv/supplier/api# pip install pdm
```



### 步驟七：更新依賴項

```shell
$ pdm install
```




### 步驟八：開始本地開發

這樣你就可以在本地開發了，輸入以下的網址，就可以吃到你剛剛新增資料夾裡面的內容

```md
`http://localhost:8002/feed?ecid=2990&publisher=fb`
```

* 重點一: 如果今天沒有做 `步驟五~步驟七`，你在輸入這一段網址時，你換在終端機看到 - 找不到該 `ec module`
* 重點二: 每改一次 `資料夾裡面的內容`，就要到 `docker` 裡面執行 `pdm install` 一次，所以建議開發時，把 `docker 容器` 開著，比較好開發



### 步驟九：實際上線網址
最後輸入以下網址，就可以看到究竟有沒有開發成功

```md
https://supplier-feed.tagtoo.com.tw/feed?ecid=107&publisher=facebook&currency=TWD&lang=zh&skip=0&limit=10000
```


還有以下幾個可以參考的 `feed` 網址:  

- 本地端的格式：http://localhost:8002/feed?ecid=2990&publisher=fb
- facebook 的格式（實際）：https://supplier-feed.tagtoo.com.tw/feed?ecid=107&publisher=fb&currency=TWD&lang=zh&skip=0&limit=10000
- GADS 的格式（實際）：https://supplier-feed.tagtoo.com.tw/feed?ecid=107&publisher=google_ads&currency=TWD&lang=zh&skip=0&limit=10000
- GMC 的格式（實際）：https://supplier-feed.tagtoo.com.tw/feed?ecid=107&publisher=gmc&currency=TWD&lang=zh&skip=0&limit=10000



### 參考連結：
1. https://www.notion.so/tagtoo-rd/77a99c7faadd42cfa362a73d6b495ba7











