---
sidebar_position: 3
---

## part3 - 在首頁印出 Product 詳細內容

這邊我們即將要做的事情就是，讓所有產品印在頁面上，並且把可以點擊進個別產品，看到的詳細內容

## 1. 新增 api

### 修改 views.py
修改 backend/api/views.py 文件：

```py
from django.http import JsonResponse
from .models import Product

# 省略

def get_product(request, pk):
    try:
        product = Product.objects.get(pk=pk)
        data = {
            'id': product.id,
            'name': product.name,
            'price': product.price
        }
        return JsonResponse({'product': data})
    except Product.DoesNotExist:
        return JsonResponse({'error': 'Product not found'}, status=404)
```


### 修改 urls.py
修改 backend/ecommerce/urls.py 文件：
```py
from django.contrib import admin
from django.urls import path
from api.views import get_products, get_product

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/products/', get_products, name='get_products'),
    path('api/product/<int:pk>', get_product, name='get_product')
]
```

### 修改前端的 App.js

這邊多新增了一個 `fetchProductDetails` 函式，透過 api 拿到個別產品的詳細資料，並且透過迴圈，當點擊個別商品時，會把點擊到的商品詳細內容印出來

```js
import React, { useState, useEffect } from 'react';
import './App.css';

function App() {
  const [products, setProducts] = useState([]);
  const [selectedProduct, setSelectedProduct] = useState(null);

  useEffect(() => {
    fetch('http://localhost:8000/api/products/')
      .then(response => response.json())
      .then(data => {
        if (data.products) {
          setProducts(data.products);
        }
      })
      .catch(error => console.error('Error:', error));
  }, []);

  const fetchProductDetails = (productId) => {
    fetch(`http://localhost:8000/api/product/${productId}/`)
      .then(response => response.json())
      .then(data => {
        if (data.product) {
          setSelectedProduct(data.product);
        }
      })
      .catch(error => console.error('Error:', error));
  };

  return (
    <div className="App">
      <h1>Hello World</h1>
      <h2>產品列表：</h2>
      {products.length > 0 ? (
        <ul>
          {products.map(product => (
            <li key={product.id}>
              {product.name} - ${product.price}
              <button onClick={() => fetchProductDetails(product.id)}>查看詳情</button>
            </li>
          ))}
        </ul>
      ) : (
        <p>正在加載產品...</p>
      )}

      {selectedProduct && (
        <div>
          <h3>產品詳情：</h3>
          <p>ID: {selectedProduct.id}</p>
          <p>名稱: {selectedProduct.name}</p>
          <p>價格: ${selectedProduct.price}</p>
        </div>
      )}
    </div>
  );
}

export default App;
```