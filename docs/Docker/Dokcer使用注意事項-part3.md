---
sidebar_position: 3
---


Docker 使用注意事項 - part3
------


## docker + python + poetry

今天如果要使用 python 和 poetry 的話，要怎麼設定 docker 和指令呢

### 先建立 dockerfile

```dockerfile
FROM python:3.9-slim

WORKDIR /srv


COPY projectTwo .
ENV POETRY_VIRTUALENVS_CREATE=0
RUN pip install -U poetry \
    && poetry install --only main \
    && rm -rf ~/.cache/pypoetry \
    && apt-get update \
    && apt-get install -y nano
```


### 建立 image
```shell
$ docker build -t two:latest -f project-two/Dockerfile . 
```

### 建立容器

```shell
$ docker run -it --name project-two -p 8080:80 two:latest
```


### 安裝 poetry
安裝 poetry 插件，一定要搭配 `pyproject.toml` 檔案，要不然他會說這一句話 - `Poetry could not find a pyproject.toml file in /srv or its parents`，因此我們來設定 `pyproject`


```toml
[tool.poetry]
name = "projectTwo"
version = "0.1.0"
description = "check product page status code, if status is not 200, it is considered expired"
authors = ["John <john0526@tagtoo.com>"]

[tool.poetry.dependencies]
python = "^3.9"
requests = "^2.31.0"
beautifulsoup4 = "^4.12.2"
django = "^3.2"

[tool.poetry.dev-dependencies]
ipython = "^8.16.1"

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
```


### 進到容器後，安裝 poetry

這個指令就像 npm install 差不多，可以把相關套件全部安裝好
```shell
test123: poetry install
```

### POETRY_VIRTUALENVS_CREATE

:::tip
如果今天你想要直接在 docker 裡面使用 poetry 安裝的 package，不想要進到 poetry shell 的環境，只要在 dockerfile 裡面多加這一段就可以 `ENV POETRY_VIRTUALENVS_CREATE=0`
這個指令的意思是: 這個環境變數告訴 Poetry 不要自動創建虛擬環境。當這個變數設置為 0 時，Poetry 不會為你的項目自動創建虛擬環境，而是使用全局環境。
:::

使用情境: 如果你希望在 Docker 容器中使用全局環境而不是虛擬環境，這是一個有用的設置。
多加這個指令，就可以當你進到 docker 檔案內後，輸入 poetry install 後，就可以直接使用輸入 pyproject.toml 內的 package


