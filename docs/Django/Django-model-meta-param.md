---
sidebar_position: 2
---

## Django model class meta param

這裡來介紹一下，在 `model.py` 頁面的 `meta` 中，有哪些參數可以控制：
https://docs.djangoproject.com/zh-hans/5.0/topics/db/models/#abstract-base-classes

### abstract

這個參數特殊的地方在於，如果今天你在該 `class` 下方下這個參數，代表這個 `model` 是一個抽象的 `model`，這是什麼意思呢？意思就是不會有一個真的 `table` 對應到這個 model，裡面的資料實際上是不存在的。  
那這個用處在哪呢？下面展示一個實際案例：     

```py
from django.db import models


# model 1
class CommonInfo(models.Model):
    name = models.CharField(max_length=100)
    age = models.PositiveIntegerField()

    class Meta:
        abstract = True

# model 2
class Student(CommonInfo):
    home_group = models.CharField(max_length=5)
```

我現在設定兩個 model，一個是 `CommonInfo` 他是抽象類別，這個類別特殊的地方在於，他不會生成 table、沒有管理器，也不能被實例和保存。
而現在有另外一個 `model`，`student`，他繼承了 `CommonInfo`，因此他有 `CommonInfo` 所含有的屬性。



### app_label

app_label 用於指定該 `model` 所屬的應用程式，每個 `django` 都包含一個或多個 `model`，這些 `model` 通常定義在應用程式的 `model.py` 中，如果今天不指定 `app_label` 的話，django 會根據 `model` 所在的應用程式來自動判斷 `app_label`。  




### db_table

這個參數就是讓你讀取你實際資料庫的名稱：

```py
db_table = "music_album"
```

為了節省時間， django 一開始會自動從 `model 類別` 和 `應用程式的名稱` 中，抓出資料表的名稱， `model 的資料表名稱`，是透過將 `model` 的 `app_label` (在 manage.py startapp 中使用的名稱) 連接到 `model 的類別名稱`，兩者之間會用下劃線來建立。


例如：如果你的 `app` 的名稱叫做 `bookstore` (經由此指令創造出來 - manage.py startapp bookstore)，然後你的 model 叫做 `class book`，那你的 `table 名稱就叫做 - bookstore_book`



## Meta 繼承

這時候順便來說一下 meta 繼承，當今天建立一個抽象類別，若子類沒有設定自己的 meta 的話，他會自動繼承父類的 meta：

```py
from django.db import models


class CommonInfo(models.Model):
    # ...
    class Meta:
        abstract = True
        ordering = ["name"]


class Student(CommonInfo):
    # ...
    class Meta(CommonInfo.Meta):
        db_table = "student_info"
```

上面這樣設定 `class Meta(CommonInfo.Meta)`，代表的是 `Student` 這個類別會自動繼承 `CommonInfo` 的 `meta`，也就是說 `Student` 在排列的時候會依照 `name` 進行順序排列。    
     
但！這邊要特別提到，雖然說會自動繼承上層的 `meta` 屬性，但唯獨 `abstract` 不會自動繼承， `meta 繼承` 預設情況下，子類別的 `abstract 預設為 false`，也就是說如果父類別是抽象類別，子類別繼承該父類別，也不會自動變成成抽象類別。  






<!-- 整個流程的跑法 -->
<!-- 1. 先到 -->