---
title: tagtoo/cron-job-v2
sidebar_label: "10. [tagtoo] cron-job-v2"
description: cron-job-v2搬家
last_update:
  date: 2023-12-11
keywords:
  - Muffet
  - cron-job-v2
tags:
  - Muffet
sidebar_position: 10
---



## cron-job-v2

### 本地開發

以下都以 `check-expire-product` 檔案為例：

1. 建立指定資料夾

```md
<!-- cron-job-v2/src/cronjobs/check-expire-product -->
- cron-job-v2
 - src
  - cronjobs
   - check-expire-product   // 新增這個資料夾，之後檔案都放裡面
```

2. 新增 dockerfile、main.py、pyproject.toml

```console
<!-- cron-job-v2/src/cronjobs/check-expire-product -->
- cron-job-v2
 - src
  - cronjobs
   - check-expire-product   // 新增這個資料夾，之後檔案都放裡面
    - dockerfile
    - main.py
    - pyproject.toml
```

順便來說明一下這三個資料夾是做啥用的，`dockerfile` 就是等等會用到的環境、`main.py` 是主要運行的檔案、 `pyproject.toml` 則是因為我們主要使用 `poetry`，他是安裝插件的主要檔案

#### dockerfile 檔案內容

```dockerfile
FROM python:3.9-slim


ARG JOB_DIR=src/cronjobs/check-expire-product

WORKDIR /srv

COPY src/shared/ ./shared/
COPY $JOB_DIR/pyproject.toml .

ENV PIP_DISABLE_PIP_VERSION_CHECK=1
ENV PIP_NO_CACHE_DIR=1
# for slim image
ENV POETRY_VIRTUALENVS_CREATE=0

RUN pip install -U poetry \
    && poetry install --only main \
    && rm -rf ~/.cache/pypoetry

COPY $JOB_DIR .
```


#### pyproject.toml 檔案內容

```toml
[tool.poetry]
name = "checkout-expect-product"
version = "0.1.0"
description = "check product page status code, if status is not 200, it is considered expired"
authors = ["John <john0526@tagtoo.com>"]

[tool.poetry.dependencies]
python = "^3.9"
requests = "^2.31.0"
beautifulsoup4 = "^4.12.2"
cron-jobs-v2-shared = {path = "./shared/"}

[tool.poetry.dev-dependencies]
ipython = "^8.16.1"

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
```

#### main.py 這個檔案內容就是放你要執行的程式

我們就先簡單的設定出一個時間函式就好，等等會印出當前的時間

```py
from django.utils import timezone

print(timezone.now(), "==========")
```



3. 把 shared package 複製到當前的資料夾下

先使用 Makefile 指令 `move-shared-to-cron-job` 把 `shared` package 複製到指定的 Cron Job 目錄下，如此即可執行 Python interactive shell 或是執行 Python Script。
這個指令目的是，因為 `shared` 是 `howard` 已經整理好的所有可能會用到的資料，像是資料庫之類的，因此先把他移動到同層級的資料夾，等等我們使用 `docker bind` 的時候，才可以讓使用這些東西


```shell
# 把 shared 整個資料夾搬移到指定的資料夾底下
$ make cron_job_dir=$(pwd)/src/cronjobs/check-expire-product/ move-shared-to-cron-job
```


4. 建立 image 檔案

先用 docker build 建立 image

```shell
$ docker build -t check-expire-product:latest -f src/cronjobs/check-expire-product/Dockerfile .
```

再用 docker run + -v(bind) 連動整個資料夾，這樣的好處就是不需要每次修改 這個資料夾裡面檔案，就重新 build 一個新的 image

```shell
$ docker run -it --name check-expire-product-bind -v /Users/yee0526/Desktop/TAGTOO/cron-jobs-v2/cron-jobs-v2/src/cronjobs/check-expire-product:/srv check-expire-product:latest /bin/bash
```


5. 安裝 poetry 檔案

這時候我們已經在 docker 裡面了，所以我們在 docker 中使用 `安裝` 這個指令，等他跑完後，會生成 `poetry.lock` 這個檔案，就證明你在 docker 中的插件都安裝好了

```shell
root@c254923ebbbf:/srv# poetry install
```



6. 開始開發程式

```shell
root@c254923ebbbf:/srv# python main.py

---
印出當前時間
```




### 使用 docker 開發

如果今天已經確認檔案已經設定好了，不用再修改，那就不需要使用 docker bind 再進行開發，因此可以少掉搬移 `shared` 資料夾到當前路徑的步驟


1. 建立指定資料夾

```md
<!-- cron-job-v2/src/cronjobs/check-expire-product -->
- cron-job-v2
 - src
  - cronjobs
   - check-expire-product   // 新增這個資料夾，之後檔案都放裡面
```



2. 新增 dockerfile、main.py、pyproject.toml

```console
<!-- cron-job-v2/src/cronjobs/check-expire-product -->
- cron-job-v2
 - src
  - cronjobs
   - check-expire-product   // 新增這個資料夾，之後檔案都放裡面
    - dockerfile
    - main.py
    - pyproject.toml
```

順便來說明一下這三個資料夾是做啥用的，`dockerfile` 就是等等會用到的環境、`main.py` 是主要運行的檔案、 `pyproject.toml` 則是因為我們主要使用 `poetry`，他是安裝插件的主要檔案

#### dockerfile 檔案內容

```dockerfile
FROM python:3.9-slim


ARG JOB_DIR=src/cronjobs/check-expire-product

WORKDIR /srv

COPY src/shared/ ./shared/
COPY $JOB_DIR/pyproject.toml .

ENV PIP_DISABLE_PIP_VERSION_CHECK=1
ENV PIP_NO_CACHE_DIR=1
# for slim image
ENV POETRY_VIRTUALENVS_CREATE=0

RUN pip install -U poetry \
    && poetry install --only main \
    && rm -rf ~/.cache/pypoetry

COPY $JOB_DIR .
```


#### pyproject.toml 檔案內容

```toml
[tool.poetry]
name = "checkout-expect-product"
version = "0.1.0"
description = "check product page status code, if status is not 200, it is considered expired"
authors = ["John <john0526@tagtoo.com>"]

[tool.poetry.dependencies]
python = "^3.9"
requests = "^2.31.0"
beautifulsoup4 = "^4.12.2"
cron-jobs-v2-shared = {path = "./shared/"}

[tool.poetry.dev-dependencies]
ipython = "^8.16.1"

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
```

#### main.py 這個檔案內容就是放你要執行的程式

我們就先簡單的設定出一個時間函式就好，等等會印出當前的時間

```py
from django.utils import timezone

print(timezone.now(), "==========")
```


3. 建立 image 檔案

#### 先用 docker build 建立 image

```shell
$ docker build -t check-expire-product:latest -f src/cronjobs/check-expire-product/Dockerfile .
```

再用 docker run + -v(bind) 連動整個資料夾，這樣的好處就是不需要每次修改 這個資料夾裡面檔案，就重新 build 一個新的 image

#### 根據 image 檔案建立 container

```shell
$ docker run -it --name check-expire-product check-expire-product:latest /bin/bash
```

4. 直接執行 main.py 檔案

今天如果是這種情況，在上一個步驟執行完指令後，不用再執行這個指令 `poetry install`，因為跑 docker 時已經安裝好了，只要直接執行下面指令就好

```shell
root@c254923ebbbf:/srv# python main.py
```








