---
sidebar_position: 1
---

0、前言
------
:::tip
**寫此篇文章的動機**  
研究如何用 fastAPI 做出一個 websocket 功能
:::



1、輸入指令
------

首先我們先起一個 npm 專案並安裝套件

```shell
$ npm init -y
$ npm install express
$ npm install ws
```
這幾個指令會把 `node_module` 和 `package.json` 和 `package-lock.json` 這三個資料夾創建好


2、建立檔案
------

* client.js 此檔案是要在後端建立好 websocket 的基本設定
* index.html 就是使用者看到的基本畫面
* client.js 是使用者輸入對話資料後，會觸發的 js 動作



```md
webSocket
  | client.js               # 新增
  | index.html              # 新增
  | server.js               # 新增
  | node_module
  | package.json
  | package-lock.json
```



3、server.js
------

首先我們來介紹一下這個資料夾的內容: 

1. 引入 express，並創建基礎的 HTTP 靜態文件
2. 引入 websocket，並把 websocket 建立在剛剛的 HTTP 上
3. 連接 websocket，並處理連接和需要發送的資訊

```js
// 引入 express 框架，為了等等建立 websocket server
const express = require('express')
// websocket 模組
const SocketServer = require('ws').Server
// 設定 port
const PORT = 3000
// 創建 express 應用實例
const app = express()

// 設定靜態文件
app.use(express.static(__dirname))

// 啟動 HTTP server
const server = app.listen(PORT, () => {
  console.log(`Listening on ${PORT}`)
})

// 創建 websocket, 並把他附加到剛剛創建的 server 上
const wss = new SocketServer({ server })

// 當有 websocket 連接時
wss.on('connection', ws => {
  console.log('Client connected')         // 有新客戶端連接時的提示

  // 監聽客戶端的 message
  ws.on('message', data => {
    console.log(data)                     // 這個 data 直接印出來，會像這樣 - <Buffer 32 32 32> 
    
    data = data.toString()
    console.log(data)                     // 這邊把它轉成字串，印出來像這樣 - 2221


    // 接著我們把資訊廣播給所有連接的客戶端，如果這一段拿掉，那這個聊天室就無法正常收到其他使用者發送的訊息
    let clients = wss.clients             // 獲取所有客戶
    clients.forEach((client) => {         
      client.send(data)                   // 將訊息發送給所有客戶
    })
  })

  ws.on('close', () => {
    console.log('Close connected')        // 客戶端斷開連接時的提示
  })
})
```


4、index.html
------

這個檔案就很基礎，只需要增加：
1. 一個呈現文字的區塊 - textarea
2. 一個輸入輸入匡 - input
1. 一個按鈕 - button
1. 一個 js 檔案 - client.js

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <textarea id="txtShow" disabled></textarea>
    <input id="txtInput" type="text">
    <button id="btnSend">送出</button>
    
    
    <script src="client.js"></script>    
</body>
</html>
```

4、client.js
------

```js
// 確保 DOM 都載入後，在運作以下資料
document.addEventListener('DOMContentLoaded', event => {
  // 抓三個 element
  let keyinDom = document.querySelector('#txtInput')
  let showDom = document.querySelector('#txtShow')
  let button = document.querySelector('#btnSend')

  // 點下按鈕後 
  button.addEventListener("click", () => {
    // 抓到 input 欄位目前的 value
    let txt = keyinDom.value
    // 通過 WebSocket 發送消息
    ws.send(txt)
  })

  // websocket 連接設定
  let url = 'ws://localhost:3000'
  var ws = new WebSocket(url)

  // websocket 事件處理
  ws.onopen = () => {
    console.log('open connection')
  }
  ws.onclose = () => {
    console.log('close connection');
  }

  // 處理收到的訊息 -> 收到服務器消息時觸發
  ws.onmessage = event => {
    let txt = event.data                  // 此為 ws 傳的內容
    // 將消息加到 textarea
    if (!showDom.value) {
      showDom.value = txt                       // 如果是第一條消息
    } else {
      showDom.value = showDom + "\n" + txt      // 如果已有其他消息，則換行添加
    }

    keyinDom.value = ""                         // 清空輸入框
    
  }
})
```

