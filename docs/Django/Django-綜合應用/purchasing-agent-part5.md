---
sidebar_position: 5
---

## part5 - 新增 sidebar page

這一章節我們來新增一個側邊欄


## 1. 新增 Sidebar.js 和 Sidebar.css 兩個資料夾

```md
| ecommerce-project
  |  backend
  |  frontend
      <!-- 忽略 -->  
   -- | src
      <!-- 忽略 -->  
      -- | Sidebar.js
      -- | Sidebar.css
```


## 2. Sidebar.js 內容

* 這個頁面會去接 `get_product` 的 api

```js
// frontend/src/Sidebar.js

import React from 'react';
import { Link } from 'react-router-dom';
import './Sidebar.css';

function Sidebar() {
  return (
    <div className="sidebar">
      <ul>
        <li><Link to="/">首頁</Link></li>
        <li><Link to="/products">產品列表</Link></li>
      </ul>
    </div>
  );
}

export default Sidebar;
```

## 3. Sidebar.css 內容


```js
// frontend/src/Sidebar.css

.sidebar {
  width: 200px;
  height: 100vh;
  background-color: #f0f0f0;
  padding: 20px;
  position: fixed;
  left: 0;
  top: 0;
}

.sidebar h2 {
  margin-bottom: 20px;
}

.sidebar ul {
  list-style-type: none;
  padding: 0;
}

.sidebar li {
  margin-bottom: 10px;
}

.sidebar a {
  text-decoration: none;
  color: #333;
}

.sidebar a:hover {
  color: #007bff;
}
```


## 4. 修改 App.js

* 把剛剛做好的 Sidebar 放進來

```js
import React from 'react';
import { BrowserRouter as Router, Route, Routes } from 'react-router-dom';
import ProductList from './ProductList';
import ProductDetail from './ProductDetail';
import Sidebar from './Sidebar';
import './App.css';

function App() {
  return (    
    <Router>      
      <div className="App">
        <Sidebar />
        <div className="main-content">
          <h1>Hello World</h1>
          <Routes>
            <Route path="/" element={<ProductList />} />          
            <Route path="/product/:id" element={<ProductDetail />} />          
          </Routes>
        </div>
      </div>
    </Router>
  );
}

export default App;
```


這樣就把側邊欄給做好了！