---
sidebar_position: 2
---

0、前言
------
:::tip
**寫此篇文章的動機**  
此篇文章是用來記錄如何在本地端用 clasp 控制 app script 的版本
:::



## 1. 首先要在本機，登入 clasp 後，拿到 json 驗證檔案，因此我們先做以下幾件事情

### 1-1. 安裝 node.js
```shell
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
$ \. "$HOME/.nvm/nvm.sh"
$ nvm install 22

# Verify the Node.js version:
node -v # Should print "v22.18.0".
```

### 1-2. 安裝 clasp
```shell
$ npm install -g @google/clasp@latest
```

### 1-3. 本機登入 clasp

這邊就會跳到登入 gmail 的地方，可以直接連接就好，連接成功後，會產生一個 `clasprc.json` 檔案
```shell
$ clasp login
```

### 1-4. 查看 clasprc.json 內容
```shell
$ cat ~/.clasprc.json

```
* 下面是我本機創建出來的 token，先把這一段複製下來
```json
{
  "tokens": {
    "default": {
      "client_id": "************",
      "client_secret": "************",
      "type": "************",
      "refresh_token": "************",
      "access_token": "*********"
    }
  }
}
```


## 2. 建立 檔案、Docker 環境
```md
APP-SCRIPT-CLASP
 | -- Dockerfile
 | -- clasprc.json
 | -- readme.md 
```


### 2-1. Dockerfile 內容
```dockerfile
# 使用官方 Node.js LTS
FROM node:18-slim

# 設定工作目錄
WORKDIR /app

# 安裝 clasp
RUN npm install -g @google/clasp@latest

COPY clasprc.json /root/.clasprc.json


# 預設進入 bash，方便操作
CMD [ "bash" ]
```

### 2-2. clasprc.json 內容

```json
{
  "tokens": {
    "default": {
      "client_id": "************",
      "client_secret": "************",
      "type": "************",
      "refresh_token": "************",
      "access_token": "*********"
    }
  }
}
```


## 3. 設定好檔案後，來建立 image
```shell
$ docker build -t clasp-env .
```


## 4. 接著設定 container (記得要掛載)
```shell
$ docker run -it -v /Users/yee0526/Desktop/TAGTOO/app-script-clasp:/app clasp-env

---
root@4bc8f6520598:/app# clasp list

formal lead program  - https://script.google.com/d/1TXOvee1NRELeekVrgHgnHYAwNTdwyeyIgsgFDf-dQ5C5gJlW2rsHewe1/edit
把匯率設定成 API           - https://script.google.com/d/1cS4dWtGQvaRKPulO7RTQFkeBtl4QDnOW_VVvBLYWV6uidJaL1QvSUtG0/edit
.....
```


## 5. 未來如果要繼續用 docker 開發，可以用這個指令

```shell
$ docker container exec -it confident_driscoll bash
```

ps. `confident_driscoll` 這個要看你的 container 名稱叫啥



## clasp 指令

### 1. clasp login

進到 container 容器內後，不需要再執行這個指令，因為 clasp login 在本機的時候執行過了，執行 + 登入後，會產生一個 `clasprc.json`，那個檔案現在已經在 container 裡面了，clasp 會根據此檔案決定你現在是連接哪個帳號



### 常用指令

```md
1. clasp clone：把雲端 Apps Script 專案拉下來
2. clasp push：把本地端程式碼推上雲端覆蓋
3. clasp pull：把雲端最新程式碼拉下來
4. clasp deploy：建立或管理一個版本（deployment）
5. clasp version：建立新版本
6. clasp list: 列出你帳號底下的專案
7. clasp pull: 拉下目前線上的版本
```

### 2. clasp list

此指令會列出你目前所有的 app script
```shell
$ root@4bc8f6520598:/app# clasp list
------
Found 8 scripts.
formal lead program  - https://script.google.com/d/1TXOvee1NRELeekVrgHgnHYAwNTdwyeyIgsgFDf-dQ5C5gJlW2rsHewe1/edit
把匯率設定成 API           - https://script.google.com/d/1cS4dWtGQvaRKPulO7RTQFkeBtl4QDnOW_VVvBLYWV6uidJaL1QvSUtG0/edit
把政府匯率存進 goog…        - https://script.google.com/d/1PtTCTXCoEefbCe_Qn9Bjf8T-NfuQC3Z052A09sysB82RKzxCgMdXUP2n/edit

```

### 3-1. clasp clone

* `clasp clone <scriptId>`
ps. list 中的網址連結 就是 `1PtTCTXCoEefbCe_Qn9Bjf8T-NfuQC3Z052A09sysB82RKzxCgMdXUP2n` 該 appScript 的 ID

```shell
$ clasp clone 1PtTCTXCoEefbCe_Qn9Bjf8T-NfuQC3Z052A09sysB82RKzxCgMdXUP2n
```

這樣輸入的時候，會在你的檔案中產生三個檔案

```md
- .clasp.json
- 程式碼 - 這個 app script 的程式
- appscript.json
```

#### clasp.json

這個檔案中的 `scriptId` 就是你剛剛輸入的 appScript ID
```json
{
  "scriptId": "1TXOvee1NRELeekVrgHgnHYAwNTdwyeyIgsgFDf-dQ5C5gJlW2rsHewe1",
  "rootDir": "",
  "scriptExtensions": [
    ".js",
    ".gs"
  ],
  "htmlExtensions": [
    ".html"
  ],
  "jsonExtensions": [
    ".json"
  ],
  "filePushOrder": [],
  "skipSubdirectories": false
}
```

#### appscript.json

```json
{
  "timeZone": "Asia/Taipei",
  "dependencies": {
  },
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8"
}
```


### 3-2. clasp pull

如果今天你本機端已經有 clone 過該檔案，只要輸入此指令，就可以把目前線上的檔案，同步到你這個資料夾

```shell
$ root@4bc8f6520598:/app/formal-lead-program# clasp pull

------
└─ appsscript.json
└─ 程式碼.js
```

### 4. clasp push

同步完後，就可以在本機端修改 `程式碼.js` 這個檔案，修改完後就可以把資料 push 上雲端

```shell
$ root@4bc8f6520598:/app/formal-lead-program# clasp push
------
Pushed 2 files.
└─ appsscript.json
└─ 程式碼.js
```



