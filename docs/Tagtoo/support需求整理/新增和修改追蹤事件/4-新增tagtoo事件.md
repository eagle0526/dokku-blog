---
title: 2930 - 新增 tagtoo 事件
sidebar_label: "4. [AddEvent] 新增 tagtoo 事件"
description: 新增 tagtoo 事件
last_update:
  date: 2024-01-29
keywords:
  - tagtoo
  - tagtoo 事件
sidebar_position: 4
---



### 詳細描述     
要做tagtoo event，廣告帳號：273380295128678


### 解決方法

tagtoo event 就是這個 `unitrack` 那一段：
```json
{
  "id": 2930,
  "google": { "trackIds": [["UA-34980571-47"], ["G-Q6JM6HV2LC"], ["G-K0W8FRESYH"]] },
  "unitrack": {
    "token": "584391d9e16be5aae9f6faf49944d39d42bd4800081a1a9415324cd627e4",
    "scopes": ["tagtoo"]
  }
}
```

不過通常我們在做新客戶的時候，就會 `跟 howard 要 token`，並設定好，所以要跟填單人確認他是要新增 tagtoo event 還是想要做 s2s，如果是s2s那就記得在scope那邊要補上 s2s，並且要跟 howard 講。


### PR、需求編號
[muffet] : https://github.com/Tagtoo/muffet/pull/2866/files