---
title: tagtoo/dashboard-api
sidebar_label: "4. [tagtoo] dashboard-api-server"
description: dashboard api server 是關於 GMC feed 產出邏輯的檔案
last_update:
  date: 2023-11-08
keywords:
  - Muffet
  - dashboard-api-server  
tags:
  - Muffet
sidebar_position: 5
---


## 本機開發 - 流程大綱

1. git clone https://github.com/Tagtoo/dashboard-api.git
2. git submodule init
3. git submodule update
4. 新增兩個環境變數 - `export DOCKER_BUILDKIT=0` 和 `export COMPOSE_DOCKER_BUILD=0`
5. docker pull asia.gcr.io/dashboard-dev-1230/dashboard-api-base:latest  - (要請 howard 或 patrick 開權限給你)
6. docker image tag asia.gcr.io/dashboard-dev-1230/dashboard-api-base:latest dashboard-api-base:latest
7. 到 test.Dockerfile 的資料夾，把 `CMD python manage.py runserver 0:8000` 上面四行程式碼刪掉 - 下面有詳細的 code
8. 編輯 test_setting.py - 把 test 吃的資料庫改成雲端資料庫
9. make test-up


## 本機開發 - 詳細步驟

### 步驟一：先把 api 的repo clone 下來
```shell
$ git clone https://github.com/Tagtoo/dashboard-api.git
```

### 步驟二：執行初始化指令

```shell
$ git submodule init

------
子模組 'SlackReport'（git@github.com:Tagtoo/SlackReport.git）已對路徑 'SlackReport' 註冊
子模組 'lib/submodule/bigquery-logger'（git@github.com:Tagtoo/python-bigquery-logger.git）已對路徑 'lib/submodules/bigquery-logger' 註冊
子模組 'lib/submodule/cache-image'（git@github.com:Tagtoo/cache-image.git）已對路徑 'lib/submodules/cache-image' 註冊
子模組 'lib/submodule/django-gcloud-storage'（git@github.com:Tagtoo/django-gcloud-storage.git）已對路徑 'lib/submodules/django-gcloud-storage' 註冊
子模組 'lib/submodule/ec'（git@github.com:Tagtoo/ec.git）已對路徑 'lib/submodules/ec' 註冊
子模組 'lib/submodule/gcloudorm'（git@github.com:Tagtoo/gcloud-python-orm.git）已對路徑 'lib/submodules/gcloudorm' 註冊
子模組 'lib/submodule/json-extend'（git@github.com:Tagtoo/json-extend.git）已對路徑 'lib/submodules/json-extend' 註冊
子模組 'lib/submodule/ndb-utils'（git@github.com:Tagtoo/ndb-utils.git）已對路徑 'lib/submodules/ndb-utils' 註冊
子模組 'lib/submodule/pyslack'（git@github.com:Tagtoo/pyslack.git）已對路徑 'lib/submodules/pyslack' 註冊
子模組 'node_modules/Tagtoo_tools'（git@github.com:Tagtoo/Tagtoo_tools.git）已對路徑 'node_modules/Tagtoo_tools' 註冊
子模組 'tagtoo-config'（git@github.com:Tagtoo/tagtoo-config.git）已對路徑 'tagtoo-config' 註冊
```

### 步驟三：執行初始化指令
```shell
$ git submodule update

------
子模組路徑「SlackReport」：已簽出「159866efd46bcdce797569bc6a56eff7310c4c98」
子模組路徑「lib/submodules/bigquery-logger」：已簽出「a52aa0c80339391b8af145d2ceb76fe4faff1ac5」
子模組路徑「lib/submodules/cache-image」：已簽出「4ad3575ffca3237c24c3348eeb8c430ff7f814e9」
子模組路徑「lib/submodules/django-gcloud-storage」：已簽出「f642a297f97b695570f344cc8cdc24b2d5936c52」
子模組路徑「lib/submodules/ec」：已簽出「637b07a3fc0def5aa8745acfedcc238a5fb5b196」
子模組路徑「lib/submodules/gcloudorm」：已簽出「04880519be92d3be9d48163049d4467b3e6c08ef」
子模組路徑「lib/submodules/json-extend」：已簽出「b3cb5e4b06627b6637d28323fefadbe004b25f57」
子模組路徑「lib/submodules/ndb-utils」：已簽出「c562410e7e377fda89848f3ea289fccb8fb8d0f4」
子模組路徑「lib/submodules/pyslack」：已簽出「bafa47863ddffbbda5b36d2452c784a4dd3ce6c8」
子模組路徑「node_modules/Tagtoo_tools」：已簽出「556906214595e8a3025ff540492e45463bed27e7」
子模組路徑「tagtoo-config」：已簽出「dd5ae8b647b2a49a147dfc0d6fbcb70a44f39f2f」
```



### 步驟四：新增環境變數

```shell
$ export DOCKER_BUILDKIT=0
$ export COMPOSE_DOCKER_BUILD=0
```







### 步驟五：拉下 image 檔案

直接拉下這個 image 檔案，會需要 howard 或 patrick 幫忙開通權限，如果遇到噴下面的錯誤要開他們說：

```shell
$ docker pull asia.gcr.io/dashboard-dev-1230/dashboard-api-base:latest

------
# 下面這一行是沒權限才會噴的錯
Error response from daemon: unauthorized: You don't have the needed permissions to perform this operation, and you may have invalid credentials. To authenticate your request, follow the steps in: https://cloud.google.com/container-registry/docs/advanced-authentication
```



### 步驟六：改標籤名

這邊改標籤名稱，是因為 dockerfile 那邊設定的關係，如果不改名稱的話吃不到
```shell
$ docker image tag asia.gcr.io/dashboard-dev-1230/dashboard-api-base:latest dashboard-api-base:latest
```

我們順便來看一下現在 image 有哪些

```shell
$ docker image list --all

------
REPOSITORY                                          TAG       IMAGE ID       CREATED       SIZE
asia.gcr.io/dashboard-dev-1230/dashboard-api-base   latest    c2b960f5badb   5 years ago   1.57GB
dashboard-api-base                                  latest    c2b960f5badb   5 years ago   1.57GB
```






### 步驟七：把 test.Dockerfile 不必要的檔案刪掉

#### 原始檔案
```dockerfile
ARG BASE_IMAGE_NAME=dashboard-api-base
FROM ${BASE_IMAGE_NAME}

LABEL author="Jay Liang (roackb2@gmail.com)" \
      maintainer="Frank Jheng (m157q.tw@gmail.com)"

ENV DJANGO_SETTINGS_MODULE=settings.test_settings
ADD . /srv/work/
RUN pip install -r requirements.txt
RUN pip install -r requirements_test.txt

# 下面四行是要刪掉的
RUN python manage.py syncdb --database=backup --noinput
RUN python manage.py syncdb --database=old-prod --noinput
RUN python manage.py syncdb --database=default --noinput
RUN echo "from django.contrib.auth.models import User; User.objects.create_superuser('tagtootest', 'test@tagtoo.org', 'tagtootestpwd')" | python manage.py shell

CMD python manage.py runserver 0:8000
```


#### 更改過後的檔案
```dockerfile
ARG BASE_IMAGE_NAME=dashboard-api-base
FROM ${BASE_IMAGE_NAME}

LABEL author="Jay Liang (roackb2@gmail.com)" \
      maintainer="Frank Jheng (m157q.tw@gmail.com)"

ENV DJANGO_SETTINGS_MODULE=settings.test_settings
ADD . /srv/work/
RUN pip install -r requirements.txt
RUN pip install -r requirements_test.txt


CMD python manage.py runserver 0:8000
```




### 步驟八：修改 test_setting.py 的資料庫設定


#### 原始檔案
```py
import os
import yaml
from base_settings import *

# os.environ['GOOGLE_APPLICATION_CREDENTIALS'] = 'tagtoo-test-825c8b08f173.json'
# from gcloud import datastore
# datastore.set_defaults("tagtoo-test")

DEBUG = True
DJANGO_GCS_BUCKET = 'tagtoo-storage-backend-test'

INSTALLED_APPS += (
    # requirements_test.txt
    'django_extensions',
)
SHELL_PLUS = "ipython"

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': 'console.db',
        'TEST': {
            'DEPENDENCIES': ['backup', 'old-prod'],
        },        
    },
    # mimic the backup instance for production environment
    'backup': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': 'console_backup.db',
    },
    # mimic the old production MySQL instance for testing
    'old-prod': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': 'console_old_prod.db',
    },
}

LOGGNING_CONFIG_PATH = os.path.join(ROOT_PATH, 'logconf-test.yaml')
LOGGING = yaml.load(open(LOGGNING_CONFIG_PATH).read())
packages = ",".join(os.listdir('apps'))
NOSE_ARGS = [
    '--cover-package=%s' % packages,
    '--with-coverage',
]

```

#### 更改過後的檔案

我們根據 base_setting.py 的設定，來更改我們 test_setting.py 的資料庫
```py
import os
import yaml
from base_settings import *

# os.environ['GOOGLE_APPLICATION_CREDENTIALS'] = 'tagtoo-test-825c8b08f173.json'
# from gcloud import datastore
# datastore.set_defaults("tagtoo-test")

DEBUG = True
DJANGO_GCS_BUCKET = 'tagtoo-storage-backend-test'

INSTALLED_APPS += (
    # requirements_test.txt
    'django_extensions',
)
SHELL_PLUS = "ipython"

DATABASES = {
    # the "sync-gothic-sql" instance of CloudSQL on GCP "dashboard-dev-1230",
    # it's also the intermidiate DB in the migration process
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'HOST': '104.155.205.124',
        'NAME': 'console',
        'USER': 'root',
        'PASSWORD': 'HadbykpemhyptOdNiOmRytKobryuInt2',
    },
}

LOGGNING_CONFIG_PATH = os.path.join(ROOT_PATH, 'logconf-test.yaml')
# LOGGING = yaml.load(open(LOGGNING_CONFIG_PATH).read())
LOGGING = yaml.safe_load(open(LOGGNING_CONFIG_PATH).read())
packages = ",".join(os.listdir('apps'))
NOSE_ARGS = [
    '--cover-package=%s' % packages,
    '--with-coverage',
]

```



### 步驟九：make test-up

這個指令會呼叫 docker compose up:

```shell
$ make test-up

------
docker-compose -f docker-compose.yml up
[+] Running 2/2
 ⠿ Container dashboard-api-cache  Recreated                                                                                                                                                                                                    0.1s
 ⠿ Container dashboard-api        Recreated                                                                                                                                                                                                    0.1s
Attaching to dashboard-api, dashboard-api-cache
dashboard-api-cache  | 1:C 21 Jun 10:03:00.270 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
dashboard-api-cache  | 1:C 21 Jun 10:03:00.271 # Redis version=4.0.14, bits=64, commit=00000000, modified=0, pid=1, just started
dashboard-api-cache  | 1:C 21 Jun 10:03:00.271 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
dashboard-api-cache  | 1:M 21 Jun 10:03:00.276 * Running mode=standalone, port=6379.
dashboard-api-cache  | 1:M 21 Jun 10:03:00.276 # Server initialized
dashboard-api-cache  | 1:M 21 Jun 10:03:00.276 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
dashboard-api-cache  | 1:M 21 Jun 10:03:00.276 * DB loaded from disk: 0.000 seconds
dashboard-api-cache  | 1:M 21 Jun 10:03:00.276 * Ready to accept connections
dashboard-api        | Validating models...
dashboard-api        | 
dashboard-api        | 0 errors found
dashboard-api        | June 21, 2023 - 18:03:14
dashboard-api        | Django version 1.6.3, using settings 'settings.test_settings'
dashboard-api        | Starting development server at http://0.0.0.0:8000/
dashboard-api        | Quit the server with CONTROL-C.
```


這樣我們就成功啟動 server 了！！！！



### 參考連結：
1. https://github.com/Tagtoo/dashboard-api
2. https://docs.google.com/document/d/1Ct2Y2eou7p4JocCyi8YoXBLVjAfE2TGa23PCw27t9zw/edit










