---
title: 2079 - 新增 customEvent 事件
sidebar_label: "5. [AddEvent] 新增 customEvent 事件"
description: 新增 customEvent 事件
last_update:
  date: 2024-04-23
keywords:
  - tagtoo
  - tagtoo 事件
sidebar_position: 5
---



### 詳細說明

"我們幫客戶做GA4事件（ 公用帳號：tagtooga）  
客戶希望在view item事件增加兩個自訂參數  
1. 目前參與募資的人數（contributors）
2. 目前累積的募資金額（totalFunds）
要抓取的欄位請參考截圖"  

需要抓的圖片: https://drive.google.com/open?id=1r2DGyUO6ejiHHlllpIOK4oKQRswuUPYP   

### 解決辦法

"[muffet] : 使用客製化事件(customEvent)完成抓取這個兩個參數，目前兩個欄位的格式有先給 nina 看過"


### PR、需求編號
[muffet] : https://github.com/Tagtoo/muffet/pull/2949
[編號] : 3851  

