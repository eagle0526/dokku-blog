---
sidebar_position: 4
---

## fastAPI Dependencies 

在 FastAPI 中，Dependencies 是一種機制，可以讓您在路由處理函數之前或之後執行一些代碼。這些依賴可以用於從請求中提取數據、驗證用戶、授權訪問、訪問數據庫或其他任何您需要在執行路由處理函數之前或之後執行的任務。   
依賴可以是同步的或異步的，這取決於您的需求和實際情況。您可以使用同步函數、異步函數、其他 FastAPI 路由或其他自定義函數作為依賴。  
在 FastAPI 中，依賴是通過裝飾器 Depends 來定義的。您可以將 Depends 裝飾器應用於路由處理函數的參數上，以指示 FastAPI 在調用路由處理函數之前解析和執行該依賴。     

而依賴主要目的分成兩個部分，一個是 `前處理` 一個是 `後處理`，

* 前處理：在 route 操作函數前進行，用於預處理輸入數據、驗證請求
* 後處理：在 route 執行函數後進行，用於後處理數據，像是日誌記錄、數據清理之類的

以下是會給兩個例子，分別是前處理和後處理的用法：



### dependency injection

我們先定義一個預處理的依賴項函數，等等

```py
from fastAPI import Depends, FastAPI

app = FastAPI()

# 依賴項函數
def common_parameters(q: str = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}
```


### 前處理

我們在進入這個 `endpoint` 前，會先使用剛剛建立好的 `common_parameters` 函示，他會接受查詢參數 `q, skip, limit`，並返回一個包含這些參數的 `dict`。
這樣可以幫助清理一些數值，最後將其返回的結果傳遞給 `read_items` 函示，通過這樣的方式，可以在操作 `read_items` 函示的時候，通過傳入 `Depends(common_parameters)`，實現了路徑執行前的 `前處理輸入數據功能`

```py
from fastAPI import Depends

# 路徑操作函數
@app.get("/items/")
async def read_items(commons: dict = Depends(common_parameters)):
    return commons
```



### 後處理


現在在給一個 `後處理` 的範例，我們先寫一個 `後處理的 dependency injection`：

```py
async def process_after(request: str):
    print("Request processed", request)
    # 這裡裡可以執行一些後處理邏輯，比如記錄日誌、清理資料
    pass
```

再來定義一個路徑處理函數，並把剛剛的依賴項加進來
```py
@app.get("/process")
async def process_data(request: str, after_process: str = Depends(process_after)):
    return JSONResponse(content={"message": "Data processed successfully", "request": request})
```

在這邊，我們的 `process_after` 是一個後處理函示，他在 `process_data` 後執行，當你訪問 `/process` 的時候， `process_data` 函示處理完請求之後， `process_after` 這個後處理函示才會執行
Ps. 值得注意的是，後處理函數是通過 yield 來實現的，但是由於我們這裡沒有任何需要回傳的值，所以我們沒有在函數中使用 yield。如果您需要在後處理函數中回傳某些值，您可以使用 yield，並將值傳遞給後續處理函數




### 多個依賴項組合 - 前處理 + 後處理

我現在想要有一個比較複雜的 API，他會有以下幾個步驟：

1. 接受請求
2. 驗證用戶的 token
3. 將請求內容轉換成 json
4. 執行某些業務邏輯
5. 生成回應
6. 記錄 log


在這個 api 中， `驗證用戶 token 和 將請求轉換成 json 是前處理`，而 `執行某些業務邏輯 和 紀錄 log 是後處理`，因為他們是在處理請求之後執行，下面我們用 code 來示範：

```py
from fastapi import FastAPI, Depends
from fastapi.security import OAuth2PasswordBearer

app = FastAPI()
oauth2_scheme = OAuth2PasswordBearer(tokeUrl = "token")


# 前處理 Dependency - 驗證 token
async def authenticate_user(token: str = Depends(oauth2_scheme)):
    # 在這邊驗證 token，錯誤的話引發 HTTPException
    print("token authenticated")
    return token


# 前處理 Dependency - 將請求內容轉換成 JSON
async def parse_request_body():
    # 在這邊將請求內容轉換成 JSON
    print("Request body parsed")
    return {"data": "example"}


# 路徑處理函數，使用兩個前處理 Dependency
@app.get("/process")
async def process_data(token: str = Depends(authenticate_user), body_data = Depends(parse_request_body)):
    # 業務邏輯
    print("business logic executed")
    # 返回回應
    return {"message": "Data processed successfully"}


# 後處理 Dependency - 紀錄 log
async def log_request():
    # 在這邊處理 log 的執行
    print("request logged")


# 將後處理 Dependency 用於路徑處理函數
@app.get("/process", dependencies = [Depends(log_request)])
async def process_data_with_logging(token: str = Depends(authenticate_user), body_data = Depends(parse_request_body)):
    # 業務邏輯
    print("Business logic executed")

    # 返回回應
    return {"message": "Data processed successful"}
```


在上面的代碼中，`authenticate_user` 和 `parse_request_body` 是前處理依賴，它們在路由處理函數 `process_data` 開始執行之前執行。而 `log_request` 是後處理依賴，它在路由處理函數 `process_data_with_logging` 執行完畢後執行。   



### !!!!!! 後處理目前還是測試失敗，後處理的部分先不要參考 !!!!!!!




參考來源：
1. https://www.runoob.com/fastapi/fastapi-path-operation-dependencies.html
2. https://fastapi.tiangolo.com/tutorial/dependencies/


