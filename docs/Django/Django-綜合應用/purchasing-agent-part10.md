---
sidebar_position: 10
---

## part10 - 會員系統 part5: 解決重整頁面後，自動登出的問題

前面設定好登入的不同狀態角色可以看到不同的內容，但是遇到另一個問題，就是重整後網頁會自動登出，我們來解決這件事情。

### 問題原因

遇到的問題是因為 Django 默認使用會話（session）來維護用戶的登入狀態，而前端應用沒有正確地處理這個會話。在單頁應用（SPA）中，我們通常使用 token 來維護用戶的登入狀態。讓我們來修改代碼以解決這個問題：

## 1. 後端修改

### requirement.txt 安裝 djangorestframework-simplejwt

```txt
Django>=4.2.2
psycopg2-binary>=2.9.3
django-cors-headers>=3.10.0                         
djangorestframework==3.14.0                 
djangorestframework-simplejwt==5.2.1          # 新增這個
```


### 修改 settings.py

1. 啟用 `simplejwt` 的驗證
2. `rest_framework_simplejwt.authentication.JWTAuthentication` 這個設定 DRF 使用 `SimpleJWT` 提供的 `JWTAuthentication` 類別來處理 API 認證，這意味著每次客戶端訪問 API 時，會期待請求攜帶 JWT

#### SIMPLE_JWT 設定
* 'ACCESS_TOKEN_LIFETIME': timedelta(minutes=60)：這行設定了 JWT 訪問權杖（Access Token）的有效期為 60 分鐘。當使用者成功登入並獲取一個 Access Token，這個 Token 只能有效使用 60 分鐘。超過這個時間後，使用者需要重新獲取一個新的 Token 才能繼續訪問受保護的 API。
* 'REFRESH_TOKEN_LIFETIME': timedelta(days=1)：這行設定了刷新權杖（Refresh Token）的有效期為 1 天。當 Access Token 過期時，使用者可以使用 Refresh Token 來獲取一個新的 Access Token，而不用重新登入。1 天之後，Refresh Token 也會過期，使用者需要重新登入來獲取新的 Token。



```py
from datetime import timedelta

INSTALLED_APPS = [
    # ...其他應用...
    'rest_framework',
    'rest_framework_simplejwt',
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    )
}

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=60),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=1),
}
```


### 修改 views.py

這邊的重點就是當登入的時候，我們新增了一個 `RefreshToken` 給他，這樣傳到前端的時候，就會有 `token` 可以判斷現在客戶的狀態是啥

```js
// backend/api/
from rest_framework_simplejwt.tokens import RefreshToken

// 忽略其他 api

@api_view(['POST'])
def user_login(request):
    email = request.data.get('email')
    password = request.data.get('password')

    try:
        user = User.objects.get(email=email)
        if user.check_password(password):
            refresh = RefreshToken.for_user(user)
            if hasattr(user, 'agent'):
                role = 'agent'
            elif hasattr(user, 'customer'):
                role = 'customer'
            else:
                role = 'unknown'
            return Response({
                'message': 'Login successful',
                'user': {
                    'id': user.id,
                    'username': user.username,
                    'email': user.email,
                    'role': role
                },
                'tokens': {
                    'refresh': str(refresh),
                    'access': str(refresh.access_token),
                }
            })
        else:
            return Response({'error': 'Invalid credentials'}, status=status.HTTP_401_UNAUTHORIZED)
    except User.DoesNotExist:
        return Response({'error': 'User not found'}, status=status.HTTP_404_NOT_FOUND)

@api_view(['GET'])
def get_user_profile(request):
    if request.user.is_authenticated:
        if hasattr(request.user, 'agent'):
            role = 'agent'
        elif hasattr(request.user, 'customer'):
            role = 'customer'
        else:
            role = 'unknown'
        return Response({
            'isAuthenticated': True,
            'user': {
                'id': request.user.id,
                'username': request.user.username,
                'email': request.user.email,
                'role': role
            }
        })
    else:
        return Response({
            'isAuthenticated': False
        })
```



## 2. 前端修改

現在有把剛剛後端設定好的 token 傳給前端設定

### UserContext.js

* 加上 `const token = localStorage.getItem('accessToken')` 、 `Headers` 的設定 和如果 fetch 失敗，就 remove 掉 `accessToken`

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
      const token = localStorage.getItem('accessToken')
      try {
        const response = await axios.get('http://localhost:8000/api/profile/', {
          headers: {
            'Authorization': `Bearer ${token}`
          }
        });        
        if (response.data.isAuthenticated) {
          setUser(response.data.user);
        }
      } catch (error) {
        console.error('Error fetching user info:', error);
        localStorage.removeItem('accessToken');
        
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

### Login.js

* 加入 setUser，確保登入的時候，可以取得 `accessToken`

```js
// frontend/src/Login.js

import React, { useState, useContext } from 'react';
import axios from 'axios';
import { UserContext } from './UserContext';
import { useNavigate } from 'react-router-dom';

function Login() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const { setUser } = useContext(UserContext)
  const navigate = useNavigate()

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      const response = await axios.post('http://localhost:8000/api/login/', {
        username,
        password
      });
      setUser(response.data.user)
      localStorage.setItem('accessToken', response.data.tokens.access);      
      console.log(response.data);
      alert('登入成功！');
      navigate('/');      
    } catch (error) {
      console.error('登入錯誤:', error.response.data);
      alert('登入失敗：' + error.response.data.error);
    }
  };

  // ... 其餘代碼保持不變 ...
}

export default Login;
```

### Sidebar.js

* 這邊一樣加上 `localStorage.removeItem('accessToken')`

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
      localStorage.removeItem('accessToken');
    } catch (error) {
      console.error('Logout error:', error);
    }
  };

  // ... 其餘代碼保持不變 ...
}

export default Sidebar;
```


這些修改將使用 JWT 令牌來維護用戶的登入狀態。登入後，令牌將被存儲在 localStorage 中，並在每次頁面加載時用於驗證用戶。這樣，即使在頁面重新加載後，用戶也能保持登入狀態。
請確保在實際部署時，使用 HTTPS 來保護令牌的傳輸，並考慮實現令牌刷新機制以增強安全性。

