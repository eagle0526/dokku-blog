---
title: tagtoo/google 強化轉換
sidebar_label: "11. [tagtoo] google 強化轉換"
description: google 強化轉換
last_update:
  date: 2024-01-17
keywords:
  - Muffet
  - google 強化轉換
  - GADS
tags:
  - Muffet
sidebar_position: 11
---



## Google 強化轉換

howard 的文件 : https://github.com/Tagtoo/muffet/issues/2287
Ps. 這一份文件只有紀錄後端部分，前端部分和GADS那邊要做的事情看 howard 的文件就好

## 本機開發 - 流程大綱(後端部分)

1. 先到 tagtoo 的 GCP 專案這個位置  `Project: tagtoo-staging > Cloud Functions > send_enhance_s2s > SOURCE > Cloud Source repository`
2. 到 `get_conversion_actions.py` 這個檔案位置，裡面內容可以看到，把這個repo 拉下來 - `https://github.com/Tagtoo/ad-tools`
3. 拉下來後，先安裝此檔案需要的相關 `requirement - 我是只缺 google-ads` ，所以只安裝 `pip install google-ads` ，接著 `cd `到 `get_conversion_actions` 這個資料夾裡面，先修改 `GADS ID`，再執行此指令 `python get_conversion_actions.py`，terminal會印出一大段東西給你


4. 再來到 `cloudSourceRepositories` 點擊 `建立本機副本` ，把整個 `repo` 抓下來，第一次使用有機會遇到 `ssh 憑證` 還沒設定的問題，可以到這個連結 `https://cloud.google.com/source-repositories/docs/authentication?&_ga=2.1532977.-1682270343.1695012024#ssh` 看怎麼設定，設定好 `gcloud 憑證` 後，再點擊一次 `建立本機副本`，就可以把整個檔案拉下來的
5. 打開剛剛的 repo，到 main.py 這個檔案，把剛剛得到的那一串資料加上去，改好後直接 `git add .   git commit -m   git push` 推上去
6. 現在再到 cloudSourceRepo 的頁面，就可以發現剛剛的 commit 有成功推上去了
7. 最後到 cloud function 那邊點選編輯，並且把它部署上去 (有可能會沒有權限，沒有的話要請 patrick 開給你)


## 本機開發 - 詳細步驟(後端部分)

### 步驟一
進到 tagtoo 的 GCP，並且找到 `tagtoo-staging` 的 `Cloud Functions`，`send_enhance_s2s > SOURCE > Cloud Source repository` 接著照著這個路徑，最後會看到下圖   
![enhanced-1](./img/enhanced%20-%201%20-%20cloudSourceRepositories%20.png)   

Ps. 也可以點這個連結直接到 `tagtoo-staging` : https://console.cloud.google.com/functions/list?referrer=search&project=tagtoo-staging   

### 步驟二

把 `ad-tools` 這個檔案給拉到本地端：https://github.com/Tagtoo/ad-tools   

### 步驟三

拉下來後打開它，`cd `到 `get_conversion_actions` 這個資料夾中，再來執行 `python get_conversion_actions.py` 這個指令，如果今天你有沒安裝好的 requirement，會噴錯誤，像我就缺的 `google-ads` 這個套件。    
因此把它安裝好 `pip install google-ads`，等他跑完後，再一次執行指令 `python get_conversion_actions.py`，你會在終端機發現印出一大段類似這樣的資料：   

檔案會吐這一段內容給你:
```md
'R_M8CJqY7sEYEJipyv8p': 'customers/1863648729/conversionActions/6580571162',  # cocomelody  購買
'1C4QCP77x8MYEJipyv8p': 'customers/1863648729/conversionActions/6584139262',  # add to cart
'UC_iCIH8x8MYEJipyv8p': 'customers/1863648729/conversionActions/6584139265',  # checkout
'2JMzCIT8x8MYEJipyv8p': 'customers/1863648729/conversionActions/6584139268',  # view item
'KpBQCIf8x8MYEJipyv8p': 'customers/1863648729/conversionActions/6584139271',  # page view
'fOteCMnW04AZEJipyv8p': 'customers/1863648729/conversionActions/6712257353',  # 25 %
'HKETCMzW04AZEJipyv8p': 'customers/1863648729/conversionActions/6712257356',  # 50 %
'KmKuCM_W04AZEJipyv8p': 'customers/1863648729/conversionActions/6712257359',  # 75 %
'y6yQCMrX04AZEJipyv8p': 'customers/1863648729/conversionActions/6712257482',  # 100 %
'ge2SCM3X04AZEJipyv8p': 'customers/1863648729/conversionActions/6712257485',  # 次要 學習事件動作
```

這個就是我們等等要用的 `label` 和 `resource`，但是我們還要改個東西，剛剛我們印出來了預設的 GADS 的轉換資料，我們要把 `customer_id = '9883037619'` 這一段換成這一次要設定的 GADS，要怎麼換呢？  

#### 3-1 先找到此次客戶的 GADS ID
![enhanced-2](./img/enhanced%20-%202%20-%20where-GADS-ID.png)

#### 3-2 把 id 回填到 本地端的資料
![enhanced-3](./img/enhanced%20-%203%20-%20revise-GADS-ID.png)

#### 3-3 最後再執行一次指令就可以了
```shell
$ python get_conversion_actions.py
```


### 步驟四
再來我們要把這一份轉換資料，回填到 `main.py` 裡面，我們先把整個檔案拉到本地端做，因此一樣先到 `cloud source repositories` 的地方，找到 `建立副本` 的按鈕，點下去就會有連結可以下載到本地端   
![enhanced-4](./img/enhanced%20-%204%20-%20clone%20main.py%20file.png)   

Ps1. 拉到本地端是因為不知道怎麼用 `cloud shell` 修改   
Ps2. 如果今天是第一次使用 gcloud，會遇到 `ssh 憑證` 還沒設定的問題，可以到這個連結 `https://cloud.google.com/source-repositories/docs/authentication?&_ga=2.1532977.-1682270343.1695012024#ssh` 看怎麼設定，設定好 `gcloud 憑證` 後，再點擊一次 `建立本機副本`，就可以把整個檔案拉下來的   


### 步驟五

接著進到檔案裡面，把前面得到的 label 和 resource 填上去就好，最後直接 `git add .   git commit -m   git push` 推上去  
![enhanced-5](./img/enhanced%20-%205%20-%20detail.png)   



### 步驟六

現在再到 cloudSourceRepo 的頁面，就可以發現剛剛的 commit 有成功推上去了


#### 推成功前的 cloud source repositories 的樣子
![enhanced-6](./img/enhanced%20-%206%20-%20local%20push%20before.png)


#### 推成功後的 cloud source repositories 的樣子
![enhanced-7](./img/enhanced%20-%207%20-%20local%20push%20final.png)

### 步驟七

修改完後，要記得部署，他才能完成啟動
![enhanced-8](./img/enhanced%20-%208%20-%20deploy.png)

Ps. 如果遇到權限不足，像是下圖的問題，要跟 patrick 要權限
![enhanced-9](./img/enhanced%20-%209%20-%20deploy權限不足.png)





