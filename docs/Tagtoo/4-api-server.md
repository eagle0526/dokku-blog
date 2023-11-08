---
title: tagtoo/api
sidebar_label: "4. [tagtoo] api-server"
description: api server 是關於 feed 產出邏輯的檔案
last_update:
  date: 2023-11-08
keywords:
  - Muffet
  - api-server  
tags:
  - Muffet
sidebar_position: 4
---

## 本機開發 - 流程大綱

1. git clone git@github.com:Tagtoo/api.git
2. git submodule init
3. git submodule update
4. 修改 api 檔案裡面的 database
    - app.yaml -> MySQLdb remove
    - requirements.txt -> add PyMySQL==0.10.1
    - postback.py、cloudsql.py、gen_case.py 這三個資料夾要修改 -> import MySQLdb -> import pymysql as MySQLdb
5. make install-requirements
6. make run-dev-appserver	

Ps. 第六點通常如果你沒有設定好 gcloud 的話，會無法啟動，所以這時候要去安裝 gcloud sdk

7. 安裝 gcloud

8. 安裝好後，開始以下的步驟輸入指令
    - gcloud init
    - gcloud components install app-engine-python
    - dev_appserver.py --help

Ps. 這時候如果有跑出有印出一大堆 `dev_appserver` 的資訊，就代表安裝成功了，不過我當初 python 環境沒搞好，所以一直發生衝突，建議大家用 `pyenv` 來管理多版本 python

9. 裝好 pyenv，一次設定好 python2 和 python3
    - pyenv global 3.11.4 2.7.18   

10. 再執行一次 `make run-dev-appserver`

11. 打開連結 `http://localhost:8080/v1/feed/facebook/2162`

12. 修改 API 檔案，看變化



## 本機開發 - 詳細步驟

### 步驟一：先把 api 的repo clone 下來
```shell
$ git clone git@github.com:Tagtoo/api.git
```

### 步驟二：執行初始化指令

```shell
$ git submodule init

------
子模組 'lib/ec'（git@github.com:Tagtoo/ec.git）已對路徑 'lib/ec' 註冊
子模組 'lib/serve-static'（git@github.com:Tagtoo/serve-static.git）已對路徑 'lib/serve-static' 註冊
子模組 'lib/share_libs'（git@github.com:Tagtoo/share_libs.git）已對路徑 'lib/share_libs' 註冊
```

### 步驟三：執行初始化指令
```shell
$ git submodule update

------
正複製到 '/Users/yee0526/Desktop/Feed API/api/lib/share_libs'...
Enter passphrase for key '/Users/yee0526/.ssh/id_rsa': 
正複製到 '/Users/yee0526/Desktop/Feed API/api/lib/serve-static'...
Enter passphrase for key '/Users/yee0526/.ssh/id_rsa': 
子模組路徑「lib/ec」：已簽出「637b07a3fc0def5aa8745acfedcc238a5fb5b196」
子模組路徑「lib/serve-static」：已簽出「da3dfb1a4f424a0e6603c6e4c44f8ac08a2fbcb3」
子模組路徑「lib/share_libs」：已簽出「f44ca3b8d6544aad0539ea414811c51e240e66eb」
子模組路徑「lib/serve-static」：已簽出「da3dfb1a4f424a0e6603c6e4c44f8ac08a2fbcb3」
```


### 步驟四：修改 api 檔案裡面的 database

直接看 Howard 寫的教學文件，裡面有附圖 - https://docs.google.com/document/d/1Ct2Y2eou7p4JocCyi8YoXBLVjAfE2TGa23PCw27t9zw/edit#heading=h.8vtr48y8qaw2




### 步驟五：安裝檔案

```shell
$ make install-requirements
```

### 步驟六：啟動 server
```shell
$ make run-dev-appserver
```

Ps. 這一步驟有很大機會錯誤，像是要安裝 gcloud、dev_appserver 或是基本一定會遇到的 python 版本問題，所以我們一步驟一步驟來，先來安裝 gcloud



### 步驟七：安裝 gcloud

根據官網文件下載 - https://cloud.google.com/sdk/docs/install


### 步驟八：安裝 dev_appserver.py

安裝 Google Cloud SDK：請確保您已經安裝了 Google Cloud SDK。您可以從 Google Cloud 官方網站下載並按照安裝指南進行安裝：https://cloud.google.com/sdk/docs/install  
配置 Google Cloud SDK：在安裝完成後，請執行以下命令以初始化 Google Cloud SDK：   

```shell
$ gcloud init
```

按照提示進行操作，選擇您的帳戶並設定項目。   
安裝 App Engine 相關組件：執行以下命令來安裝 App Engine 相關組件：   

```shell
$ gcloud components install app-engine-python
```

這將安裝 App Engine 所需的 Python 相關組件。     
驗證安裝：執行以下命令來驗證 dev_appserver.py 是否安裝成功：         


```shell
$ dev_appserver.py --help
```

如果您看到 dev_appserver.py 的幫助訊息，則表示安裝成功。請注意，dev_appserver.py 是 App Engine 的本地開發伺服器，用於在本機模擬和運行 App Engine 應用程式。安裝和使用 dev_appserver.py 前，確保您已經了解並配置了適當的 App Engine 項目和相關設定。  


Ps. 通常這裡會遇到其他問題，像是 python 版本錯誤，所以接受來建議用 pyenv 來管理

### 步驟九：安裝 pyenv - 安裝 python2



參考連結： https://ephrain.net/python-%E4%BD%BF%E7%94%A8-pyenv-%E5%9C%A8-mac-%E4%B8%8A%E5%AE%89%E8%A3%9D%E5%B7%B2%E7%B6%93%E6%B6%88%E5%A4%B1%E7%9A%84-python-2-%E7%89%88%E6%9C%AC/


#### 9-0 : 用 pyenv 來轉換 py 的版本

由於我之前電腦裡面python的環境很亂，後來找到用這個東西可以像 `rvm` 那樣，安裝一堆python版本，並且切換到自己想要的版本

#### 9-1 : 安裝 pyenv

```shell
$ brew install pyenv
```

#### 9-2 : pyenv install -l

看看有什麼版本的 Python 可以安裝
```shell
$ pyenv install -l 
```


#### 9-3 : pyenv versions

查看目前本機安裝的版本
```shell
$ pyenv versions

------
  system
* 2.7.18 (set by /Users/yee0526/Desktop/Feed API/api/.python-version)
  3.5.10
  3.11.4
```


####  9-4 : pyenv install 2.7.18

安裝指定版本

```shell
$ pyenv install 2.7.18
```

#### 9-5 : 設定要使用的 Python 版本
pyenv 允許使用者設定三種環境下的 Python 版本：

* shell: 設定目前 shell 階段 (session) 裡的 Python (相當於設定環境變數 PYENV_VERSION)
* local: 設定此目錄或子目錄的 Python (會建立 .python-version 檔案)
* global: 設定全域級別的 Python (會建立 $(pyenv root)/version 檔案)


下面這個指令就是設定全域的變數
```shell
$ pyenv global 2.7.18
```

#### 9-6 : 一次設定 python2 和 python3 的版本

由於 Gcloud 有時會說你 python2 沒安裝，有時又會說 python3 沒安裝，所以我們來用下面這個指令，一次設定兩種版本


輸入這行後，下面有跟你說怎麼用這個指令
```shell
$ pyenv help global

------
Sets the global Python version(s). You can override the global version at
any time by setting a directory-specific version with `pyenv local'
or by setting the `PYENV_VERSION' environment variable.

<version> can be specified multiple times and should be a version
tag known to pyenv.  The special version string `system' will use
your default system Python.  Run `pyenv versions' for a list of
available Python versions.

Example: To enable the python2.7 and python3.7 shims to find their
         respective executables you could set both versions with:

'pyenv global 3.7.0 2.7.15'
```

最後輸入我們本機已經安裝的python2和python3兩個版本，這樣就大功告成！

#### 9-7 : 一次指定 python2 和 python3 版本
```shell
$ pyenv global 3.11.4 2.7.18   
```



### 步驟十：啟動 server

```shell
$ make run-dev-appserver	

------
dev_appserver.py --env_var MYSQL_IP=104.199.199.63 --env_var MYSQL_USER=root --env_var MYSQL_PASSWORD=HadbykpemhyptOdNiOmRytKobryuInt2 .
INFO     2023-06-17 07:22:05,465 devappserver2.py:321] Skipping SDK update check.
INFO     2023-06-17 07:22:05,527 <string>:398] Starting API server at: http://localhost:60732
INFO     2023-06-17 07:22:05,553 dispatcher.py:276] Starting module "api" running at: http://localhost:8080
INFO     2023-06-17 07:22:05,554 admin_server.py:70] Starting admin server at: http://localhost:8000
WARNING  2023-06-17 07:22:05,554 devappserver2.py:417] No default module found. Ignoring.
INFO     2023-06-17 07:22:07,842 instance.py:294] Instance PID: 24319
```

輸入後，會發現這幾行，可以看到 `:8080` 那一行，這一行我們等等可以拿來做本地端設定





### 步驟十一：讀取 API


下面是 api 的格式
```md
* ​​https://api.tagtoo.com.tw/<VERSION>/feed/<CHANNEL_NAME>[/special]/<EC_ID>
    * https://api.tagtoo.com.tw/v1/feed/facebook/EC_ID
    * https://api.tagtoo.com.tw/v1/feed/facebook/special/EC_ID
```



使用說明：如果你今天的客戶 `ec_id` 是 `2162`，那你可以直接這樣輸入 `https://api.tagtoo.com.tw/v1/feed/facebook/2162`，會發現網頁上會有一大堆產品資訊，這個就是我們去打 API，得到這間廠商的所有商品資訊 (我們用 muffet 抓到的)。      
     
之後交給 OP 的就是這一行網址，這一行網址就是所謂的 `feed` ， op 會把這行網址放到目錄那邊設定。   
Ps. OP 會完成: Commerce Manager > Catalog > Data Sources > Data Feeds    


:::tip 本地更改測試
還記得我們剛剛啟動本機後的這一個嗎 `http://localhost:8080`，我們來把這個網址後面接上剛剛的正式的feed，像這樣 `http://localhost:8080/v1/feed/facebook/2162`，這樣本機就可以看到修改的結果了！
:::

### 步驟十二：實際測試修改文件

#### 12-1 : 原始狀態
我們來測試原本的客戶 API：https://api.tagtoo.com.tw/v1/feed/facebook/2162，輸入這個網址可以看到客戶網站裡面所有的產品


#### 12-2 : 增加條件
條件： "排除特定商品" ( 商品類別除了 ''美妝'' 類別的產品， 其他類別都要排除 )
```py
# api/apps/feed/query_products.py

# ...略
def skip_this_item(advertiser_id, item, publisher_id):
    # ttl    
    if advertiser_id == 2162 and publisher_id == 71:
        if u'美妝' not in item['category_path'] :
            return True

    return item['product_key'] in not_wanted            
```
像上面那樣設定好，我們來啟動server - `make run-dev-appserver`，並且輸入這個網址：`http://localhost:8080/v1/feed/facebook/2162` 就可以發現裡面的產品，都只剩下 `美妝` 類型的產品



### 參考連結：
1. [api repo] : https://github.com/Tagtoo/api
2. [Howard 教學文件] : https://docs.google.com/document/d/1Ct2Y2eou7p4JocCyi8YoXBLVjAfE2TGa23PCw27t9zw/edit
