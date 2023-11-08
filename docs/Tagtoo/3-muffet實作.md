---
title: tagtoo/muffet
sidebar_label: "3. [tagtoo] muffet 實作"
description: muffet 是 tagtoo 的前端爬蟲
last_update:
  date: 2023-11-07
keywords:
  - tagtoo
  - muffet
tags:
  - Muffet
sidebar_position: 2  
---

正式開始寫 - 第一間 2878
------

### 1. 7 行碼確認
確認客戶是否有把 `7行碼` 埋到官網 -> 直接在 `console` 輸入 `tagtoo`，看有沒有值出現，或者是用 `network` 查看也可以

### 2. 要渠道權限
跟 patrick 要 facebook 、 GADs 、 GA 權限，要從後台看新客戶的像素有哪些  
Ps. 這個網頁有塔圖所有帳號密碼：https://docs.google.com/document/d/13Fyvh8xi1J1blpIfjH2drC0xFCaa7u11o_RzHsthRww/edit#，有所有的帳號/密碼設定     

### 3. 設定 muffet 檔案

#### 3-1 新增客戶

```shell
$ npm run create-ec 2878
```
這樣會多一個資聊夾，這個資料夾就是我們要寫爬蟲的地方

#### 3-2 安裝 npm 

這個可以安裝 muffet 所需要的插件
```shell
$ npm install
```

#### 3-3 npm run preview 2878

接下來我們就可以在終端機執行這一段程式碼

```shell
$ npm run preview 2878
```

Ps. 啟動後，可以按鍵盤上的快捷鍵，`q` 是離開啟動環境，`s`會印出一段程式


按下 `s` 後，會印出下面這一段：
```shell
window.tagtoo_advertiser_id = 2878;
window.tgDataLayer = window.tgDataLayer || [];
function tgk() { window.tgDataLayer.push(arguments); }
const $script = document.createElement('script');
$script.charset = 'UTF-8';
$script.src = `http://localhost:3000/${window.tagtoo_advertiser_id}.js`;
document.body.appendChild($script);
tgk('autoRun');
```

#### 3-4 使用 CJS
  
接著我們就可以打開 CJS 插件，並且把剛剛那一段 code，貼上去後啟動，這樣就可以在本地端開發了。    
Ps. 這樣做的原因，是因為我們現在還沒有 push 到客戶網站上，所以如果用原本的 `ModHeader` 的方法，會無法在本地端觸發那些事件   


### 4. ad online check 確認這過網站有哪些像素

這個表單也可以參考，這裡有所有客戶的像素表單整理
https://docs.google.com/spreadsheets/d/1eW2kmFs1NU2zSYrlgDw-QrCCNUH7kOrAiGBeCVRo5KI/edit#gid=1592073239


Ps. 自從 `Howard` 把 track.js 搬到 muffet 後，就不用在進行第五步驟，改成進行第六步驟就好

### 5. 設定 Dashboard-legacy 的 media/ad/track.js，需要再用 minify-track 這個 script 把 track.js 做檔案壓縮

照著以下步驟完成:  

#### 5-1 找到 dashboard-legacy repo
從 dashboard-legacy 這個 repo 找到 media/ad/track.js 檔案 -> repo 連結在這：https://github.com/Tagtoo/dashboard-legacy

#### 5-2 找到 track.js
照個這個路徑，`DASHBOARD-LEGACY/media/ad/track.js`，並把負責的 `ec_id` 加入檔案，像下面這樣:
```md
>  var ecId = window.tagtoo_advertiser_id.toString();
>   switch (ecId) {
>       case '49':
>       case '94':
>       case '100':
>       case '107':
>       ...
>       case '2878':      => 我這次的新客戶是 2878
>  }
```

#### 壓縮檔案
執行下面這一段指令

```shell
$ npm run minify-track
```

這樣輸入後，就會完成把 track.js 檔案壓縮





### 6. muffet 的 track.js

* 自從 Howard 把 `Dashboard-legacy` 的 `track.js` 檔案轉移到 `muffet` 後，直接到 muffet 資料夾，新增新客戶的 `ec-id` 就好，以下為例：

```md
<!-- src/track.js/index.js -->

>  var ecId = window.tagtoo_advertiser_id.toString();
>   switch (ecId) {
>       case '49':
>       case '94':
>       case '100':
>       case '107':
>       ...
>       case '2878':      => 我這次的新客戶是 2878
>  }
```

加上去後就可以準備發 PR 了

### 7. 發 PR 注意事項

* branch 名稱可以用 EC_ID-john 這樣的格式，或者單純 EC_ID 就可 - ex. 2878
* PR 名稱就用 feat 那個 - ex. feat(ec/2878): new client
* PR 的內容區塊要寫完整一點，以這個為例：https://github.com/Tagtoo/muffet/pull/2681
* 因為塔圖要設定自動跑 CI/CD，如果今天沒有設定好，會自動發生 error，如果PR發生錯誤，可以點擊GCB DETAIL，會進到他跑 CI/CD 的地方，他顯示你哪邊有錯誤，根據你錯的地方再修改就好

Ps. patrick 會幫你開塔圖帳號的GCP，記得 chrome 的使用者要先改成塔圖的，要不然他會說你沒權限



### 8. muffet 設定細節

#### 8-1 trackProduct
tracker('viewItem', product)
tracker('addToCart', product)
tracker('addToWishlist', product)


#### 8-2 trackCartPage
tracker('trackCart', cart)      
tracker('viewCart', cart)      
tracker('checkout', cart)   
tracker('addShippingInfo', cart)            
tracker('addPaymentInfo', cart)   


#### 8-3 trackCheckout
tracker('checkout', cart)

#### 8-4 trackPurchase
tracker('purchase', cart) 

#### 8-5 trackSearch
tracker('search', searchKey)

#### 8-6 trackRegister
tracker('register', 'member')
tracker('register', 'line')
tracker('register', 'facebook')

#### 8-7 trackLogin
tracker('login', 'member')
tracker('login', 'line')
tracker('login', 'facebook')




### 一些設定備註

如果今天有一個案例，有兩個網址如下：

產品頁：https://www.jdgift.com.tw/products/plaid-picnic
搜尋頁：https://www.jdgift.com.tw/products?query=%E7%B4%85%E8%B1%86

可以看到兩個網頁都有包含 products，所以如果你今天產品頁的觸發是像這樣

```md
> pageView.path.contains('products')
```
就可能造成當你在做搜尋動作的時候，也觸發了產品頁的觸發動作，因此我們可以加上這一段
```md
> pageView.path.contains('products').and(pageView.path.not.contains('?query'))
```

這樣就可以解決兩個網址都有一樣字串的問題。


前人寫的案例：
```md
> trigger: pageView.path.contains('/Product').and(pageView.path.not.contains('/Brand')),
```


### 額外注意事項

1. facebook 事件是 OP 要設定，如果他們沒設定，要記得跟他們溝通
2. google ADwords 是 teddy 會設定，不過他不會馬上設定，所以最後在把轉換像素設定好就好


