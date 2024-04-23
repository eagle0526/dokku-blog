---
title: 1697 - purchase 數據落差太大，使用 GTM 串接 purchase 原始渠道碼
sidebar_label: "1. [data survey] 修正 purchase 數據落差太大的問題"
description: 新增 tiktok 事件
last_update:
  date: 2024-04-23
keywords:
  - tagtoo
sidebar_position: 1
---



### 詳細說明
客戶的ga4：378084356（tagtooga_gxbd@tagtoo.org）經上次調整後，客戶還是覺得數據上有落差，內部業績4萬但ga4只記錄到1.5萬 


### 解決方法

前情提要，此客戶之前已經修改過很多次 muffet 的購買流程，包括在購買完成頁面再抓一次，或者是手機版也抓一次，但是事件掉的機率還是很高，因此跟 Howard 討論後，決定跟 Orbis 一樣，用客戶的 GTM 來設定 `g-tag 原始的購買渠道事件`。    
   
這樣設定的好處是，可以讓購買事件觸發順序提前，可以提前觸發的原因是因為，muffet 整個檔案中，我們做了蠻多事情，當到達 `完成頁面` 的時候，會先觸發其他設定後，才會送出 `purchase` 事件，但是如果今天是用 GTM 設定的話，由於是直接設定原始渠道碼，當到達完成頁面的時候，購買事件就會直接觸發，這樣就可以達成 `提前觸發購買事件`。

#### 設定原始渠道事件注意事項 
1. 客戶開給我們的GTM帳號，在 tagtooshop@tagtoo.com 這個帳號內，而 GTM ID 為 `GTM-KK6RJWV`，到達該帳號後，會發現兩個 TAG `Tagtoo GA4 Purchase Page Init`、`Tagtoo GA4 Purchase Page Event`。
2. 我幫客戶設定的事客製化事件，不是標準的 purchase 事件，因為當初想要做的是，保留 muffet 的 purchase 事件，並且新增一個由 GTM 設定的客製化事件，不過這個客製化事件會有訂單的詳細資料，這樣就可以兩者比較訂單的差異
3. 如果今天要不要用客製化事件，也就是只留下 GTM 的 purchase 事件，要記得讓 muffet 的 purchase 事件不要觸發，要怎麼設定呢？可以到 orbis 的 config 檔案最下方可以找到，簡單的說就是，除了購買完成頁面的事件不要觸發，其他頁面的事件都正常觸發

#### 收益比較
在 1697 的GA4，有設定好比較的客製化報表，以下是數據比較：  
   
成效紀錄，目前看10/28~10/29的數據，記錄到的比數有變多：    
muffet - 原始記錄到的筆數：43筆訂單 - 66,680   
GTM - 新事件記錄到的比數：49筆訂單 - 75,057  
客戶給的訂單比數：57筆訂單 - 88,809   