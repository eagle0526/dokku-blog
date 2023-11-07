---
sidebar_position: 1
---

## 如何在 Django 中使用 tailwind 語法呢？
在本教學中，我會來一步一步的教學如何在 Django 中啟用 Tailwind 功能～


### 1. 使用 pip 來安裝 Django-tailwind 的 package

```shell
$ python -m pip install django-tailwind

------
Collecting django-tailwind
  Using cached django_tailwind-3.6.0-py3-none-any.whl (12 kB)
Requirement already satisfied: django>=3.2.14 in ./virtualenv/lib/python3.11/site-packages (from django-tailwind) (4.2.6)
Requirement already satisfied: asgiref<4,>=3.6.0 in ./virtualenv/lib/python3.11/site-packages (from django>=3.2.14->django-tailwind) (3.7.2)
Requirement already satisfied: sqlparse>=0.3.1 in ./virtualenv/lib/python3.11/site-packages (from django>=3.2.14->django-tailwind) (0.4.4)
Installing collected packages: django-tailwind
Successfully installed django-tailwind-3.6.0
```




### 2. 在 settings 啟用 tailwind 

```py
# settings

INSTALLED_APPS = [
  # other Django apps
  'tailwind',  
]
```


### 3. 在專案中新增 Tailwind CSS 的初始化資料夾

```shell
$ python manage.py tailwind init

------
Cookiecutter is not found, installing...
WARNING: pip is being invoked by an old script wrapper. This will fail in a future version of pip.
Please see https://github.com/pypa/pip/issues/5599 for advice on fixing the underlying issue.
To avoid this problem you can invoke Python with '-m pip' instead of running pip directly.
....省略一大段

 [1/1] app_name (theme): theme
Tailwind application 'theme' has been successfully created. Please add 'theme' to INSTALLED_APPS in settings.py, then run the following command to install Tailwind CSS dependencies: `python manage.py tailwind install`
```

輸入 `python manage.py tailwind init` 後，後面會要你填寫有關 `tailwind` 檔案的名稱，我這邊照著他的建議直接填上 `theme`



### 4. 在 settings 啟用剛剛新增的 theme
```py
# settings
INSTALLED_APPS = [
  # other Django apps
  'tailwind',
  'theme'
]
```


### 5. 註冊 tailwind 的名稱

在 settings 新增以下這一段
```py
# settings
TAILWIND_APP_NAME = 'theme'
```


### 6. 生成 tailwind 基礎文件

```shell
$ python3 manage.py tailwind install

------
added 135 packages, and audited 136 packages in 12s

30 packages are looking for funding
```



### 7. 查看剛剛生成的文件

剛剛 `install` 指令執行完後，可以在依照這個路徑 `your_tailwind_app_name/templates/base.html`，找到 tailwind 的範例文件，如果你已經有其他 `layout` 可以把這個基本文件刪掉，想要使用他的話也可以隨意擴展此基礎文件。



### 8. 如果你不想要使用剛剛的 base.html 文件當作 layout，可以這樣來觸發 tailwind


#### 8-1 我今天已經新增了一個自己的側邊欄檔案 `sidebar.html`，我希望把它當作我整個檔案的 `layout`
```html
<!-- online/templates/sidebar.html -->

{% load static tailwind_tags %}
<!doctype html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">  
  {% tailwind_css %}
</head>
<body>
  <h1 class="text-3xl font-bold underline">
    我是側邊欄位文件
  </h1>
  
  <div class="">
    {% block content %}
    {% endblock %}
  </div>

</body>
</html>
```

#### 8-2 我新增一個 `index.html` 頁面，並且要有側邊欄

```html
<!-- online/templates/index.html -->

{% extends 'sidebar.html' %}
{% block content %}
<h2 class="text-xl font-bold underline">
    12312321312
</h2>
{% endblock %}
```

### 9. 啟動 tailwind server

當想要設定的檔案都設定好後，就可以來啟動 `tailwind` 的 `server`

```shell
$ python3 manage.py tailwind start
```

啟動後輸入指定的連結，就可以看到你的 tailwind 成功被啟動了






### 10. 自動更新 tailwind 的 package

不過今天你想要更新 `tailwind` 語法的時候，希望網頁自動更新，吃到改到過後的 `tailwind`，你就會需要多安裝一個 `package`

```shell
$ python -m pip install django-browser-reload

------
Collecting django-browser-reload
  Using cached django_browser_reload-1.12.0-py3-none-any.whl (12 kB)
Requirement already satisfied: Django>=3.2 in ./virtualenv/lib/python3.11/site-packages (from django-browser-reload) (4.2.6)
Requirement already satisfied: asgiref<4,>=3.6.0 in ./virtualenv/lib/python3.11/site-packages (from Django>=3.2->django-browser-reload) (3.7.2)
Requirement already satisfied: sqlparse>=0.3.1 in ./virtualenv/lib/python3.11/site-packages (from Django>=3.2->django-browser-reload) (0.4.4)
Installing collected packages: django-browser-reload
Successfully installed django-browser-reload-1.12.0
```






參考連結：
1. https://django-tailwind.readthedocs.io/en/latest/installation.html
2. https://django-tailwind.readthedocs.io/en/latest/django_browser_reload.html

