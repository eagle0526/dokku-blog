---
sidebar_position: 4
---

## SQLALchemy 介紹

SQLAlchemy 是 Python 的一款開源軟體，提供 SQL 工具包及 ORM 的功能；為開發人員提供 SQL 的功能並保持靈活性；允許開發人員使用 Python 類來定義數據庫模型，並自動將這些類的對象映射到數據庫表格；專為高效的資料庫存取而設計；為Python中最廣泛使用的ORM框架

參考來源：https://reginapanpan.medium.com/sqlalchemy-%E6%98%AF%E4%BB%80%E9%BA%BC-%E8%88%87-orm-%E7%9A%84%E9%97%9C%E4%BF%82-a57b16053aec

## 實際舉個例子


### 1. 安裝 sqlalchemy

```shell
$ pip install sqlalchemy
$ pip install pymysql
```


### 2. 初始化

```py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()
engine_url = "<url>"
engine = create_engine(engine_url, echo=True)
```

* 若設定 echo 為 True，會將執行過程印到 terminal 上
* 使用 sqlite 則 engine_url 為 .db 的檔案位置，範例如下: `sqlite:///C:\\<path>\\test.db`
* 使用 mysql 則為需要設定使用者名稱、密碼、端口等，格式如下: `mysql+pymysql://<username>:<password>@<host>:<port>/<database_name>`


### 3. 建立資料表

#### (一) 設定資料表結構  
要特別注意 `sqlalchemy` 不允許修改表結構，如果需要修改的話，需要刪除重建

```py
from sqlalchemy import Column
from sqlalchemy import Integer, String, DATETIME

class Test(Base):
    __tablename__ = "test"
    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(55))
    time = Column(DATETIME)    
```

#### (二) 建立資料表和刪除資料表      
```py
def create_table():
    Base.meta.create_all(engine)

def drop_table():
    Base.meta.drop_all(engine)

if __name__ == '__main__':
    drop_table()
    create_table()
```

#### (三) 建立操作實體

```py
from sqlalchemy.orm import sessionmaker

def create_session():
    Session = session(bind=engine)
    session = session()
    return session
```



### 介紹 import 的函示


* create_engine
這是 SQLAlchemy 中的一個函式，用於創建一個數據庫引擎（database engine）的實例。數據庫引擎用於管理與數據庫的通信，並處理執行 SQL 命令的所有細節。您需要提供一個數據庫的連接 URL，告訴 SQLAlchemy 如何連接到您的數據庫。

* declarative_base
這是 SQLAlchemy 中的一個 class，用於創建數據庫模型的 base class。通過繼承這個 base class，您可以定義您的數據庫表格（table）以及它們的列（column）。這樣做的好處是您可以使用 Python 類來表示數據庫表格，並使用類的屬性來表示表格中的列。

* Column
這是 SQLAlchemy 中的一個 class，用於定義數據庫表格中的列。您可以使用它來指定列的名稱、類型、約束等信息。

* sessionmaker
這是 SQLAlchemy 中的一個 class，用於創建會話（session）的 factory。會話是一個用於與數據庫進行交互的上下文，它包含了所有的數據庫操作（例如查詢、添加、更新、刪除等）。通過 sessionmaker，您可以配置並創建會話對象，以便進行數據庫操作。




### 參考資料來源
* https://ithelp.ithome.com.tw/articles/10282661   
* https://ithelp.ithome.com.tw/articles/10282664   