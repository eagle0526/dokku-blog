---
title: 2878 - 新增 Line UTM 參數
sidebar_label: "1. [UTM] 新增Line UTM參數"
description: 修改 Line 的 UTM 參數
last_update:
  date: 2023-11-07
keywords:
  - tagtoo
  - Line UTM
sidebar_position: 1
---

### 詳細描述
Line 目錄每個商品的到達網址後面可以幫我統一加上「utm_source=tagtoo&utm_medium=cpc&utm_term=2878:474:0」這個嗎～感恩


### 解決方法
到 Tagtoo / cron-jobs 這個 repo，加上下面這一段，這樣加完之後，line feed 產出的檔案，產品上就會帶上 UTM 的連結

```yaml
<!-- cron_jobs/line_lap_feed/config.yaml -->

2878:
  source: tagtoo
  medium: cpc
  term: 2878:474:0
```

### PR、需求編號

[cron-job] : https://github.com/Tagtoo/cron-jobs/pull/187
[編號] : 3611