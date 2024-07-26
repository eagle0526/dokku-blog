---
sidebar_position: 1
---




Docker + fastAPI 應用 - part1
------

## 初始架構

首先我們先在空的資料夾中，建立兩個兩個檔案：

```md
| FASTAPI_Docker
  |  Dockerfile
  |  main.py
```


## 第一版 dockerfile - init dockerfile


### dockerfile 內容

```dockerfile
# Dockerfile

FROM python:3.9-slim
```

一開始只先設定這樣，先看一下等我們 build image 後，裡面的內容會長什麼樣子：

### 建立 image + container

#### image
以下這個 `.` 表示去找當前路徑最接近的 Dockerfile，建立他的 image 檔案，假設今天你的整個檔案今天有很多個 `Dockerfile`，建議使用 `-f` 這個指令來讓系統知道你想要建立的 `Dockerfile` 是哪一個

```shell
$ docker build -t fastapi:latest .
```

#### container
* name 是幫現在建立的 container 命名
* fastapi:latest - 是指你要吃的 image 檔案名稱
* bash 是指當你把容器建立好後，要不要直接進入 bash 環境

```shell
$ docker run -it --name fastapi-docker fastapi:latest bash

-----
root@9065ad0857ce:/# ls
bin  boot  dev  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

root@9065ad0857ce:/# exit
```

:::tip
這邊可以特別說一下，進到剛剛建立好的 image 後，會發現裡面有很多個資料夾，這些資料夾就是初始你建立 docker 時會有的
:::

#### exec 進入容器

如果今天離開了容器，又想進入後，可以輸入以下指令，再進入 bash 環境一次，記得要先啟動 container 才能進入喔

```shell
$ docker container exec --interactive --tty fastapi-docker bash
```



### 新增 pyproject.toml 檔案

新增 `pyproject` 檔案，這個檔案的作用就是下載 `package` 用的，就是 `pip 的 requirement`：

```md
| FASTAPI_Docker
  |  Dockerfile
  |  pyproject.toml  
  |  main.py
```

再來補上裡面的內容，順面介紹一下裡面的 package 都是做啥用的：  
   
* python: python 的版本
* fastapi: fastapi 的版本
* SQLAlchemy: 可以讓 python 用於關聯式資料庫進行互動 (使用 ORM)
* alembic: 資料庫 migration 的工具，幫忙做版本控制
* pydantic: Pydantic 主要是拿來做資料的驗證與設定，可幫你驗證資料的 data type ，及是否符合規則 (像是對應欄位是否為 emil)。
* PyMySQL: 讓 python 可以連接資料庫
* uvicorn: 這是一個 ASGI 服務器，用於運行 FastAPI 應用

```toml
[tool.poetry]
name = "fastAPI"
version = "0.1.0"
description = "fastAPI"
authors = ["eagle <john0526@tagtoo.com>"]

[tool.poetry.dependencies]
python = "^3.9"
fastapi = "0.104.1"
SQLAlchemy = "2.0.30"
alembic = "1.13.1"
pydantic = "2.5.2"
PyMySQL = "1.1.0"
uvicorn = "0.23.2"

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
```

Ps. 記住 name, version 那些都要填寫，要不然我 build 不起來