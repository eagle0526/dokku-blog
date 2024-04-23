---
title: 1859 - 新增 pageView 追蹤事件
sidebar_label: "2. [AddEvent] 新增 pageView 事件"
description: 新增 pageView 事件
last_update:
  date: 2024-01-29
keywords:
  - tagtoo
  - pageView 事件
sidebar_position: 2
---



### 詳細描述     
新增 1859 的 pageView 事件


### 解決方法

需求單很少會只要求新增 `pageview` 事件，因為 pageView 事件是很基本的事件，你在新增以下的 `config` 設定時，就會自動觸發 `pageView` 事件

```json
{
  "id": 1859,
  "google": {
    "trackIds": [["UA-34980571-37"]]
  },
  "facebook": {
    "trackIds": [["6167890019991870"]]
  }
}
```

接下來只要在 `index` 的地方設定好 `trigger`，就會自動觸發 `pageView` 事件

如果今天想要新增 `GADS` 的 `pageView` 像以下設定就好

```json
{
  "id": 1859,
  "google": {
    "trackIds": [["UA-34980571-37"], ["AW-11202968721"]]
  },
  "facebook": {
    "trackIds": [["6167890019991870"]]
  },
  "conversions": [
    {
      "action": "pageView",
      "google": "AW-11202968721/bz0ZCMOBia4YEJH5_t0"
    }
  ]
}
```