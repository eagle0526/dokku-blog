---
sidebar_position: 4
---




Docker + fastAPI 應用 - part4
------

## 第三版的 dockerfile - 設定 port 

這一段的目的是，讓我們啟用 docker 的時候，可以自動啟用 host，也就是可以自動打開 `port`

```dockerfile
FROM python:3.9-slim

WORKDIR /srv

COPY pyproject.toml .

ENV PIP_DISABLE_PIP_VERSION_CHECK=1
ENV PIP_NO_CACHE_DIR=1
# for slim image
ENV POETRY_VIRTUALENVS_CREATE=0

RUN pip install -U poetry \
    && poetry install --only main \
    && rm -rf ~/.cache/pypoetry \
    && apt-get update \
    && apt-get install -y nano

COPY . .

CMD ["uvicorn", "DBoperator.app:app", "--host", "0.0.0.0", "--port", "8000"]  #=> 加上這一段
```


### 重新執行 docker build 指令

由於修改了檔案內容，因此需要重新 `build image, container`

```shell
$ docker build -t fastapi:latest .
$ docker run -it --name fastapi-docker -p 8000:8000 fastapi:latest
```

這樣執行後，可以在網址輸入 `http://127.0.0.1:8000/docs#`，應該會出現剛剛在 `app.py` 設定好的 `endpoint`。  
     
BUT!!!! 這個會有一個問題，就是這時候如果你打這個路徑 `http://127.0.0.1:8000/get_products?ec_id=2990`，會發生錯誤，內容是你不能連結 `docker - MySQL`，出現像這樣的訊息：  

```shell
# ...省略
packages/pymysql/connections.py", line 358, in __init__
    self.connect()
  File "/usr/local/lib/python3.9/site-packages/pymysql/connections.py", line 711, in connect
    raise exc

sqlalchemy.exc.OperationalError: (pymysql.err.OperationalError) (2003, "Can't connect to MySQL server on 'localhost' ([Errno 99] Cannot assign requested address)")
```
     
     
會有這個錯誤的原因，就是因為無法直接使用本機的資料庫，連上 `docker`，因此接下來要解決這件事情。          
