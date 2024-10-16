---
sidebar_position: 2
---

## part2 - 制定 Product Model + Hello World

## 1. 創建 Product Model
編輯 `backend/api/models.py`:

```py
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=200)
    price = models.DecimalField(max_digits=10, decimal_places=2)

    def __str__(self):
        return self.name
```

## 3. Database 設定
編輯 `backend/ecommerce/settings.py`:

```py
# ...其他忽略...

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'ecommerce',
        'USER': 'postgres',
        'PASSWORD': 'postgres',
        'HOST': 'db',
        'PORT': 5432,
    }
}

# ...其他忽略...
```

## 4. 具現化資料庫

```shell
$ docker-compose run backend python manage.py makemigrations
$ docker-compose run backend python manage.py migrate
```

## 5. 新增 api


### 修改 views.py
修改 backend/api/views.py 文件：

```py
from django.http import JsonResponse
from .models import Product

def get_products(request):
    products = Product.objects.all().values('id', 'name', 'price')
    return JsonResponse(list(products), safe=False)
```

### 修改 urls.py
修改 backend/ecommerce/urls.py 文件：
```py
from django.contrib import admin
from django.urls import path
from api.views import get_products

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/products/', get_products, name='get_products'),
]
```

## 6. 創建 Product 的測試資料
在 backend/api/management/commands/ 目錄下創建一個新文件 create_sample_products.py：


```md
| ecommerce-project
  |  backend
   -- | api
       -- | management
           -- | commands
               -- | `create_sample_products.py`
      <!-- 忽略 -->
  |  frontend
      <!-- 忽略 -->  
  |  docker-compose.yml
```

* create_sample_products 資料夾內容：

```py
from django.core.management.base import BaseCommand
from api.models import Product

class Command(BaseCommand):
    help = 'create sample products'

    def handle(self, *args, **kwargs):
        products = [
            {'name': 'Product 1', 'price': 100},
            {'name': 'Product 2', 'price': 200},
            {'name': 'Product 3', 'price': 300},
            {'name': '海賊帽', 'price': 500},
            {'name': '梅莉號', 'price': 99999},
        ]
        for product in products:
            Product.objects.create(**product)

        self.stdout.write(self.style.SUCCESS('Successfully created sample products'))
```

## 7. 啟用新寫好的 command
確保 api 應用已經被添加到 INSTALLED_APPS 中，檢查 `backend/ecommerce/settings.py` 文件:

```py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'api',                          # 添加這一行
]
```

## 8. 執行 create_sample_products 指令

* 這行指令建立好後，資料庫就確實會有資料出現

```shell
$ docker-compose run backend python manage.py create_sample_products
```


### 進入 console 看資料庫

如果實在不確定資料庫到底有沒有資料，可以用以下的指令進入 console 確認

```shell
$ docker-compose run backend python manage.py shell


------ # 進入後，執行以下三行，就可以確認 products 到底有沒有測試資料
from api.models import Product
products = Product.objects.all()
products
```


## 9. 避免跨網域的問題
為了解決跨域問題，我們需要在後端安裝 `django-cors-headers`：

* 修改 backend/requirements.txt：

```txt
Django>=4.2.2
psycopg2-binary>=2.9.3
django-cors-headers>=3.10.0     -> 新增這個
```

* 修改 backend/ecommerce/settings.py：

```py
# 在 INSTALLED_APPS 中添加：
INSTALLED_APPS = [
    # ...
    'corsheaders',
    # ...
]

# 在 MIDDLEWARE 中添加（盡量靠前）：
MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    # ...
]

# 在文件底部添加：
CORS_ALLOW_ALL_ORIGINS = True  # 僅在開發環境中使用，生產環境應該設置具體的允許來源
```


## 10. 建立 Hello World 頁面

確定測資準備好了，接著我們來把資料印在頁面上 -> `frontend/src/App.js`：

```js
import React, { useState, useEffect } from 'react';
import './App.css';

function App() {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    fetch('http://localhost:8000/api/products/')
     .then(response => response.json())
     .then(data => {
      if (data.products) {
        setProducts(data.products);
      }
     })
     .catch(error => console.error('Error:', error))
  }, []); 

  return (
    <div className="App">
      <h1>Hello World</h1>
      <h2>產品列表</h2>
      <ul>
        {products.map(product => (
          <li key={product.id}>{product.name} - ${product.price}</li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```

## 11. 執行 docker-compose + 觀看頁面

上面都建立好後，就可以執行 docker 了！

```shell
$ docker-compose up --build
```

等一陣子之後，輸入此連結 - `http://localhost:3000/`，應該就可以看到 `Hello World` 和所有的 `產品資料` 了



ps. 如果今天想要先刪掉所有 docker 相關，可以用這個指令 - `docker-compose down`


