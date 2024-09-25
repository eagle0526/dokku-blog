---
title: tagtoo/contentAPI
sidebar_label: "15. [cron-job-v2] sync-product-feed"
description: sync-product-feed 要如何在本地端測試
last_update:
  date: 2024-09-02
keywords:
  - cron-job-v2
  - sync-product-feed
tags:
  - cron-job-v2
sidebar_position: 15
---



## sync-product-feed

這個在 cron-job-v2 的專案，主要目的是更新客戶資料庫的產品資料，以往我們通常會使用 muffet 去更新資料庫的產品，不過有些客戶不想用 muffet，想使用自己提供的 xml 檔案，讓我們去主動讀取那些檔案，更新進資料庫，這樣客戶就可以限制自己廣告可以出現哪些產品


## cron-job-v2/sync-product-feed 開啟本地端測試

sync-product-feed 本地測試，先在本地端起好 image 後，接著建立有掛載的 container，再來去修改sync-product-feed 裡面的 config 檔案，像我們這次只想要測試 caco-183，所以我們把除了 183 以外的檔案全部註解掉，就只留 183，最後在執行指令。


### 步驟一: 設定 shared-func

```shell
$ make cron_job_dir=$(pwd)/src/cronjobs/sync-product-feed/ move-shared-to-cron-job
```

### 步驟二: 建立 image

```shell
$ docker build -t sync-product-feed:latest -f src/cronjobs/sync-product-feed/Dockerfile .    
```

### 步驟三: 建立掛載的 container

```shell
$ docker run -it --name sync-product-feed-bind -v /Users/yee0526/Desktop/TAGTOO/cron-jobs-v2/正式上線用/cron-jobs-v2/src/cronjobs/sync-product-feed:/srv sync-product-feed:latest /bin/bash 
```

### 步驟四: 執行程式

```shell
$ python sync_daily.py
```

ps. 由於 183 是 SYNC_DAILY，所以才執行這個指令，如果是 hourly 才是執行另外一個檔案