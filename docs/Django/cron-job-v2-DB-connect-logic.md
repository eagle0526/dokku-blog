---
sidebar_position: 4
---

## cron-job-v2 - 資料庫連接邏輯


## 大綱介紹
1. 建立好 model.py 的資料，並且設定好 Meta 屬性 - ex. app_label, managed, db_table
2. 設定好 enums.py，這個資料的目的是讓程式比較好閱讀 + 可重複使用變數
3. 設定好 db_routers.py，這個檔案很重要，是主要判斷現在是連結哪個資料庫的路徑檔案
4. 最後設定 init.py (普通的 django 是設定在 settings.py)，這個檔案就是控制 django 所有基礎設定的檔案，包含資料庫、時區、語言...等等之類的



## 詳細介紹

### 1. models.py 檔案介紹

1-1、app_label 是指定現在的應用程式，而這個 label 之後會對應到 db_routers 裡面的資料，等等會提到
1-2、managed 這個參數的意思就是，你要不要給 django 自動管理該 model 自動對應 table 的創建和修改
1-3、db_table 就是你的 table 名稱


ps. `DatabaseLabelEnum.old_db.value` 的意思就是 `old-db`

```py
# models.py

from db.orm.enums import DatabaseLabelEnum

class OldDBProduct(DirtyFieldsMixin, models.Model):
    advertiser_id = models.IntegerField(null=True, blank=True)
    # ...省略

    class Meta():
        app_label = DatabaseLabelEnum.old_db.value        
        managed = False
        db_table = 'console_productad'



class NewDBProduct(DirtyFieldsMixin, models.Model):
    ec_id = models.IntegerField()

    # ...省略

    class Meta():
        app_label = DatabaseLabelEnum.new_db.value
        managed = False
        db_table = 'products'
```


### 2. enums.py 檔案介紹

這個檔案就是在設定一些 enums，想要知道詳細的應用，可以到 `django-enum` 那邊看

```py
# enums.py
from enum import Enum
from typing import List


class BaseEnum(Enum):
    @classmethod
    def has_value(cls, value):
        return value in cls._value2member_map_

    @classmethod
    def get_values(cls) -> List:
        return [enum_obj._value_ for enum_obj in cls._value2member_map_.values()]


class DatabaseNameEnum(BaseEnum):
    old_db_primary = 'old-db-primary'
    old_db_replica = 'old-db-replica'
    new_db_primary = 'new-db-primary'
    new_db_replica = 'new-db-replica'


class DatabaseLabelEnum(BaseEnum):
    old_db = 'old-db'
    new_db = 'new-db'
```


### 3. db_routers 檔案介紹

這邊不管是新資料庫還是舊資料庫，我們都有個有額外準備一個，也就是一個主要是寫入，一個是讀取

1. 首先寫一個方法判斷這個 model 的 label 是否是 `old-db`
2. 再來寫 `NewDBRouter` 和 `OldDBRouter` 兩個類別，等等要給在 init.py (settings.py) 那邊做判斷要連哪個 db 
3. 用第一點的方法 + `model._meta.app_label` 就可以知道現在 model 的 app_label 是哪一個
4. `db_for_read` 、 `db_for_write` 是 django router 基本上都會要有的方法，主要就是一個是寫入，一個是讀取

ps. https://stackoverflow.com/questions/53859629/how-to-add-database-routers-to-a-django-project 這裡有更多 django router 的設定資訊

```py
# db_routers.py

from db.orm.enums import DatabaseLabelEnum, DatabaseNameEnum

"""
Writes go to primary DB
Reads go to replica DB
"""


def is_old_db_label(db_label) -> bool:
    return db_label == DatabaseLabelEnum.old_db.value

 
class NewDBRouter:
    def db_for_read(self, model, **hints):
        if is_old_db_label(model._meta.app_label):
            return None
        return DatabaseNameEnum.new_db_replica.value

    def db_for_write(self, model, **hints):
        if is_old_db_label(model._meta.app_label):
            return None
        return DatabaseNameEnum.new_db_primary.value


class OldDBRouter:
    def db_for_read(self, model, **hints):
        if is_old_db_label(model._meta.app_label):
            return DatabaseNameEnum.old_db_replica.value

    def db_for_write(self, model, **hints):
        if is_old_db_label(model._meta.app_label):
            return DatabaseNameEnum.old_db_primary.value

```

### 4. init_py 檔案介紹

最後就是這個設定檔案介紹，原本 django 檔案都是在 setting.py 檔案裡面設定，這個專案比較特別。   

1. 先用 pymysql 等等可以連到 mysql 的資料庫
2. `orm_app_path` 這個路徑要加的是資料夾下的路徑 -> ex. 這邊的資料夾架構就是 `db/orm/__init__.py`，因此這裡的 `orm_app_path` 就是 `db.orm`
3. `db_router_path` 這個是最重要的判斷要讀取哪個資料庫的路徑，由於我們 `db_routers` 放的架構是 `db/orm/db_routers.py`，因此這邊才會這樣設定 `db.orm.db_routers`
4. `db_router_path` 這個變數要配合 `DATABASE_ROUTERS`，還記得我們在 `db_routers` 設定的兩個 `class` 嗎？這個路徑會去讀取要吃哪個 `class`，然後看是 `model` 對照到哪個 `app_label`，並且判斷現在是要 `read` 還是 `write`
5. 最後依照 `db_router.py` 返回的值， `DatabaseNameEnum.new_db_replica.value` or `DatabaseNameEnum.old_db_replica.value` 之類的，就可以知道 `DATABASES` 到底要連哪一個，在 `settings.py` 這邊會把資料庫真實的主機位置和密碼寫好，這樣最後就可以成功連上資料庫。  
```py
import django
import pymysql
from django.conf import settings
from db.orm.enums import DatabaseNameEnum

pymysql.install_as_MySQLdb()
orm_app_path = 'db.orm'
db_router_path = 'db.orm.db_routers'

settings.configure(
    DEBUG=True,
    USE_TZ=True,
    TIME_ZONE='Asia/Taipei',
    INSTALLED_APPS=[
        orm_app_path,
        'django.contrib.postgres',
        'bulk_update_or_create'
    ],
    DATABASE_ROUTERS=[
        f"{db_router_path}.NewDBRouter",
        f"{db_router_path}.OldDBRouter",
    ],
    DATABASES={
        DatabaseNameEnum.old_db_primary.value: {        
            'ENGINE': 'django.db.backends.mysql',
            'HOST': '主機位置',
            'USER': '使用者名稱',
            'NAME': '主機名稱',
            'PASSWORD': '主機密碼',
            'OPTIONS': {'charset': 'utf8mb4'}
        },
        DatabaseNameEnum.old_db_replica.value: {
            'ENGINE': 'django.db.backends.mysql',
            'HOST': '主機位置',
            'USER': '使用者名稱',
            'NAME': '主機名稱',
            'PASSWORD': '主機密碼',
            'OPTIONS': {'charset': 'utf8mb4'}
        },
        DatabaseNameEnum.new_db_primary.value: {
            'ENGINE': 'django.db.backends.postgresql',
            'HOST': '主機位置',
            'USER': '使用者名稱',
            'NAME': '主機名稱',
            'PASSWORD': '主機密碼',
            'PORT': 5432,
            "OPTIONS": {
                "connect_timeout": 10,
            },
        },
        DatabaseNameEnum.new_db_replica.value: {
            'ENGINE': 'django.db.backends.postgresql',
            'HOST': '主機位置',
            'USER': '使用者名稱',
            'NAME': '主機名稱',
            'PASSWORD': '主機密碼',
            'PORT': 5432,
            "OPTIONS": {
                "connect_timeout": 10,
            },
        },
        'default': {}
    },
)
django.setup()
```


