---
sidebar_position: 4
---

## Pydantic model 介紹

`Pydantic` 是以 `type hints` 為基礎，幫我們做資料型態驗證的套件，可幫你驗證資料的 `data type` ，及是否符合規則 (像是對應欄位是否為 emil)，我們可以用它定義「Model」。




### 實際舉個例子

我們現在創建一個 `model`：

```py
from pydantic import BaseModel

class StudentModel(BaseModel):
    name: str
    age: int
```

再來我們新增兩個物件：

```py
student = StudentModel(
    name='John',
    age=15,
)

student2 = StudentModel(
    name='Tom',
    age='17',
)
```
這時候 `student2` 會噴出以下錯誤：

```md
Traceback (most recent call last):
    <!-- ...省略 -->
    raise validation_error
pydantic.error_wrappers.ValidationError: 1 validation error for StudentModel
age
  value is not a valid integer (type=type_error.integer)
```


## FastAPI + pydantic

FastAPI 和 pydantic 深度整合，只要在創建 model 的時候先聲明好，所有的 api 端點都會有自動驗證機制。

以下是一個範例：

```py
from fastapi import FastAPI, Query
from pydantic import BaseModel

class Product(BaseModel):
    title: str
    description: str = None
    price = float
    tax: Optional[float] = None


app = FastAPI()

@app.post("/products/")
async def create_product(product: Product):
    return product
```

在 Product 的日個屬性中，我們定義了四個屬性，比較特別的是 `tax` 的 `type hint` 是 `optional`，也就是如果今天建立 `object` 時沒有給他這個屬性值，他就會是 `None`。    
最後新增的一個 `/products/` 的端點，並且在下方新增了 `product` 的參數，這個參數就是前面建立好的 `Product class`，也就是今天前端送來的 `request body` 會自動被 `pydantic` 解析，如果沒有照著前面建立好了 `class` 規則傳送，則會噴出錯誤。

Ps. 可以輸入這個網址 `http://127.0.0.1:8000/docs`，這個有 `api` 端點介面可以讓你更清楚你建立了哪些 `endpoint` 和 `規則`







參考資源：
1. https://www.runoob.com/fastapi/fastapi-request-response.html
2. https://editor.leonh.space/2023/pydantic/