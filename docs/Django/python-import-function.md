---
sidebar_position: 4
---

## python import function 的方式


### import 規則

1. 引入檔案在同層級
```py
import file
```
2. 引入檔案在下層級目錄的文件
* 需要先在該引入的文件中新增一個 `__init__` 檔案
```py
from dir import file
```

3. 引入上層目錄中，其他層級的資料
```py
import sys
sys.path.append("指定層級")
from dir import file
```

第三個有點難懂沒關係，等等會有實際範例

假設我現在有個 `python` 專案，並且資料夾架構是以下這樣：



## 實際案例

### 資料夾結構
```md
main
- cron-job
  - first-project
    - main.py
  - second-project
    - main.py          -> 我想要在這個資料夾，引用 function 資料夾下面 add.py 中的 function
    - sub.py
- db
  - orm
    - __init__.py
    - db_routers.py
    - enums.py
    - models.py
  - manage.py
- function
  - __init__.py
  - add.py
```

:::tip __init__.py 檔案
__init__.py 是 Python 中的特殊檔案，其存在的目的是將目錄視為 Python 套件。當 Python 搜索模組或套件時，它會查找包含 __init__.py 檔案的目錄，並將其視為套件。

這意味著，如果您想要將一個目錄視為 Python 套件並在其中包含多個模組或子套件，您需要在該目錄中放置一個空的 __init__.py 檔案。這告訴 Python 該目錄是一個套件，可以被引入和使用。
:::



### 第一、二種引入 - 同層級 or 低層級的

1. 先設定好 `sub.py` 裡面的函示
```py
# main/cron-job/second-project/sub.py

def subFuc(one, two):
    return one - two
```

2. 在 main.py 中引入使用


```py
from sub import subFuc
def main():
    print(subFuc(10,1))

if __name__ == "__main__":
    main()
```


### 第三種引入 - 上層資料夾的其他資料引入

1. 先設定好 `function 資料夾下的 add.py 檔案內容`：

* 記得一定要在 `function資料夾` 下面新增 `__init__.py` 這個檔案，這樣才可以讓 python 知道這個資料夾是可以 import 的
```py
# function/add.py

def addFuc(one, two):
    return one + two
```

2. 接下來到 `second-project` 這個資要夾下面的 main.py 引入進去

```py
# cron-job/second-project/main.py

import sys
import os

# 下面這段可以拿到目前該檔案的絕對路徑 - ex. /Users/yee0526/Desktop/python/connectDB/main/cron-job/second-project
main_dir = os.path.abspath(os.path.dirname(__file__))

# 下面這段可以根據剛剛的路徑往上一層 - ex. /Users/yee0526/Desktop/python/connectDB/main/cron-job
parent_dir = os.path.abspath(os.path.join(main_dir, os.pardir))

# 下面這段可以再往上一層 - ex. - /Users/yee0526/Desktop/python/connectDB/main
target_dir = os.path.abspath(os.path.join(parent_dir, os.pardir))

print(target_dir)
# 上面三段可以用以下這一段一次取代
# target_dir = os.path.abspath(os.path.join(os.path.dirname(__file__), '../../'))

sys.path.append(target_dir)
from function.add import addFuc

def main():
    print(addFuc(5, 9))

if __name__ == "__main__":
    main()
```






### 參考資料來源
https://blog.csdn.net/liupeng19970119/article/details/111991651