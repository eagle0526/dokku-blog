---
sidebar_position: 4
---

## part4 - 多個資料夾設定路徑，最後 import 入 main.py 資料夾統一觸發


## 1. 資料夾架構

我們在 part3 的時候，最後的資料庫樣式是這樣
```md
| FASTAPI
  |  -- sql_alchemy
        | -- __init__.py  
        | -- query.py
        | -- sql_alchemy.py
        | -- build_data.py
        | -- database.py
        | -- legacy_router.py
  |  -- virtualenv
  |  app.py
```

Ps. app.py 和 query.py 裡面都是空的


## 2. 在 sql_alchemy 資料夾裡面新增 main.py 檔案

我們等等會在 main.py 檔案裡面，把 legacy_router.py 建立好的 router，import 進 main.py 中


```md
| FASTAPI
  |  -- sql_alchemy
        | -- __init__.py  
        | -- query.py
        | -- sql_alchemy.py
        | -- build_data.py
        | -- database.py
        | -- legacy_router.py
        | -- main.py             => 新增這個檔案
  |  -- virtualenv
  |  app.py
```


### 2-1 - 先修改 legacy_router.py 檔案


由於我們前面是直接用 legacy_router.py 這個檔案開啟 server，現在我們要改用 main.py 開啟 server，所以要先來修改這個檔案

```py
from database import SessionLocal
from fastapi import FastAPI, APIRouter, status, Depends
from sqlalchemy.orm import Session

from pprint import pprint
from sql_alchemy import Website

router = APIRouter(
    prefix='/legacy',
    tags=["legacy"]
)

# Dependency
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()


@router.get("/get_products", status_code=status.HTTP_200_OK)
async def get_products(id, db: Session = Depends(get_db)):
    results = (
        db.query(Website)
        .filter(
            Website.id == id
        ).all()
    )

    print("********** results", results)
    return {"status_code": status.HTTP_200_OK, "products": results}
```


ps. 這邊拿掉 `app = FastAPI()` 和 `app.include_router(router)`，這兩行等等會在 `main.py` 這邊補上


### 2-2 - main.py 檔案

```py
# main.py

from fastapi import FastAPI, Request, status, Depends
import uvicorn
from database import SessionLocal

# 最重要的就這段
from legacy_router import router as legacy_router


app = FastAPI()

# Depends
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()


@app.get("/")
async def root():
    return {"message": "Database operator"}


app.include_router(legacy_router)

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)

```

### 2-3 - 啟動 server

先確認自己在現在的位置
```shell
$ pwd

======
/Users/yee0526/Desktop/python/fastAPI
```


確認後，就可以直接輸檔案指令
```shell
$ python sql_alchemy/main.py

======
INFO:     Started server process [91223]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

接下來輸入網址就可以看到你想要的資料了！












