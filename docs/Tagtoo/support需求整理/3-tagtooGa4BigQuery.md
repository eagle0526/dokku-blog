---
title: 2990 - ga4-biqQuery
sidebar_label: "3. [bigQuery] 2990 到 ga4-biqQuery 看資訊"
description: BigQuery
last_update:
  date: 2024-01-29
keywords:
  - tagtoo
  - BigQuery
sidebar_position: 3
---



### 詳細描述     
由於大成的 GA4 數據一直有問題，所以我們幫客戶的 GA4 串了 BigQuery，直接來看送到 BigQuery 的數據是否有誤


### 解決方法
#### (1) 通知 patrick 串接 BigQuery

1. 正常情況下，客戶如果有自己的 GA4，OP或業務那邊會直接填單請 Patrick 串接 BigQuery
2. 而如果是 tagtoo 的 GA4，是由我們這邊來提醒 Patrick 要串接 BigQuery - 直接私訊 Patrick 就好

Ps. 記得提供評估ID、ec_id

#### (2) 到 GCP 看數據

過幾天後，等BigQuery有資料後，就可以上去看數據了～


1. 到 Tagtoo-GA4 這個 GCP 帳號，選擇 BigQuery 的項目，就可以看到有很多 GA4 資料在這邊
2. 找到相對應的 GA4 數據，檔案命名的規則是 - analytics_GA4名稱. ex: analytics_395350185
3. 接著點選進去，點擊 `查詢` -> `新分頁開啟` -> `輸入 SQL 指令` 

假設今天我想要找出0804的 `purchase` 資料，可以這樣輸入：

```sql
SELECT * FROM `tagtoo-ga4.analytics_395350185.events_20230804` where event_name = 'purchase' LIMIT 1000
```

這樣我們可以得到這一天，所有購買的資料。



### PR、需求編號
