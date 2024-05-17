---
sidebar_position: 4
---

## fastAPI setting 

在 FastAPI 中，基本路徑是定義 API 端點的關鍵。 每個路徑都會對應到應用程式中的函數，用於處理特定的 HTTP 請求，並傳回對應的回應。

參考範例：https://hackmd.io/@YMont/python-fastapi-4



### fast API - APIRoute()

今天來介紹 `APIRoute()`，今天來用這個路徑，來做出一個有多個分頁的網站路徑，假設現在我有一個這個 `催眠網站 - https://er.sleeps.app/`，而他的相關分頁有以下幾個：

```md
https://er.sleeps.app/         -> home
https://er.sleeps.app/about    -> about
https://er.sleeps.app/price    -> price
```

分別是一個首頁，另外兩個一個是關於我們、價錢相關的頁面，不過以上有點太簡單，通常還會再往下一層多件分頁，像是以下：

```md
https://er.sleeps.app/                  -> home
https://er.sleeps.app/about             -> about
https://er.sleeps.app/about/teacher1
https://er.sleeps.app/about/teacher2    
https://er.sleeps.app/price             -> price
https://er.sleeps.app/price/service1
https://er.sleeps.app/price/service2
```

像是這樣的分頁架構，確認好我們應該會有這幾個分頁後，我們來實作資料夾的架構吧！

### 實際操作


### 資料夾架構
```md
|--app
    |-- __init__.py
    |-- main.py
    |-- 
    |-- \route 1
    |--      |-- __init__.py
    |--      |-- page1.py
    |-- 
    |-- \route 2
    |--      |-- __init__.py
    |--      |-- page1.py

```

### URL 架構

可以把上面的資料夾架構轉換成 URL 架構比較好懂
```md
|--127.0.0.1:8001/
    |-- 
    |-- 127.0.0.1:8001/router1
    |--      |-- 127.0.0.1:8001/router1/page1
    |-- 
    |-- 127.0.0.1:8001/router2
    |--      |-- 127.0.0.1:8001/router2/page1
```


不過今天只是要介紹如何使用 APIRouter，所以只會做到第二層，也就是像以下：

```md
|--127.0.0.1:8001/
    |-- 127.0.0.1:8001/router1
    |-- 127.0.0.1:8001/router2    
```


### router 1

我們現在來建立這個 `endpoint` 的資料 - `127.0.0.1:8001/router1`
```py
# router1.py

from fastapi import FastAPI, APIRouter, HTTPException

router = APIRouter()

@router.get("/router1")
def router1() -> dict:
    return {"msg": "Hello Router1"}
```


### router 2
再來建立這個 `endpoint` 的資料 - `127.0.0.1:8001/router2`


```py
# router1.py
from fastapi import FastAPI, APIRouter, HTTPException

router = APIRouter()
@router.get("/router2")
def router2() -> dict:
    return {"msg":"Hello Router2"}
```

### main.py 資料夾

剛剛把兩個分頁的路徑建好了，現在完成 `127.0.0.1:8001` 的資料：

```py
from fastapi import FastAPI, APIRouter, HTTPException
# import 剛剛建好的兩個路徑
from router1 import router1
from router2 import router2

app = FastAPI()            # 以app作為FastAPI實例
api_router = APIRouter()   # 以api_router作為APIRouter實例，本次重點!

# 重點來了
api_router.include_router(router1.router)   # 把router1檔案裡的路由結合進api_router
api_router.include_router(router2.router)   # 把router2檔案裡的路由結合進api_router


# 設定 root 的 endpoint
@api_router.get("/")
def root() -> dict:
    """
    Root Get
    """
    return {"msg": "Hello root"}

# 把 app 實例融合進剛剛的 api_router 裏面
app.include_router(api_router)


if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="127.0.0.1:8001", port=8001)
```


最後你依照以下三個路徑，會分別看到你想要的資料：
```md
127.0.0.1:8001
127.0.0.1:8001/router1
127.0.0.1:8001/router2
```


### 進階應用

如果今天這樣設定 : 
```py
from typing import Optional
from fastapi import FastAPI, APIRouter
from pydantic import BaseModel

app = FastAPI() # 建立一個 Fast API application
router = APIRouter()

router = APIRouter(
    prefix="/team",
    tags=["team"]
)

@router.get("/users", tags=["users"])
async def read_users():
    return [{"username": "Rick"}, {"username": "Morty"}]
```

那你的路徑就會變成這樣: `http://127.0.0.1:8000/team/users`，簡單的說，就是 router 會先去吃一開始設定的 `router` 路徑，再來才是吃 `@router.get` 設定的路徑，如果今天把這一段拿掉：

```py
from typing import Optional
from fastapi import FastAPI, APIRouter
from pydantic import BaseModel

app = FastAPI() # 建立一個 Fast API application
router = APIRouter()

# 拿掉
# router = APIRouter(
#     prefix="/team",
#     tags=["team"]
# )

@router.get("/users", tags=["users"])
async def read_users():
    return [{"username": "Rick"}, {"username": "Morty"}]
```

那路徑就會變回這樣: `http://127.0.0.1:8000/users`


### 參數介紹
這邊直接介紹有幾個常用的參數：   
     
* prefix - 這個就是路徑的字串    
* tag - 如果今天有加上這個參數，在 `http://127.0.0.1:8000/docs#/` 這個 api 輔助介面的地方，會有很明顯的標籤顯示在該路徑那邊 


不過由於參數有點多，其他的直接點這個連結進去看就好：     
參考連結：https://fastapi.tiangolo.com/reference/apirouter/













