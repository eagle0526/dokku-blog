---
sidebar_position: 2
---




Docker + fastAPI 應用 - part2
------

## 第二版的 dockerfile - 安裝 package

前面已經新增好 `pyproject 檔案後`，接下來 `docker` 這樣修改，我會詳細描述每一步驟都在做啥：

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
```

#### FROM
* 首先先把 python:3.9 版本給拉下來

#### WORKDIR
* 接著我們直接指定我們要在哪個資料夾運行 (還記得我們前面有把 docker 檔案全部印出來，其中就有 /srv)

#### COPY
* 把 `pyproject.toml` 這個檔案複製一份進到 docker 裡面 (COPY 左邊參數是本地端檔案位置，右邊參數是 dockerfile 的檔案位置)

#### ENV
* 再來設定一些環境變數，要設定變數的原因，可以到 `tagtoo/cron-job-v2` 這個檔案位置看原因，就不再說一次

#### RUN
* 安裝 `poetry` 套件，這個套件是類似 pip，協助進行 package 管理，不過功能比 pip 還強大很多
* 相較於 `pip 的 requirement`， poetry 的就是 `pyproject.toml`，因此 `poetry install` 就是在安裝 `pyproject` 裡面寫好的 package
* rm -rf : 清理 pypoetry 裡面的緩存文件
* apt-get update: 確保在運行 `apt-get install` 命令之前，系統知道最新的 package version。這樣可以避免安裝過時或有潛在安全問題的 package。
* 安裝 nano 這個 package

#### COPY
* 最後在補一個 `COPY . .`，把整份檔案複製進 docker 裡面



### 重新執行 docker build 指令

記得要把前一個 image 給刪掉，前一個只是測試是否能 build image 測試用的：

* 建立好 image, container 後，進入 container 裡面，可以看到裡面有我們本地端所有的檔案

```shell
$ docker build -t fastapi:latest .
$ docker run -it --name fastapi-docker fastapi:latest bash

------
root@c49805e823d0:/srv# ls
Dockerfile  db-oprator  main.py  poetry.lock  pyproject.toml

```

* 並且可以輸入 `poetry show` 指令，查看目前有哪些 package 已經被 poetry 裝好
```shell
root@ 49805e823d0:/srv# poetry show
-----
alembic           1.13.1  A database migration tool for SQLAlchemy.
annotated-types   0.7.0   Reusable constraint types to use with typing.Annotated
anyio             3.7.1   High level compatibility layer for multiple asyn
...
...

```

PS. 這邊要特別注意，現在這種建立容器的方法，會讓你更新檔案後，就要重 build 一次，如果不想要每次都重 build，可以使用 `bind` 的方法