---
sidebar_position: 1
---

## part1 - 創建資料庫

## 1. 資料夾架構

初始資料夾架構，一開始先像下面這樣，用 virtualenv 創造一個虛擬環境，並且多新增一個 `sql_alchemy` 的資料夾，等等串接資料庫的相關程式都會放在這一區塊
```md
| FASTAPI
  |  -- sql_alchemy
        | -- __init__.py
        | -- query.py
  |  -- virtualenv
  |  app.py
```
Ps. app.py 和 query.py 裡面都是空的
Ps. 有 `__init__.py` 是因為如果今天要 import 一些函式，要把 `sql_alchemy` 變成一個 package 才能 import 


## 2. 使用 SQLAlchemy 創建新的 db

現在我們先來用 `SQLAlchemy` 創建 DB，並且用政府提供的一些資料，把資料塞進去資料庫，等等我們可以來用

### 2-1 - 先安裝 sqlalchemy package
```shell
$ pip install SQLAlchemy
```

### 2-2 再來創建檔案 - sql_alchemy.py
```md
| FASTAPI
  |  -- sql_alchemy
        | -- __init__.py  
        | -- query.py
        | -- sql_alchemy.py  => 新增這個
  |  -- virtualenv
  |  app.py
```

### 2-3 引入會用到的函式

* create_engine: create_engine 是告訴 sqlalchemy 要怎麼連到你的資料庫
* declarative_base: 是創建基礎 model 的 base class
* Column 是創建 table 的基本欄位
* Integer, String, DATETIME, TEXT 是 table 欄位的屬性
* sessionmaker 讓資料庫可以使用 ORM 跟數據互動

```py
# sql_alchemy.py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column
from sqlalchemy import Integer, String, DATETIME, TEXT
from sqlalchemy.orm import sessionmaker
```


### 2-4 連接資料庫

```py
# 以下是連接資料庫的資訊
MYSQL_HOST = 'localhost'
MYSQL_USER = 'newuser'
MYSQL_PASSWORD = 'newpassword'
MYSQL_PORT = 3306
MYSQL_DB = 'test'

SQLALCHEMY_DATABASE_URL = f"mysql+pymysql://{MYSQL_USER}:{MYSQL_PASSWORD}@{MYSQL_HOST}:{MYSQL_PORT}/{MYSQL_DB}"
```

ps. 這些變數的來源，都是一開始創建 DB 會知道的值，如果不太懂可以參照 `SQL -> MySQL 語法` 檔案裡面的 `遠端 MySql 連接` 內容，目前是在本地端執行，所以 `HOST` 是 `localhost`


### 2-5 連接資料庫 + 創建 model 基礎的 base_class

```py
# declarative_base 是創建基礎 model 的 base class，所以等等創建的 model 會繼承他
Base = declarative_base()

# create_engine 是告訴 sqlalchemy 要怎麼連到你的資料庫 - echo 是會把連接的所有訊息印出來
engine = create_engine(SQLALCHEMY_DATABASE_URL, echo=True)
```

### 2-6 設定 table 的屬性

* create_table 執行後，可以直接在指定的地方創建 table
* drop_table 可以刪掉 table
* create_session 可以跟 table 用 orm 互動

```py
# 這裡設定的是等等要創建的
class Website(Base):
    __tablename__ = "website"
    id = Column(Integer, primary_key=True, autoincrement=True)
    title = Column(String(100))
    content = Column(TEXT)
    picture = Column(TEXT)  # Chrome 網址長度上限
    category = Column(String(100))
    youtube = Column(TEXT)  # Chrome 網址長度上限
    slideshare = Column(TEXT)  # Chrome 網址長度上限
    publish_time = Column(DATETIME)
    update_time = Column(DATETIME)


# 創建 table
def create_table():
    Base.metadata.create_all(engine)


# 刪除 table
def drop_table():
    Base.metadata.drop_all(engine)

# 讓資料庫可以使用 ORM 跟數據互動
def create_session():
    Session = sessionmaker(bind=engine)
    session = Session()

    return session
    

if __name__ == '__main__':
    # drop_table()
    create_table()
```


### 最終完整檔案
```py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column
from sqlalchemy import Integer, String, DATETIME, TEXT
from sqlalchemy.orm import sessionmaker


# 設定好要連接的資料庫
MYSQL_HOST = 'localhost'
MYSQL_USER = 'newuser'
MYSQL_PASSWORD = 'newpassword'
MYSQL_PORT = 3306
MYSQL_DB = 'test'

SQLALCHEMY_DATABASE_URL = f"mysql+pymysql://{MYSQL_USER}:{MYSQL_PASSWORD}@{MYSQL_HOST}:{MYSQL_PORT}/{MYSQL_DB}"

# declarative_base 是創建基礎 model 的 base class，所以等等創建的 model 會繼承他
Base = declarative_base()

# create_engine 是告訴 sqlalchemy 要怎麼連到你的資料庫 - echo 是會把連接的所有訊息印出來
engine = create_engine(SQLALCHEMY_DATABASE_URL, echo=True)

# 這裡設定的是等等要創建的
class Website(Base):
    __tablename__ = "website"
    id = Column(Integer, primary_key=True, autoincrement=True)
    title = Column(String(100))
    content = Column(TEXT)
    picture = Column(TEXT)  # Chrome 網址長度上限
    category = Column(String(100))
    youtube = Column(TEXT)  # Chrome 網址長度上限
    slideshare = Column(TEXT)  # Chrome 網址長度上限
    publish_time = Column(DATETIME)
    update_time = Column(DATETIME)


# 創建 table
def create_table():
    Base.metadata.create_all(engine)


# 刪除 table
def drop_table():
    Base.metadata.drop_all(engine)

# 讓資料庫可以使用 ORM 跟數據互動
def create_session():
    Session = sessionmaker(bind=engine)
    session = Session()

    return session
    

if __name__ == '__main__':
    # drop_table()
    create_table()
```

