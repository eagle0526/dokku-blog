---
sidebar_position: 6
---

## part6 - 會員系統 part1: 新增 User Model - Agent、Customer

接著我們來新增會員系統，我們總共有 2 個 Model 要設定，一個是 Agent (代購主)，一個是 (想要代購的人)，今天我們就來新增這兩個 Model


## 1. 修改 model.py 

* 這邊新增兩個繼承自 Django 預設的 User Model，分別用 `一對一關聯` 的方式連接
* 繼承自 Model 後，在各自的 Model 新增會用到的 element
* 要找到 `Agent` 或是 `Customer` 的資訊，可以使用 `related_name`，像這樣：`user.agent_profile 或 user.customer_profile`

```py
# backend/api/model.py

from django.db import models
from django.contrib.auth.models import User

class Product(models.Model):
    # 省略

class Agent(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE, related_name='agent_profile')
    commission_rate = models.DecimalField(max_digits=5, decimal_places=1, default=0)
    rating = models.DecimalField(max_digits=3, decimal_places=1, default=0)
    specialties = models.CharField(max_length=255, blank=True)
    phone_number = models.CharField(max_length=15, blank=True)

    def __str__(self):
        return f"Agent: {self.user.name}"

class Customer(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE, related_name='customer_profile')
    preferred_categories = models.CharField(max_length=255, blank=True)
    loyalty_points = models.IntegerField(default=0)
    phone_number = models.CharField(max_length=15, blank=True)    
    
    def __str__(self):
        return f"Customer: {self.user.username}"
```



### 創建 migration

```shell
$ docker-compose run backend python manage.py makemigrations
$ docker-compose run backend python manage.py migrate
```


## 新增 User資料

接下來我們來新增測試資料

### 新增 create_sample_users.py

在 backend/api/management/commands/ 目錄下創建一個新文件 create_sample_users.py：


```md
| ecommerce-project
  |  backend
   -- | api
       -- | management
           -- | commands
               -- | `create_sample_products.py`
               -- | `create_sample_users.py`          -> 新增這個
      <!-- 忽略 -->
  |  frontend
      <!-- 忽略 -->  
  |  docker-compose.yml
```

### 修改 create_sample_users.py 內容

* 用迴圈增加不同的使用者名稱
* 先新增好 User 物件後，

```py
# create_sample_users.py

from django.core.management.base import BaseCommand
from django.contrib.auth.models import User
from api.models import Agent, Customer
from django.db import transaction
import random

class Command(BaseCommand):
    help = 'create sample users, agents, and customers'  
    
    @transaction.atomic    
    def handle(self, *arg, **kwargs):
        for i in range(5):
            username = f'agent{i+1}'
            user = User.objects.create_user(
                username=username,
                email=f'{username}@example.com',
                password='password123',
            )
            Agent.objects.create(
                user=user,
                commission_rate=random.uniform(5.0, 10.0),
                rating=random.uniform(3.0, 5.0),
                specialties=random.choice(['Electronics', 'Fashion', 'Books', 'Home Goods']),
                phone_number=f'1234567{i:03d}'
            )
            self.stdout.write(self.style.SUCCESS(f'Created agent: {username}'))
        
        for i in range(10):
            username = f'customer{i+1}'
            user = User.objects.create_user(
                username=username,
                email=f'{username}@example.com',
                password='password123',                
            )
            Customer.objects.create(
                user=user,
                preferred_categories=random.choice(['Electronics', 'Fashion', 'Books', 'Home Goods']),
                loyalty_points=random.randint(0, 1000),
                phone_number=f'1234567{i:03d}'                
            )
            self.stdout.write(self.style.SUCCESS(f'Successfully created customer: {username}'))
        
        self.stdout.write(self.style.SUCCESS('Successfully created sample users, agents, and customers'))        
```

### 執行創建測資指令

```shell
$ docker-compose run backend python manage.py create_sample_users
```


## 介紹 Django 預設的 User Model

Django 的內建 User 模型提供了一組標準的欄位，用於處理用戶認證和基本信息。以下是 Django User 模型的主要欄位：

1. username：用戶名（必填，唯一）: 最大長度 150 字符，可以包含字母、數字和 @/./+/-/ 字符
2. password：密碼（必填）: 存儲為哈希值
3. email：電子郵件地址（可選，但建議填寫）
4. first_name：名字（可選），最大長度 150 字符
5. last_name：姓氏（可選）: 最大長度 150 字符
6. is_active：表示用戶賬戶是否活躍（布爾值）: 默認為 True
7. is_staff：表示用戶是否可以訪問管理後台（布爾值）: 默認為 False
8. is_superuser：表示用戶是否具有所有權限（布爾值）: 默認為 False
9. date_joined：用戶創建賬戶的日期和時間 : 自動設置為創建用戶時的時間戳
10. last_login：用戶最後一次登錄的日期和時間
11. groups：用戶所屬的組（多對多關係） : 與 Group 模型關聯
12. user_permissions：用戶的特定權限（多對多關係） : 與 Permission 模型關聯

這些欄位涵蓋了基本的用戶管理需求，包括認證、授權和用戶信息存儲。使用 Django 的內建 User 模型，您可以直接利用這些欄位，並且可以通過創建關聯模型（如我們之前討論的 Agent 和 Customer 模型）來擴展額外的用戶信息。

