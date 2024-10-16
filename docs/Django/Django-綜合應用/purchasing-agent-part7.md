---
sidebar_position: 6
---

## part7 - 會員系統 part2: 新增 User 登入/登出 後端的 API

剛剛新增好 User 的 Model，現在來把 `註冊/登入/登出` 的 API 新增好


## 1. 修改 views.py 


### 註冊 api - user_register
* 取得前端傳過來的所有 request
* 先判斷此 email 之前是否存在
* 在判斷這個使用者要新增的是 customer 還是 agent

```py
# backend/api/views.py

from django.contrib.auth import authenticate, login, logout
from django.contrib.auth.models import User
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response
from .models import Customer, Agent


# 忽略

@api_view(['POST'])
def user_register(request):
    username = request.data.get('username')
    email = request.data.get('email')
    password = request.data.get('password')
    user_type = request.data.get('user_type')

    if User.objects.filter(email=email).exists():
        return Response({'error': 'User already exists'}, status=status.HTTP_400_BAD_REQUEST)
    
    user = User.objects.create_user(username=username, email=email, password=password)

    if user_type == 'customer':
        Customer.objects.create(user=user)
    elif user_type == 'agent':
        Agent.objects.create(user=user)
    else:
        return Response({'error': 'Invalid user type'}, status=status.HTTP_400_BAD_REQUEST)    
```


### 登入 api - user_login

* 先使用 Django 內建的 function - `get_user_model` 得到 `User`
* 接著得到登入會輸入的 request，並且利用內建的 `check_password` 函式，確定使用者的 `email 和 password` 是不是對的上
* 確認這個 User 是 agent 還是 customer，並且最後 return 要傳給前端的所有資訊

```py
from django.contrib.auth import login, logout, get_user_model

# 忽略

User = get_user_model()

@api_view(['POST'])
def user_login(request):
    email = request.data.get('email')
    password = request.data.get('password')

    try:
        user = User.objects.get(email=email)
        if user.check_password(password):
            login(request, user)
            # 確定用戶角色
            
            if hasattr(user, 'agent'):
                role = 'agent'
            elif hasattr(user, 'customer'):
                role = 'customer'
            else:
                role = 'unknown'
            return Response({
                'message': 'Login Successful',
                'user': {
                    'id': user.id,
                    'username': user.username,
                    'email': user.email,
                    'role': role
                }
            })
        else:
            return Response({'error': 'Invalid credentials'}, status=status.HTTP_401_UNAUTHORIZED)
    except User.DoesNotExist:
        return Response({'error': 'User not found'}, status=status.HTTP_404_NOT_FOUND)
```


### 登出 api - user_logout

* 這個比較單純，直接使用預設的 `logout(request)` 就可以

```py
@api_view(['POST'])
def user_logout(request):
    logout(request)
    return Response({'message': 'Logout successful'})
```

### 使用者訊息 api - get_user_profile

```py
@api_view
def get_user_profile(request):
    user = request.user
    if hasattr(user, 'agent_profile'):
        role = 'agent'
    elif hasattr(user, 'customer_profile'):
        role = 'customer'
    else:
        role = 'unknown'

    return Response({
        'isAuthenticated': True,
        'user': {
            'id': user.id,
            'username': user.username,
            'email': user.email,
            'role': role
        }
    })
```

## 2. 修改 urls.py

剛剛在 views.py 現在把 url 改加上去

```py
from api.views import get_products, get_product, user_register, user_login, user_logout, get_user_profile, get_users

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/products/', get_products, name='get_products'),
    path('api/products/<int:pk>/', get_product, name='get_product'),
    path('api/users/', get_users, name='get_users'),
    # User authentication
    path('api/register/', user_register, name='user_register'),
    path('api/login/', user_login, name='user_login'),
    path('api/logout/', user_logout, name='user_logout'),    
    path('api/profile/', get_user_profile, name='get_user_profile')
]
```
