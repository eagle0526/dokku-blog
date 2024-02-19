---
sidebar_position: 1
---

0、前言
------
:::tip
**寫此篇文章的動機**  
練習用 fastAPI 來測試一些功能
:::


官方文件：https://fastapi.tiangolo.com/#example
參考文件：https://minglunwu.com/notes/2021/fast_api_note_1.html/


fastAPI 跟 flaskAPI 非常像，所以之前是用 `flask` 的人轉換陣痛不會很大，最大的優點是 fast 非常的快

## 安裝 fastAPI

```shell
$ pip install fastapi # FastAPI
$ pip install uvicorn[standard] # ASGI Server
```


## 撰寫 API

直接新增 main.py 檔案，等等把資料都新增在裡面，語法跟 flask 很像，也是透過 @app 裝飾的方式定義 API 路徑。
不過跟 flask 不一樣的是，FastAPI 在定義路徑時，會透過裝飾子一併定義 API 可提供的 HTTP Method:

* @app.post() : Post 方法
* @app.get() : Get 方法
* @app.put() : Put 方法
* @app.delete() : Delete 方法



### 把以下貼在檔案上

```py
# main.py

from typing import Optional

from fastapi import FastAPI

app = FastAPI() # 建立一個 Fast API application

@app.get("/") # 指定 api 路徑 (get方法)
def read_root():
    return {"Hello": "World"}


@app.get("/users/{user_id}") # 指定 api 路徑 (get方法)
def get_user(user_id: int, q: Optional[str] = None):
    return {"user_id": user_id, "q": q}
```



## 啟動服務

```shell
$ uvicorn main:app --reload
# uvicorn <檔名>:<app名稱>
# --reload 在專案更新時，自動重新載入，類似Flask的debug模式
```


### 輸入路徑

啟動伺服器後，可以輸入以下網址 `127.0.0.1:8000/users/2?q=player`，會得到以下的資訊
```md
{"user_id":2,"q":"player"}
```
當我們鍵入 `127.0.0.1:8000/users/2?q=player` FastAPI 會在定義好的裝飾子中，尋找符合路徑前綴: /users 的 function，也就是 get_user()。   
這部分的機制與 Flask 相當類似。  


:::tip
**flask 和 fast 的差異**  
FastAPI 在建立API時，需要以 型態提示 (Typing Hints) 來定義參數的型態
:::


## 型態提示(Typing Hints)

以上為例，我們前面在 `get_user` 定義以下格式：

```py
@app.get("/users/{user_id}") # 指定 api 路徑 (get方法)
def get_user(user_id: int, q: Optional[str] = None):
    return {"user_id": user_id, "q": q}
```

在定義 Function 的參數時，使用 Python 內建的型態檢查語法，指定各參數的型態，以上圖的Function來看，我們分別設定了下列的參數及型態:

1. user_id: int格式
2. q: str格式(Optional)

### 使用 Typing Hint 有兩項優點：


1. FastAPI 在收到請求時，會自動檢查參數的格式是否符合條件

當 FastAPI 收到 Request時，會先檢查URL，尋找符合 @app 所定義的路徑規則。

在解析 Request 所攜帶的參數時，將比對定義的變數型態，如果有不符合的，將會回傳錯誤訊息，舉例來說：

如果我們嘗試在瀏覽器中輸入 不符合規則的URL:

127.0.0.1:8000/users/error_type?q=player

(將原本的整數 2 換成字串 error_type)

此時因為收到的參數 error_type?q=player 因為參數 error_type 並不符合預先定義好的參數型態: int，此次Request無效。

不像 Flask 會回傳驚心動魄的錯誤訊息， FastAPI 會自動將錯誤訊息轉換成下列「有用」且「清楚」的錯誤訊息後，進行回傳:

```md
{"detail":[
    {"loc":["path","user_id"],"msg":"value is not a valid integer","type":"type_error.integer"}
    ]
}
```

2. 自動轉換文件

FastAPI 的另一項特色在於 - `自動生成 api 文件` : 完成了第一支 API 後，在瀏覽器輸入: `http://127.0.0.1:8000/docs:`
會看到 - 除了API的名稱外，參數的型態及Response的結果都包含在自動產生的文件中！




