---
title: tagtoo/tracknew
sidebar_label: "13. [tagtoo] tracknew"
description: muffet 的前端更新是如何匯入資料庫
last_update:
  date: 2023-11-08
keywords:
  - Muffet
  - tracknew
tags:
  - Muffet
sidebar_position: 9
---



## tracknew

這個 repo 就是 muffet 的前端更新，這一段的來源：

```js
const product = new Product(productInfo)    
insertPixel(`//track.tagtoo.com.tw/up?t=u&${product.toQueryString()}`)
```
