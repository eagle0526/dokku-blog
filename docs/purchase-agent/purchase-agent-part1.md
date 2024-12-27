---
sidebar_position: 1
---

0、前言
------
:::tip
**寫此篇文章的動機**  
研究如何用 django 做出一個 `代購網`，這個專案，我們主要用 `djangoREST` + `poetry` + `postgreSQL` + `docker` + `tailwind` 來做出一個網站
:::



## 基礎環境建置

### django init

1. 新增資料夾
```shell
$ mkdir 代購網
$ cd 代購網
```

2. django init
```shell
$ django-admin startproject purchase_gent .
```


3. create docker and pyproject
```shell
$ touch Dockerfile
$ touch pyproject.toml
```

4. 這樣來看一下目前整體資料夾結構
```md
├── Dockerfile            # 放環境
├── pyproject.toml        # 放 package
├── README.md
├── manage.py
└── purchase_agent        
    ├── __init__.py
    └── asgi.py
    └── settings.py
    └── urls.py
    └── wsgi.py            
```

### pyproject.toml

這個是使用 `poetry` 會用到的文件，類似於 `pip` 的 `requirements`，用來放 `package` 的地方，我們來放上會用到的 package

```toml
[tool.poetry]
name = "purchase-agent-website"
version = "0.1.0"
description = "purchase-agent-website"
authors = ["eagle <a034506618@gmail.com>"]

[tool.poetry.dependencies]
python = ">=3.10,<4.0" 
Django = "5.1.3"
djangorestframework = "3.14.0"
django-tailwind = "3.8.0"
python-dotenv = "^1.0.0"
django-cors-headers = "^4.3.1"
psycopg2-binary = "^2.9.9"

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
```


### Dockerfile

這個專案我們要使用 Docker 來管理，因此建置一個環境，這邊主要就是：
1. 使用 python 3.10
2. 讀取 pyproject.toml
3. 設立 poertry 會用到的環境變數
4. 執行基本指令
5. 最後打開 PORT 並且在本地開發

 
```dockerfile
FROM python:3.10-slim

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

# port
EXPOSE 8000

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

### docker-compose.yaml
#### 新增資料夾

```md
├── docker-compose.yml
├── Dockerfile
├── pyproject.toml
├── README.md
├── manage.py
└── purchase_agent        
    ├── __init__.py
    └── asgi.py
    └── settings.py
    └── urls.py
    └── wsgi.py            
```

#### 檔案內容
再來我們來新增 `docker-compose.yaml`，一開始可能會有疑問，明明有 `Dockerfile` 了，為什麼還要有 `docker-compose`，原因就是 `Dockerfile` 只是幫你建置 `image`，但是如果想要順便把 `多個容器` 和 `DB` 建立好，就會需要有 `docker-compose.yaml` 來幫你統整。     

順便說一下這個 `yaml` 我們做了哪些事情：

1. 建置 container 名稱 - `purchase-agent-web`
2. 在容器內執行兩個指令 - `migrate` 、 `runserver`
3. 設定 `volumes`、`ports`、`environment`
4. 設定 `DB` 的內容，包含用哪個 DB、密碼

```yaml
version: '3.8'

services:
  web:
    build: .
    container_name: purchase-agent-web
    command: >
      bash -c "python manage.py migrate &&
               python manage.py runserver 0.0.0.0:8000"
    volumes:
      - .:/srv
    ports:
      - "8000:8000"
    environment:
      - DJANGO_SETTINGS_MODULE=purchase_agent.settings.production      
      - POSTGRES_DB=purchase_agent
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_HOST=db
      - ALLOWED_HOSTS=localhost,127.0.0.1
    depends_on:
      - db

  db:
    image: postgres:15
    container_name: purchase-agent-db
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=purchase_agent
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres

volumes:
  postgres_data:
```

這樣設定好之後，當我們等等執行 `docker-compose up` 就會直接幫我們把 `image、container、volume` 一起建立起來，並且名字都是你寫好的


### 修改 settings 檔案內容

由於我們有安裝幾個想要用的 package，因此我們要在 settings 裡面啟用他們

```py
# settings
from pathlib import Path
import os
from dotenv import load_dotenv

load_dotenv()

# 忽略....

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # 加上下面這三個
    'rest_framework',
    'corsheaders',
    'tailwind',
]

# 先加上這一行，等等運行 python manage.py tailwind init 來創建資料夾
TAILWIND_APP_NAME = 'theme' 

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',                   # 加上這一行

    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

# 忽略


# 加上這一行，本地開發用
CORS_ALLOWED_ORTINGS = [
    "http://localhost:8000",
]


# 加上 REST 基本設定
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated'
    ],
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.BasicAuthentication',
    ]
}

# 忽略
```

### poetry shell + python manage.py runserver

現在我們來進到 poetry 的環境裡面，再執行 runserver 進入 Django 的本地端，如果想要知道 poetry 更多細節，可以到這邊: `https://blog.kyomind.tw/poetry-pyenv-practical-tips/`。

因此現在來執行下面兩個指令：

```shell
$ poetry show                       # 先進到 poetry 環境
$ python manage.py runserver        # 再執行 runserver
```

Ps. 如果今天沒進到 poetry 環境裡面就執行 runserver，正常來說會噴無法讀取到一些 package 的錯誤，因為目前我們的 package 是用 pyproject 來設定，因此是被裝在 poetry 裡面

### 創建 tailwind init

這個是 tailwind 的基本檔案建置
```shell
$ python manage.py tailwind init
```

這個指令執行後，會創建多個 tailwind 相關資料夾

```md
├── docker-compose.yml
├── Dockerfile
├── pyproject.toml
├── README.md
├── manage.py
└── purchase_agent        
    ├── __init__.py
    └── asgi.py
    └── settings.py
    └── urls.py
    └── wsgi.py           
└── theme                   # 這個資料夾會自動生成
    └── static_src
        └── src
            └── styles.css
        └── package.json
        └── postcss.config.js
        └── tailwind.config.js                               
    └── template
        └── base.html
    └── __init__.py
    └── apps.py
```

### 把 本地 和 Docker 使用的資料庫給拆開

由於我們想要在本地端開發，而且又要跟 Docker 的拆開，因此我們來把原本放在 settings 裡面的資料庫設定，拆分開來

#### 新的資料夾架構

* 刪掉原本的 settings.py
* 新增一個 settings 資料夾
* 新增以下幾個檔案 `__init__.py`, `base.py`, `local.py`, `production.py`
* base.py 就是原本 `settings.py` 檔案的內容，除了 `Database` 設定
* local.py 就是你在本地端想要測試的資料庫
* production.py 就是要在 Docker 的資料庫設定

```md
├── docker-compose.yml
├── Dockerfile
├── pyproject.toml
├── README.md
├── manage.py
└── purchase_agent        
    ├── __init__.py
    └── asgi.py
    └── urls.py
    └── wsgi.py
    └── settings          # 新增這四個
        └── __init__.py
        └── base.py
        └── local.py
        └── production.py
└── theme
    └── 省略
```

#### local.py 內容

* 這個資料庫要注意，你要先在本地端新建一個 `purchase_agent` 名稱的 `postgres` database，這一段才能正常運作，如果你本機沒有這個 DB，他會噴錯
* 想要知道如何建立，可以看這一篇 `https://eagle0526.netlify.app/docs/SQL/PSQL%E8%AA%9E%E6%B3%95`
```py
from .base import *

DEBUG = True

ALLOWED_HOSTS = ['localhost', '127.0.0.1']

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'purchase_agent',
        'USER': 'postgres',
        'PASSWORD': 'postgres',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

#### production.py 內容

* 這裡可以看到我們是使用環境變數，當我們在執行 docker-compose 的時候，就已經把這些環境變數設定好了，因此直接執行就可以連接到 DB

```py
from .base import *

DEBUG = True  # 暫時改為 True 來查看詳細錯誤信息

ALLOWED_HOSTS = os.getenv('ALLOWED_HOSTS', '').split(',')

# 添加日誌配置
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
        },
    },
    'root': {
        'handlers': ['console'],
        'level': 'INFO',
    },
}

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.getenv('POSTGRES_DB'),
        'USER': os.getenv('POSTGRES_USER'),
        'PASSWORD': os.getenv('POSTGRES_PASSWORD'),
        'HOST': os.getenv('POSTGRES_HOST', 'db'),
        'PORT': os.getenv('POSTGRES_PORT', 5432),
    }
}
```

### 本地執行 migrate

接下來就到最後一步了，完成這一步我們就可以把基本環境設定好，就是把 Django 預設的資料給送進剛剛創建好的本地端資料庫，記得因為我們是用 poetry，所以要先進 poetry 環境，最後再執行 `runserver`，就可以開始本地端開發了。

```shell
$ poetry shell
$ python manage.py migrate
$ python manage.py runserver
```



### hot-reload Docker container

順便補充一個，就是很多人應該會只想純粹用 Docker 來開發，但是又會遇到你在本機檔案修改，無法馬上反映在 Docker 裡面，現在我們就來修改一下，讓我們可以在本地端啟用 Docker 之後，當我們修改檔案時，會直接反映到 Docker Container 裡面。

Ps. 也就是其實也根本不需要 poetry 在本地端開發，直接用 Docker 開發就好了，我前面多分出 `local` 的確是多此一舉

```yaml
# docker-compose.yml

version: '3.8'

services:
  web:
    # 省略
    volumes:
      - .:/srv  # 掛載當前目錄到容器的 /srv
      - python_packages:/usr/local/lib/python3.10/site-packages/  # 緩存 Python 包 - 新增
    ports:
      - "8000:8000"
    environment:
      - DJANGO_SETTINGS_MODULE=purchase_agent.settings.production      
      - POSTGRES_DB=purchase_agent
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_HOST=db
      - ALLOWED_HOSTS=localhost,127.0.0.1,0.0.0.0
      - PYTHONDONTWRITEBYTECODE=1  # 防止生成 .pyc 文件 - 新增
      - PYTHONUNBUFFERED=1  # 實時輸出日誌 - 新增
    depends_on:
      - db
    # 添加重啟策略
    restart: unless-stopped  #- 新增

  db:
    image: postgres:15
    container_name: purchase-agent-db
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=purchase_agent
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    # 添加重啟策略
    restart: unless-stopped   #- 新增

volumes:
  postgres_data:
  python_packages:  # 新增 volume 用於緩存 Python 包
```



