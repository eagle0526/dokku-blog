---
sidebar_position: 6
---

## part8 - 會員系統 part3: 新增 User 登入/登出 前端的 介面

剛剛新增好 User 的後端 API，現在我們只要把前端的 API 接上去就好



## 1. 新增 Login.js、Register.js 頁面

```md
| ecommerce-project
  |  backend
  |  frontend
      <!-- 忽略 -->  
   -- | src
      <!-- 忽略 -->  
      -- | Login.js
      -- | Register.js
```
## 2. 新增 Login.js 內容

### userState Hook
1. const [username, setUsername] = useState('');：定義了一個 username 狀態變數，初始值是空字串。setUsername 函數用來更新這個變數的值。
2. const [password, setPassword] = useState('');：同樣地，password 變數用來儲存使用者輸入的密碼。

### handleSubmit
1. 這是一個處理表單提交的異步函數，當使用者點擊「登入」按鈕時觸發
2. e.preventDefault();：這行用來防止表單的預設行為（例如刷新頁面）。
3. axios.post('http://localhost:8000/api/login/', { username, password })
3-1. 向後端發送 POST 請求，網址是 http://localhost:8000/api/login/，並且將 username 和 password 作為請求的主體內容發送。
3-2. 如果請求成功，會回傳使用者的登入資料並記錄在控制台中（console.log(response.data)），並顯示一個「登入成功！」的提示。

### return
1. 第一個輸入框：用來輸入信箱（email）。當使用者輸入時，會透過 onChange 事件更新 email 狀態變數。
2. 第二個輸入框：用來輸入密碼（password），同樣地，當使用者輸入時，password 變數會被更新。
3. 按鈕：當使用者點擊按鈕時，觸發 onSubmit 事件，也就是 handleSubmit 函數，來發送登入請求。

```js
// frontend/src/Login.js

import React, { useState } from 'react';
import axios from 'axios';

function Login() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      const response = await axios.post('http://localhost:8000/api/login/', {
        username,
        password
      });
      console.log(response.data);
      alert('登入成功！');
    } catch (error) {
      console.error('登入錯誤:', error.response.data);
      alert('登入失敗：' + error.response.data.error);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="電子郵件"
        required
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="密碼"
        required
      />
      <button type="submit">登入</button>
    </form>
  );
}

export default Login;
```



## 3. 新增 Register.js 內容

### useState Hook

1. username, email, password 這跟前面 login 那邊邏輯一樣
2. 比較特別的是 userType，因為他直接給他預設值 - `customer`

```js
import React, { useState } from 'react';
import axios from 'axios';

function Register() {
  const [username, setUsername] = useState('');
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [userType, setUserType] = useState('customer');

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      const response = await axios.post('http://localhost:8000/api/register/', {
        username,
        email,
        password,
        user_type: userType
      });
      console.log(response.data);
      alert('註冊成功！');
    } catch (error) {
      console.error('註冊錯誤:', error.response.data);
      alert('註冊失敗：' + error.response.data.error);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={username}
        onChange={(e) => setUsername(e.target.value)}
        placeholder="用戶名"
        required
      />
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="電子郵件"
        required
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="密碼"
        required
      />
      <select value={userType} onChange={(e) => setUserType(e.target.value)}>
        <option value="customer">客戶</option>
        <option value="agent">代購主</option>
      </select>
      <button type="submit">註冊</button>
    </form>
  );
}

export default Register;
```


## 4. 把這兩個頁面加進 App.js 中

```js
// frontend/src/App.js

import React from 'react';
import { BrowserRouter as Router, Route, Routes, Link } from 'react-router-dom';
import ProductList from './ProductList';
import ProductDetail from './ProductDetail';
import Register from './Register';
import Login from './Login';
import './App.css';

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
        </Routes>
      </div>
    </Router>
  );
}

export default App;
```

## 5. 把註冊和登入頁面連結放進 sidebar

```js
import React from 'react';
import { Link} from 'react-router-dom';
import './sidebar.css';

function Sidebar() {
  return (
    <div className="sidebar">
      <ul>
        <li><Link to="/">首頁</Link></li>        
      </ul>
      <div className="auth-buttons">
        <div>         
          <Link to="/login" className="login-button">登入</Link>
        </div>
        <div> 
          <Link to="/register" className="register-button">註冊</Link>        
        </div>
      </div>
    </div>
  )
}

export default Sidebar;
```

## 6. 安裝 axios 和 Django Rest Framework

由於我們會用到這邊 package，因此在前後端的 docker 和 requirement 來新增一下

### 後端要安裝的 package
```txt
Django>=4.2.2
psycopg2-binary>=2.9.3
django-cors-headers>=3.10.0                         
djangorestframework==3.14.0                   # 新增這個
```

### 前端要安裝的 package

```dockerfile
# ... 其他 Dockerfile 內容 ...

RUN npm install
RUN npm install react-router-dom
RUN npm install axios                         # 新增這個 

# ... 其他 Dockerfile 內容 ...
```

## 7. 重啟 docker-compose

```shell
$ docker-compose down
$ docker-compose up --build
```

到這邊重開本地端的連結，應該就可以成功看到 `註冊/登入` 功能，可以實際登入試試看





















