---
sidebar_position: 6
---

## part9 - 會員系統 part4: 新增 User 登入後的狀態

剛剛新增好 User 登入和註冊的畫面，現在我們要來保持客戶登入後，他擁有的狀態


## 1. 新增 context 來管理用戶狀態 - UserContext

### 新增 UserContext.js

```md
| ecommerce-project
  |  backend
  |  frontend
      <!-- 忽略 -->  
   -- | src
      <!-- 忽略 -->  
      -- | UserContext.js
```

### UserContext

* `export const UserContext = createContext()` 創建了一個新的 `UserContext`，這是一個共享的狀態容器。當我們使用這個 Context 時，可以在 React 組件樹的不同層級之間傳遞資料，而不需要層層傳遞 props
### UserProvider

1. 這個組件用來提供 UserContext 的資料給它的子組件

1-1. useState(null)
* user 是一個狀態變數，初始值為 null，用來儲存目前使用者的資訊。
* setUser 是修改這個狀態的函數。

1-2. useState(true):
* loading 是用來追蹤是否正在從 API 獲取使用者資料。初始值設為 true，表示正在載入。
* setLoading 是修改這個狀態的函數

### useEffect

* useEffect 是一個 React 的 Hook，用來執行副作用（如 API 請求），類似於生命週期方法 componentDidMount。在這裡，它會在組件第一次渲染後執行，負責獲取使用者資訊
* fetchUserInfo 是一個異步函數，使用 axios 發送一個 GET 請求到後端 (http://localhost:8000/api/profile/)，來獲取使用者資訊
* 如果後端回應表示使用者已通過驗證 (response.data.isAuthenticated)，那麼它會透過 setUser 函數來更新 user 狀態變數
* 請求結束後，不管是否成功，都會把 loading 設為 false，表示載入結束

### UserContext.Provider
* UserContext.Provider 是 Context API 中的提供者，它會將 user、setUser 和 loading 傳遞給所有消費這個 Context 的子組件。
* {children} 是一個特性，表示這個組件可以包裹其他組件，並將 UserContext 的值提供給它們。這樣，所有包裹在 UserProvider 內的組件都可以透過 useContext(UserContext) 獲取 user、setUser 和 loading 的值。


```js
// frontend/src/UserContext.js

import React, { createContext, useState, useEffect } from 'react';
import axios from 'axios';

export const UserContext = createContext();

export const UserProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchUserInfo = async () => {
      try {
        const response = await axios.get('http://localhost:8000/api/profile/');
        if (response.data.isAuthenticated) {
          setUser(response.data.user);
        }
      } catch (error) {
        console.error('Error fetching user info:', error);
      }
      setLoading(false);
    };

    fetchUserInfo();
  }, []);

  return (
    <UserContext.Provider value={{ user, setUser, loading }}>
      {children}
    </UserContext.Provider>
  );
};
```

## 2. 更新 App.js 以使用 UserProvider

```js
import React from 'react';
import { BrowserRouter as Router, Route, Routes } from 'react-router-dom';
import { UserProvider } from './UserContext';
// ... 其他 import ...

function App() {
  return (
    <UserProvider>
      <Router>
        {/* ... 其他組件 ... */}
      </Router>
    </UserProvider>
  );
}

export default App;
```


## 修改 sidebar，把註冊登入的按鈕放上去

* 先匯入剛剛設定和的 UserContext
* 設定一個登出函式 `handleLogout`，當點下登入按鈕時，設定 `setUser(null)`

```js
import React, { useContext } from 'react';
import { Link } from 'react-router-dom';
import { UserContext } from './UserContext';
import axios from 'axios';
import './sidebar.css';

function Sidebar() {
  const { user, setUser } = useContext(UserContext);

  const handleLogout = async () => {
    try {
      await axios.post('http://localhost:8000/api/logout/');
      setUser(null);
    } catch (error) {
      console.error('Logout error:', error);
    }
  };

  return (
    <div className="sidebar">
      <ul>
        <li><Link to="/">首頁</Link></li>        
      </ul>
      <div className="auth-buttons">
        {user ? (
          <div>
            <span>歡迎, {user.username}!</span>
            <button onClick={handleLogout}>登出</button>
          </div>
        ) : (
          <>
            <div><Link to="/login" className="login-button">登入</Link></div>
            <div><Link to="/register" className="register-button">註冊</Link></div>
          </>
        )}
      </div>
    </div>
  );
}

export default Sidebar;
```


## 新增 UserProfile.js 的頁面
現在來多新增一個使用者登入介面，不能角色登入後，進入該介面看到的內容是不同的


### 新增 UserProfile.js
```md
| ecommerce-project
  |  backend
  |  frontend
      <!-- 忽略 -->  
   -- | src
      <!-- 忽略 -->  
      -- | UserProfile.js
```

### 新增 UserProfile.js 的內容

```js
import React, { useContext } from 'react'
import { Link } from 'react-router-dom'
import { UserContext } from './UserContext'

function UserProfile() {
  const { user } = useContext(UserContext)

  return (
    <div>
      <h2>使用者個人資料頁面</h2>
      {user ? (
        <p>
          我是 {user.role === 'agent' ? '代購團主' : user.role === 'customer' ? '消費者' : '未知角色'}
          <div>
            {user.username}
          </div>          
        </p>
      ) : (
        <p>請先登入</p>
      )}
    <Link to="/">返回首頁</Link>
    </div>
  )
}

export default UserProfile;
```


### 在 App.js 註冊此頁面

```js
// frontend/src/App.js

import React from 'react';
import { BrowserRouter as Router, Route, Routes, Link } from 'react-router-dom';
import ProductList from './ProductList';
import ProductDetail from './ProductDetail';
import Register from './Register';
import Login from './Login';
import './App.css';
import UserProfile from './UserProfile';

function App() {
  return (
    <Router>
      <div className="App">
        <Sidebar />
        <Routes>
          <Route path="/" element={<ProductList />} />
          <Route path="/product/:id" element={<ProductDetail />} />
          <Route path="/register" element={<Register />} />
          <Route path="/login" element={<Login />} />
          <Route path="/profile" element={<UserProfile />} />             // 這一行
        </Routes>
      </div>
    </Router>
  );
}

export default App;
```