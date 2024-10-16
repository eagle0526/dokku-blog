---
sidebar_position: 4
---

## part4 - 新增 ProductList 頁面 + ProductDetail 頁面

前面把產品的資料都印出來了，但是都是印在同一頁面，現在要改成點擊商品後，會進到產品的個別頁面，裡面會有產品的詳細內容
這邊我們即將要做的事情就是，讓所有產品印在頁面上，並且把可以點擊進個別產品，看到的詳細內容

### 兩個頁面內容
* ProductList: 展示出所有產品頁面
* ProductDetail: 並且當使用者點擊進產品個別頁面後，裡面會顯示產品的詳細內容


## 1. 安裝 react-router-dom 插件

編輯 `frontend/Dockerfile`:

```Dockerfile
FROM node:14

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm install

# 安裝 react-router-dom
RUN npm install react-router-dom

COPY . .

EXPOSE 3000

# 如果使用 create-react-app，可以添加以下命令啟動
CMD ["npm", "start"]
```


## 2. 重新啟動 docker

```shell
$ docker-compose down
$ docker-compose up --build
```

## 3. 新增 ProductList.js 和 ProductDetail.js 兩個資料夾

```md
| ecommerce-project
  |  backend
  |  frontend
      <!-- 忽略 -->  
   -- | src
      <!-- 忽略 -->  
      -- | ProductList.js
      -- | ProductDetail.js      
```


## 4. ProductDetail.js 內容

* 這個頁面會去接 `get_product` 的 api

```js
// frontend/src/ProductDetail.js

import React, { useState, useEffect } from 'react';
import { useParams, Link } from 'react-router-dom';

function ProductDetail() {
  const [product, setProduct] = useState(null);
  const { id } = useParams();

  useEffect(() => {
    fetch(`http://localhost:8000/api/product/${id}/`)
      .then(response => response.json())
      .then(data => {
        if (data.product) {
          setProduct(data.product);
        }
      })
      .catch(error => console.error('Error:', error));
  }, [id]);

  if (!product) {
    return <div>加載中...</div>;
  }

  return (
    <div>
      <h2>{product.name}</h2>
      <p>價格: ${product.price}</p>
      <p>ID: {product.id}</p>
      <Link to="/">返回產品列表</Link>
    </div>
  );
}

export default ProductDetail;
```

## 5. ProductList.js 內容

* 這個頁面會去接 `get_products` 的 api

```js
// frontend/src/ProductList.js

import React, { useState, useEffect } from 'react';
import { Link } from 'react-router-dom';

function ProductDetail() {
  const [product, setProduct] = useState(null)  

  useEffect(() => {
    fetch('http://localhost:8000/api/products/')
      .then(response => response.json())
      .then(data => {
        if (data.products) {
          setProducts(data.products)
        }
      })
      .catch(error => console.error('Error: ', error))
  }, [])

  return (
    <div>
      <h2>產品列表：</h2>
      {products.length > 0 ? (
        <ul>
          {products.map(product => (
            <li key={product.id}>
              <Link to={`/product/${product.id}`}>{product.name}</Link> - ${product.price}
            </li>
          ))}
        </ul>
      ) : (
        <p>正在加載產品...</p>
      )}
    </div>    
  )
}
```


## 6. 修改 App.js

* 這邊我們改用 Router, Route 這些參數包住自己想要的頁面

```js
import React from 'react';
import { BrowserRouter as Router, Route, Routes, Link } from 'react-router-dom';
import ProductList from './ProductList';
import ProductDetail from './ProductDetail';
import './App.css';

function App() {
  return (
    <Router>
      <div className="App">
        <h1>Hello World</h1>
        <Routes>
          <Route path="/" element={<ProductList />} />
          <Route path="/product/:id" element={<ProductDetail />} />
        </Routes>
      </div>
    </Router>
  );
}

export default App;
```


## 7. 修改 index.js

* 這邊特別要注意去修改 index.js 這個檔案，因為我們現在改成用 Router 了，所以要把原本這個頁面的  `BrowserRouter` 給拿掉，如果不拿掉，畫面上會是空的

```js
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```



### BrowserRouter 介紹
BrowserRouter 是 React Router 的一個 component，它為您的應用程序提供 route 功能。當您在應用程序中使用 BrowserRouter 時，它會創建一個 route 上下文，這個上下文包含了當前的路由狀態和一些用於導航的方法。
如果今天一個專案用了兩次 `BrowserRouter` 可能會發生以下問題：
* 重複的 route 上下文：當您在 `index.js` 和 `App.js` 中都使用 `BrowserRouter` 時，實際上創建了兩個獨立的路由上下文。這會導致內部的路由組件（如 Route、Link 等）無法正確工作，因為它們可能會與錯誤的上下文進行交互。
* `component` 層級問題：在 React 應用中，`component tree` 的結構非常重要。當您在 `index.js` 中使用 `BrowserRouter` 包裹 `App component` 時，App 及其所有 `sub component` 都會在同一個路由上下文中。但如果您在 `App.js` 中又使用了 `BrowserRouter`，就會創建一個新的、獨立的路由上下文，這個新的上下文與外層的上下文是斷開的。
* route 匹配失效：由於存在多個路由上下文，路由匹配可能會失效。例如，您在 App.js 中定義的路由可能無法正確匹配 URL，因為它們處於一個獨立的、與瀏覽器 URL 不同步的路由上下文中。
* 渲染問題：在某些情況下，多個 `BrowserRouter` 可能會導致組件渲染出現問題，甚至可能導致頁面完全空白，因為 route 系統無法正確決定應該渲染哪些 `component`

正確的做法是在整個應用中只使用一個 `BrowserRouter`