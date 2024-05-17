---
sidebar_position: 4
---


## type hint

Python 的 type hint 是一種在程式碼中用來指定變量、函數參數和返回值類型的標註方式。這種標註可以幫助開發者更清晰地理解程式碼的意圖，並提供靜態分析工具和 IDE 更好的程式碼提示和錯誤檢查功能。

以下介紹 type-hint 的幾個要點：

### 1. 變量標註

這裡的 x 是一個整數類型的變數，初始值為 5

```py
x: int = 5
```

### 2. 函數參數標注

這個函數接受兩個整數型的參數 `x` 和 `y`，最後會返回一個 `int`
```py
def add(x: int, y: int) -> int:
    return x + y
```

### 3. 函數返回值標註

這裡個函數接受一個字串型的參數，最後會返回一個 `str`

```py
def greeting(name: str) -> str:
    return "Hello, " + name
```


### 4. 複雜類型

這個函數接受一個字典陣列，其中每個字典都有一個鍵為 `value` 的整數值，最後會返回一個整數的陣列，該陣列包含每個字典的 `value` 鍵值

```py
from typing import List, Dict

def process_data(data: List[Dict[str, int]]) -> List[int]:
    return [item['value'] for item in data]
```


### 5. 可選類型

這個函數接受一個可選的字串參數 `name`，如果 `name` 為空，則返回 `Hello, world`，否則返回 `Hello, ` 加上 `name` 值

```py
from typing import Optional
def greet(name: Optional[str] = none) -> str:
    if name is None:
        return "Hello, World!"
    else:
        return "Hello, " + name
```


### 6. 泛型

這個函數接受一個 `tuple` 陣列，其中每個 `tuple` 的第一個元素是字串，第二個元素是整數，最後返回一個 >> `包含第一個和最後一個 -> items[0] 和 items[-1]`，的 `第一個元素的tuple -> items[0][0] 和 items[0][-1]` 

```py
from typing import List, Tuple

def first_last(item: List[Tuple[str, int]]) -> tuple[str, str]:
    return items[0][0], items[-1][0]
```




