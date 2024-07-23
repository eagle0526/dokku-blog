---
sidebar_position: 2
---

## Google App Engine
* 此教學為如何 deploy 自己的專案到 GAE 上面
* 參考文章：https://ikala.cloud/google-app-engine-host-website/

### 第一步驟: 準備檔案

可以直接點下上方的參考文章，他們有提供一下下載範例(https://gaesamplesite.appspot.com/downloads.html)，這個範例裡面有包含主要的 `index.html` 和 `app.yaml` 檔案，部署會需要這些檔案


### 第二步驟: 創建 Google 雲端平台專案

這個很簡單，就是指示單純到 GCP 專案那邊，創建一個新的 `project`，名字可以先隨便取



### 第三步驟: 修改檔案

* 這邊我們先來修改一下範例檔案，改成這樣是由於原本提供的檔案，已經有點過時， GAE 已經沒有支援 `python2` 版本，所以來改成 `python3` 版本

```yaml
# SAMPLE-APP/app.yaml

runtime: python39

handlers:
- url: /
  static_files: website/index.html
  upload: website/index.html

- url: /
  static_dir: website
```


### 第四步驟: 部署你的應用程式

* 先確認一下位置
```shell
$ pwd

------
/Users/yee0526/Downloads/sample-app
```

* 改完 `yaml` 檔案之後，就可以來準備部署了，首先我們連接 `gcloud`:

```shell
$ gcloud init 

------
## 選擇剛剛建立專案的信箱帳號
```

* 執行部署指令，不過這時候會遇到一個問題，就是沒有設定 `Cloud Storage` 權限

```shell
$ gcloud app deploy 

------
Beginning deployment of service [default]...
╔════════════════════════════════════════════════════════════╗
╠═ Uploading 1 file to Google Cloud Storage                 ═╣
╚════════════════════════════════════════════════════════════╝
File upload done.
Updating service [default]...failed.                                                                                                                                                                                          
ERROR: (gcloud.app.deploy) Error Response: [13] Failed to create cloud build: com.google.net.rpc3.client.RpcClientException: <eye3 title='/ArgoAdminNoCloudAudit.CreateBuild, FAILED_PRECONDITION'/> APPLICATION_ERROR;google.devtools.cloudbuild.v1/ArgoAdminNoCloudAudit.CreateBuild;invalid bucket "staging.flash-nimbus-429604-q1.appspot.com"; default Cloud Build service account or user-specified service account does not have access to the bucket;AppErrorCode=9;StartTimeMs=1721117079493;unknown;ResFormat=uncompressed;ServerTimeSec=2.363718952;LogBytes=256;Non-FailFast;EndUserCredsRequested;EffSecLevel=none;ReqFormat=uncompressed;ReqID=a49c14973a032d99;GlobalID=0;Server=[2002:a05:730a:b2a5:b0:4d6:7ad9:a9aa]:4001.
  Yee Mac-Air  ~/Downloads/sample-app 
```

Ps. 因為我們在部署到 GAE，會用到 Cloud Storage 的功能，會需要把專案的檔案都丟到 `storage` 裡面，所以我們會需要到 `IAM` 開設權限


### 第五步驟: IAM開設權限

接著我們到 `IAM` 把權限給設定好：

* 參考文件: `https://kejyuntw.gitbooks.io/google-cloud-platform-learning-notes/content/google-cloud-sql/proxy/google-cloud-sql-proxy-README.html`


Ps. 在開設 `設定服務帳戶角色` 的時候，要記得設定 `環境與 storage 物件管理員`，這樣才可以有權限部署上去


### 第六步驟: 再次部署

* 加好權限後，我們再來輸入一次，如果有顯示以下類似資訊，就代表成功部署了：

```shell
$ gcloud app deploy 

------
Services to deploy:

descriptor:                  [/Users/yee0526/Downloads/sample-app/app.yaml]
source:                      [/Users/yee0526/Downloads/sample-app]
target project:              [flash-nimbus-429604-q1]
target service:              [default]
target version:              [20240716t163811]
target url:                  [https://flash-nimbus-429604-q1.de.r.appspot.com]
target service account:      [220337819286-compute@developer.gserviceaccount.com]


Do you want to continue (Y/n)?  Y

Beginning deployment of service [default]...
╔════════════════════════════════════════════════════════════╗
╠═ Uploading 0 files to Google Cloud Storage                ═╣
╚════════════════════════════════════════════════════════════╝
File upload done.
Updating service [default]...done.                                                                                                                                                                                            
Setting traffic split for service [default]...done.                                                                                                                                                                           
Deployed service [default] to [https://flash-nimbus-429604-q1.de.r.appspot.com]

You can stream logs from the command line by running:
  $ gcloud app logs tail -s default

To view your application in the web browser run:
  $ gcloud app browse
```


1. 指令一: `gcloud app browse` : 讓網站跑起來
2. 指令二: `gcloud app logs tail -s default` : 查看網站的 log


### 第七步驟: 啟動網站

```shell
$ gcloud app browse

------
Opening [https://flash-nimbus-429604-q1.de.r.appspot.com] in a new tab in your default browser.
```

最後輸入: `https://flash-nimbus-429604-q1.de.r.appspot.com` 網址就可以看見目前網站的執行狀況





