---
sidebar_position: 1
---

0、前言
------
:::tip
**寫此篇文章的動機**  
研究如何用 fastAPI 做出一個 websocket 功能
:::



1、創建資料夾
------

這一次我們會使用 fastapi 搭配 docker 來建置一個 websocket 功能的簡易聊天室

```md
├── Dockerfile            # 放環境
├── pyproject.toml        # 放 package
├── server.py             # 放 websocket 基本建制
└── static        
    ├── client.js         # 放輸入框響應的 js
    └── index.html        # 放 index.html 畫面
```

參考連結：https://fastapi.tiangolo.com/advanced/websockets/#await-for-messages-and-send-messages




2、Dockerfile 內容
------

首先我們先來把 Dockerfile 內容給建好：

```Dockerfile
FROM python:3.9-slim

WORKDIR /srv

COPY pyproject.toml .

ENV PIP_DISABLE_PIP_VERSION_CHECK=1
ENV PIP_NO_CACHE_DIR=1
# for slim image
ENV POETRY_VIRTUALENVS_CREATE=0

COPY . .

CMD ["uvicorn", "server:app", "--host", "0.0.0.0", "--port", "8000"]
```




3、pyproject.toml 內容
------

```toml
[tool.poetry]
name = "fastAPI-websockets"
version = "0.1.0"
description = "fastAPI-websockets"
authors = ["eagle <john0526@tagtoo.com>"]

[tool.poetry.dependencies]
python = "^3.9"
fastapi = "0.104.1"
websockets = "14.1"
uvicorn = "^0.24.0"

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
```



4、server.py
------

前面把環境設定好，這邊我們就來連接 websocket:

```py
# 這邊先引進會用到的 package
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from fastapi.responses import HTMLResponse
from fastapi.staticfiles import StaticFiles
from typing import List

app = FastAPI()

# 掛載靜態文件到目錄
app.mount("/static", StaticFiles(directory="static"), name="static")

# 連接管理器：管理所有 websocket 連接
class ConnectionManager:
  def __init__(self):
    self.active_connection: List[WebSocket] = []        # 儲存所有活動連接

    async def connect(self, websocket: WebSocket):      # 處理新的連接
      await websocket.accept()
      self.active_connections.append(websocket)
    
    def disconnect(self, websocket: WebSocket):         # 處理連接斷開
      self.active_connections.remove(websocket)
    
    async def broadcast(self, message: str):            # 廣播消息給所有連接的人
      for connection in self.active_connection:
        await connection.send_text(message)


manage = ConnectionManager()

# 處理 root route, 返回 HTML 頁面
@app.get('/')
async def get():
  with open('static/index.html', 'r', encoding='utf-8') as f:
    html_content = f.read()
  return HTMLResponses(html_content)

# websocket 端點，處理及時通訊
@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
  await manage.connect(websocket)

  try:
    while True:
      data = await websocket.receive_text()
      await manager.broadcast(f"用戶發送: {data}")
  except WebsocketDisconnect:
    manager.disconnect(websocket)
    await manager.broadcast(f"有用戶離開了聊天室")
```



5、index.html
------

這邊來設定靜態頁面的資訊，這邊有幾個要注意：

1. sendMessage(event) 這個 function 是等等我們會在 client.js 建立的 func，而這個 event 是當我們送出表單時，會自動把表單的相關資訊送進這個 event 中

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>WebSocket 聊天室</title>
    <style>
        #messages { 
            list-style-type: none;
            margin: 0;
            padding: 0;
        }
        #messages li {
            padding: 0.5rem 1rem;
            background: #f8f9fa;
            margin: 0.5rem 0;
            border-radius: 4px;
        }
    </style>
</head>
<body>
    <h1>WebSocket 聊天室</h1>
    <form action="" onsubmit="sendMessage(event)">
      <input type="text" id="messageText" autocomplete="off">
      <button>發送</button>
    </form>
    <ul id="messages">
    </ul>
    <script src="/static/client.js"></script>
</body>
</html>
```


6、client.js
------

整體功能：
1. 連接管理
 - 自動檢測使用 ws:// 還是 wss:// 協議
 - 自動獲取當前主機地址
 - 處理連接斷開的情況並自動重連
2. 消息接收
 - 監聽服務器發來的消息
 - 將消息動態添加到頁面上
 - 使用 DOM 操作來更新頁面內容
3. 消息發送
 - 處理表單提交事件
 - 獲取用戶輸入的消息
 - 通過 WebSocket 發送到服務器
 - 清空輸入框
 - 阻止表單默認行為
4. 錯誤處理
 - 監控連接狀態
 - 在連接斷開時自動嘗試重連

```js
// 根據當前網頁的協議（http/https）決定使用 ws 還是 wss
const protocol = window.location.protocol === 'https:' ? 'wss:' : 'ws:'

// 構建 WebSocket 連接地址
const wsUrl = protocol + '//' + window.location.host + '/ws'

// 創建 WebSocket 連接
var ws = new WebSocket(wsUrl)

ws.onmessage = function(event) {
  // 獲取顯示消息的容器
  var messages = document.getElementById('messages')

  // 創建新的列表項
  var message = document.createElement('li')

  // 創建文本節點，內容是收到的消息
  var content = document.createTextNode(event.data)
  
  // 將文本添加到列表項中
  message.appendChild(content)

  // 將列表項添加到消息容器中
  messages.appendChild(message)
}

function sendMessage(event) {
  // 獲取輸入框元素
  var input = document.getElementById('messageText')

  // 發送輸入框中的內容
  ws.send(input.value)

   // 清空輸入框
  input.value = ''

  // 阻止表單默認提交行為
  event.preventDefault()
}

// 添加重連機制
ws.onclose = function(event) {
  console.log("連接已關閉，正在重新連接...");
  setTimeout(() => {
    ws = new WebSocket(wsUrl)
  }, 1000)
}
```

7、結論
------

### 整體工作流程：
1. 服務器端 (server.py):
 - 提供 Web 服務器功能
 - 管理 WebSocket 連接
 - 處理消息的廣播
 - 提供靜態文件服務
2. 靜態文件 (static 目錄):
 - index.html: 提供用戶界面
 - client.js: 處理前端的 WebSocket 連接和消息收發
3. 通訊流程:
 - 用戶訪問網站時，服務器返回 index.html
 - 瀏覽器加載 client.js 並建立 WebSocket 連接
 - 用戶發送消息時，通過 WebSocket 發送到服務器
 - 服務器收到消息後，廣播給所有連接的用戶
 - 所有用戶的瀏覽器接收消息並更新界面
 
這種架構實現了即時通訊的功能，允許多個用戶同時連接並互相發送消息。