---
title: tagtoo/cron-job
sidebar_label: "5. [tagtoo] cron-job"
description: cron-job 是關於 line feed 的產出檔案
last_update:
  date: 2023-11-08
keywords:
  - Muffet
  - cron-job
tags:
  - Muffet
sidebar_position: 6
---



:::tip
如果今天遇到一個 feed 長得像這樣 `ftp://cron-jobs.tagtoo.com.tw/line-beams.csv`，要怎麼下載或修改內容呢?  
https://github.com/Tagtoo/cron-jobs  
:::    



解決步驟
------

### 步驟一 : 下載 FTP 連線軟體

1. 免費 FileZilla
2. 付費 cyberduck


### 步驟二 : 連線進入

這邊我用 FileZilla 這個軟體當作示範

1. 先開啟這個檔案 - https://docs.google.com/document/d/13Fyvh8xi1J1blpIfjH2drC0xFCaa7u11o_RzHsthRww/edit ，裡面有所有的帳密資料
2. 確認目前使用的 feed 和名稱是哪一個，目前我確認到 feed 是這個 `ftp://cron-jobs.tagtoo.com.tw/line-beams.csv`，並且ID是 `TagtooBeamsLineFeed`
3. 確認好之後打開軟體並開始連接，主機 - `cron-jobs.tagtoo.com.tw`、使用者名稱 - `TagtooBeamsLineFeed`、密碼 - `上面的連結有`，這樣就成功連結
4. 連進去後就可以下載檔案




### 步驟三 : cron-job 服務介紹

cron-job 這個檔案，也是 tagtoo feed 服務的其中一環，他主要是產生 CSV 檔案，但是 api 不是就可以產生 csv 檔案了嗎？為什麼還要用 cron-job 呢？原因就是有些渠道不接受 https 協議，只接受 ftp 協議，像是 Line 的 feed 就是一定要用 FTP。    
而 cron-job 主要是還是在負責產生 CSV 檔案，他會向 api 檔案發出請求，把資料拉過來，所以如果今天客戶想要改 feed 的某些設定，比如上標籤的話，還是要用 api 檔案修改，cron-job 檔案主要只是提出需求。   
   
Ps. 如果今天只是單純改 UTM 參數，要在 crob-job 檔案中修改喔！  


### 步驟四 : cron-job GCP

由於是使用 FTP 設定，所以我們在雲端開了一個 VM，裡面有一台 FTP SERVER，檔案都在那裡面。    
Ps. cron-job 專案在 tagtoo-new 裡面


如果想要進入觀看的話，可以用 gcloud 指令登入查看：

#### 4-1 : 先初始化，並登入 tagtoo 的帳號
```shell
$ gcloud init
```
#### 4-2 : 指定 cron-job 的GCP

選擇 `[1] dashboard-dev-1230`

```md
Pick cloud project to use: 
 [1] dashboard-dev-1230
 [2] gothic-province-823
 [3] starlit-cocoa-390006
 [4] tagtoo-adchief
```

#### 4-3 : 連進 cron-job

```shell
$ gcloud compute ssh --zone "asia-east1-a" "cron-jobs" --project "dashboard-dev-1230" 
```


最後再輸入這一行：

```shell
yee0526@cron-jobs:~$ cd /mnt/cron-jobs-data/github/cron-jobs
```


### 檔案介紹



## 新增 cron-job 服務

:::tip
如果今天有一個新客戶想要使用 Line Feed 要怎麼做？
:::

首先可以看一下 line_lap_feed 這個資料夾，主要的 feed 產生都在這裡:

### config.yaml
首先會根據 config.yaml，產生指定客戶的 line feed (填上 ec_id) 就好，上面的 yaml 裡面的 name 就是指 CSV 的檔案名稱。   
其他參數就是 UTM 上會帶的參數。  


### download.py

這個檔案就是會去 api 那邊把資料都抓過來，最後產生 CSV 檔案！
想要測試可以使用 ipython，直接在 ipython 環境裡面 import fc 然後看結果就好。



### 看目前所有的 feed

如果今天想要看所有 cron-job 的 feed 可以照著下面輸入：

#### 先初始化，並登入 tagtoo 的帳號
```shell
$ gcloud init
```
#### 指定 cron-job 的GCP

選擇 `[1] dashboard-dev-1230`

```md
Pick cloud project to use: 
 [1] dashboard-dev-1230
 [2] gothic-province-823
 [3] starlit-cocoa-390006
 [4] tagtoo-adchief
```

#### 連進 cron-job

```shell
$ gcloud compute ssh --zone "asia-east1-a" "cron-jobs" --project "dashboard-dev-1230" 
```


#### 到達 etc/crobtab

```shell
yee0526@cron-jobs:$ cd /etc
yee0526@cron-jobs:/etc$ vim crontab
```

這樣可以進到 `crontab` 這個檔案，可以看到裡面有所有 feed 的詳細資訊！



### 幫客戶新增一個 FTP 帳戶 ----- 以下不確定

* 如果今天客戶的資料量太大，我們會幫客戶多開一個帳號

幫客戶開一份自己個 FTP 帳號密碼 - 在設定一個定期的時間，定期把內部的檔案， copy 一份到客戶的資料夾底下

做法在這邊 ： https://github.com/Tagtoo/cron-jobs/wiki/Create-a-new-FTP-account


## Line feed 失效

### 先登入 gcloud 中
```shell
$ gcloud compute --project "dashboard-dev-1230" ssh --zone "asia-east1-a" "cron-jobs"
```

### 查看各家客戶的 code 資訊

```shell
$ cd /mnt/cron-jobs-data/github/cron-jobs/cron_jobs/line_lap_feed
```


### 查看各家客戶的 CSV 資訊

```shell
$ cd /mnt/cron-jobs-data/vsftpd-virtual-root/TagtooLineFTPFeed 
```

### 進到 line_lap_feed 執行 ec_download 指令

如果今天遇到 line_feed 失效，可以直接輸入以下的指令，看看目前的 cron-job 是否有正常運作。

```shell
$ cd /mnt/cron-jobs-data/github/cron-jobs/cron_jobs/line_lap_feed
$ sudo python ec_download.py 107
```
ps. 執行 `ec_download.py` 這個程式，後面 107 是 ec_id 的編號


如果輸入完後，還是發現 csv 無法產出，可以出入以下指令來查看 log：

```shell
$ cd /mnt/cron-jobs-data/github/cron-jobs/logging
$ cat line_lap_feed.log | grep {ecid}
```


 


