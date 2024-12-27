---
sidebar_position: 2
---

0、前言
------
:::tip
**寫此篇文章的動機**  
前面把基礎環境建置好了，我們現在來新增 Hello World 頁面
:::


## Hello World

### 進入 Docker

由於我們前面已經把 Docker 設定成 `hot reload` 的，之後我們都可以直接用 docker 開發就好

```shell
$ docker-compose exec web bash
```
ps. 這個 `web` 是我們在 `docker-compose.yml` 中定義的服務名稱


### 創建一個 main app
```shell
# 然後在容器內執行命令
root@947b1a4b194b:/srv# python manage.py startapp main
root@947b1a4b194b:/srv# python manage.py makemigrations
root@947b1a4b194b:/srv# python manage.py migrate
```
ps. 因為我們做了目錄掛載，所以在容器內創建的文件會直接反映到本地目錄中


### 資料夾架構

當我們剛剛輸入這行指令：`python manage.py startapp main`，會多產出一個 `main` 的資料夾：

```md
├── docker-compose.yml
├── Dockerfile
├── pyproject.toml
├── README.md
├── manage.py
└── main                    # 這一個
    └── migration
    └── __init.py__    
    └── apps.py
    └── models.py
    └── tests.py
    └── views.py
└── purchase_agent
    └── 省略
└── theme
    └── 省略
```

### settings 掛入 main 

記得新增一個 app，都一定要到 settings 那邊新增才行

```py
# purchase_agent/settings/base.py

# 忽略

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # 第三方
    'rest_framework',
    'corsheaders',
    'tailwind',

    # app
    'main',                           # 加上這個
]

# 忽略
```


### 修改 main 資料夾

這個 main 資料夾就是我們要顯示 hello world 的地方，所以我們首先來修改 `mian/views` 這個檔案

#### views.py

這個檔案掌管頁面顯示的邏輯

```py
from django.shortcuts import render
from django.http import HttpResponse
# Create your views here.
def hello_world(request):
    return render(request, 'hello.html')

```

#### main/templates/hello.html
`templates` 這個檔案，一開始是不存在的，需要自己手動加上去，架構會是這樣：

```md
├── docker-compose.yml
├── Dockerfile
├── pyproject.toml
├── README.md
├── manage.py
└── main
    └── migration
    └── templates               # 加上這個資料夾和裡面的檔案
        └── hello.html
    └── __init.py__    
    └── apps.py
    └── models.py
    └── tests.py
    └── views.py
└── purchase_agent
    └── 省略
└── theme
    └── 省略
```

這個檔案掌管頁面顯示的頁面：

```html
<!-- main/templates/hello.html -->

<!DOCTYPE html>
<html>
<head>
    <title>Hello World</title>
</head>
<body>
    <h1>Hello World!</h1>
    <p>Welcome to Purchase Agent</p>
</body>
</html>
```

### urls.py

這個檔案掌管頁面的連結：

```py
from django.contrib import admin
from django.urls import path
from main.views import hello_world

urlpatterns = [
    path('admin/', admin.site.urls),

    # hello world
    path('', hello_world, name='hello_world'),
]

```

這樣都設定好後，就可以用 `docker-compose up` 進到 `http://0.0.0.0:8000/`，會發現頁面已經變成我們在 `hello.html` 檔案設定的內容







