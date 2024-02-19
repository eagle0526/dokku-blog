---
title: SUSU - 客製化 feed
sidebar_label: "2. [reviseFeed] SUSU - 客製化 feed"
description: 動態饋給問題
last_update:
  date: 2023-12-20
keywords:
  - tagtoo
  - 動態饋給問題
sidebar_position: 2
---


由於 susu 的 feed 需求比較特別，因此要新增一段 queryString 來 query 指定的 產品



### 詳細描述     

目前在撈取 SOUSOU 中文商品資料時，會先撈取 15000 個商品，
撈出來後再去篩選出那些是中文的商品，所以可能會漏掉一些商品，
我們希望可以透過像 https://api.tagtoo.com.tw/v2/feed/facebook/special/1878?lang=tw 的網址，
就拿到「所有的」中文商品。


### 解決方法
方法: 在撈取產品時，會先判斷 feed link 是否含 pk_lang 的 query string，有的話會先根據 product_key，過濾出該語言產品，再進行其他產 feed 邏輯 。


### PR、需求編號
https://github.com/Tagtoo/api/pull/1836