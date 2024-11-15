---
sidebar_position: 2
---

0、前言
------
:::tip
**寫此篇文章的動機**  
前面一篇已經寫出基本的 websocket 應用，我們現在把它改造一下，讓聊天室可以改變姓名，看到其他人是怎麼對話的
:::


1、server.py
------

要讓留言可以看到姓名，那我們在一開始使用者進入聊天室時，就要可以儲存使用者的資訊，因此我們原先 `ConnectionManager init` 是 `List`，現在要改成 `dict`:

```py
# 前面一樣引入要用到的 package
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from fastapi.responses import HTMLResponse
from fastapi.staticfiles import StaticFiles
from typing import List, Dict
import os
import uuid
import json

app = FastAPI()

# 掛載靜態文件目錄
app.mount("/static", StaticFiles(directory="static"), name="static")

'''
修改 ConnectionManager
'''
class ConnectionManager:
  def __init__(self):
    self.active_connections: Dict[str, Websocket] = {}
    self.user_info: Dict[str, dict] = {}

    # connect 方法
    async def connect(self, websocket: Websocket):
      await websocket.accept()
      user_id = str(uuid.uuid4())

      # 例如：{'123e4567-e89b-12d3-a456-426614174000': <WebSocket連接對象>}
      self.active_connections[user_id] = websocket

      # 例如：{'123e4567-e89b-12d3-a456-426614174000': {'id': '123e4567-e89b-12d3-a456-426614174000', 'name': '用戶1'}}
      self.user_info[user_id] = {
        "id": user_id,
        "name": f"用戶{len(self.active_connections)}"
      }

      # 先將用戶添加到連接列表中，再廣播消息
      message_data = {
        "message": f"{self.user_info[user_id]['name']} 進入聊天室",
        "sender": "系統"
      }

      await self.broadcast(message_data)
      return user_id

      # disconnect 方法
      def disconnect(self, user_id: str):
        if user_id in self.active_connection:
          del self.active_connections[user_id]
        if user_id in self.user_info:
          del self.user_info[user_id]
      
      # broadcast 方法 - 把訊息傳給大家
      async def broadcast(self, message_data: dict):
        disconnected_users = []

        for user_id, connection in self.active_connections.items():
          try:
            await connection.send_text(json.dumps(message_data))
            except RuntimeError:
              # 如果發送失敗，將用戶添加到待刪除列表
              disconnected_users.append(user_id)
        
        for user_id in disconnected_users:
          self.disconnect(user_id)
      
      # update_user_name 方法
      async def update_user_name(self, user_id: str, new_name: str):
        if user_id in self.user_info:
          old_name = self.user_info[user_id]['name']
          self.user_info[user_id]['name'] = new_name

          message_data = {
            "message": f"{old_name} 更改名稱為 {new_name}",
            "sender": "系統"            
          }

          await self.broadcast(message_data)



'''
修改 websocket route 的設定
'''
manager = ConnectionManager()

@app.get('/')
async def get():
    # 讀取 index.html 文件
    with open('static/index.html', 'r', encoding='utf-8') as f:
        html_content = f.read()
    return HTMLResponse(html_content)


@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    # 這邊我們會得到 connect 方法最後傳出來的 user_id
    user_id = await manager.connect(websocket)
    try:
        while True:
            # receive_text() 接收到的是字符串            
            data = await websocket.receive_text()
            try:
                message_data = json.loads(data)
                if message_data.get("type") == "rename":
                    await manager.update_user_name(user_id, message_data["name"])
                else:
                    await manager.broadcast({
                        "message": message_data["message"],
                        "sender": manager.user_info[user_id]["name"]
                    })
            except json.JSONDecodeError:
                await manager.broadcast({
                    "message": data,
                    "sender": manager.user_info[user_id]["name"]
                })
    except WebSocketDisconnect:
        manager.disconnect(user_id)
        if user_id in manager.user_info:
            await manager.broadcast({
                "message": f"{manager.user_info[user_id]['name']} 離開了聊天室",
                "sender": "系統"
            })
  
```




### 1. 修改一開始創建 socket 的儲存格式 - active_connection 像這樣：
```py
self.active_connections: Dict[str, Websocket] = {}


{
    "user_id_1": WebSocket連接1,
    "user_id_2": WebSocket連接2,
    "user_id_3": WebSocket連接3,
    # ... 更多連接
}
```

### 2. user_info 像這樣：

```py
self.user_info: Dict[str, dict] = {}

{
    "user_id_1": {
        "id": "user_id_1",
        "name": "用戶1"
    },
    "user_id_2": {
        "id": "user_id_2",
        "name": "用戶2"
    },
    # ... 更多用戶信息
}
```


### 3. websocket.receive_text()


當前端像這樣在輸入框輸入資料：
```js
// 當用戶發送普通消息
ws.send(JSON.stringify({
    type: "message",
    message: "你好！"  // 用戶在輸入框輸入的內容
}))

// 或者當用戶更改名稱時
ws.send(JSON.stringify({
    type: "rename",
    name: "小明"      // 用戶輸入的新名稱
}))
```


後端這邊會使用 `websocket.receive_text` 來接收前端送來的資訊

```py
@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    user_id = await manager.connect(websocket)
    try:
        while True:
            # receive_text() 接收到的是字符串
            data = await websocket.receive_text()
            # 如果是普通消息，data 可能是：
            # '{"type": "message", "message": "你好！"}'
            
            # 如果是改名請求，data 可能是：
            # '{"type": "rename", "name": "小明"}'
           try:
                # 將接收到的 JSON 字符串轉換為 Python 字典
                message_data = json.loads(data)
                
                if message_data.get("type") == "rename":
                    # 處理改名請求
                    # message_data = {"type": "rename", "name": "小明"}
                    await manager.update_user_name(user_id, message_data["name"])
                else:
                    # 處理普通消息
                    # message_data = {"type": "message", "message": "你好！"}
                    await manager.broadcast({
                        "message": message_data["message"],
                        "sender": manager.user_info[user_id]["name"]
                    })
            except json.JSONDecodeError:
                # 如果不是 JSON 格式，直接當作普通文本處理
                await manager.broadcast({
                    "message": data,
                    "sender": manager.user_info[user_id]["name"]            
                })

```


2、index.html

* 跟之前相比，我們多做了一個 `changeName func`，來幫助我們修改使用者名稱

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>WebSocket 聊天室</title>
    <style>
        .message-container {
            margin: 10px 0;
            padding: 10px;
            border-radius: 5px;
            background: #f5f5f5;
        }
        .sender-name {
            font-weight: bold;
            color: #2196F3;
        }
    </style>
</head>
<body>
    <h1>WebSocket 聊天室</h1>
    <button onclick="changeName()">更改名稱</button>
    <form action="" onsubmit="sendMessage(event)">
        <input type="text" id="messageText" autocomplete="off">
        <button>發送</button>
    </form>
    <ul id="messages"></ul>
    <script src="/static/client.js"></script>
</body>
</html>
```

3、client.js
------

```js
// 動態獲取當前主機地址
const protocol = window.location.protocol === 'https:' ? 'wss:' : 'ws:';
const wsUrl = protocol + '//' + window.location.host + '/ws';

console.log("wsUrl", wsUrl)

var ws = new WebSocket(wsUrl);

ws.onmessage = function(event) {
    const data = JSON.parse(event.data);
    var messages = document.getElementById('messages')
    var message = document.createElement('li')
    // 顯示發送者和消息
    var content = document.createTextNode(`${data.sender}: ${data.message}`)
    message.appendChild(content)
    messages.appendChild(message)
}

function sendMessage(event) {
    var input = document.getElementById("messageText")
    // 發送JSON格式的消息
    ws.send(JSON.stringify({
        type: "message",
        message: input.value
    }))
    input.value = ''
    event.preventDefault()
}

// 添加改名功能
function changeName() {
    const newName = prompt("請輸入新的名稱：");
    if (newName) {
        ws.send(JSON.stringify({
            type: "rename",
            name: newName
        }));
    }
}

// 添加重連機制
ws.onclose = function(event) {
    console.log("連接已關閉，正在重新連接...");
    setTimeout(() => {
        ws = new WebSocket(wsUrl);
    }, 1000);
}
```