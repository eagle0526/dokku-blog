---
sidebar_position: 1
---

## part1 - 制定好資料夾架構

## 1. 創建專案的目錄結構

初始資料夾架構，先創建基本資料夾 + 新增前後端的資料夾，以下是指令：

```shell
$ mkdir ecommerce-project
$ cd ecommerce-project
$ mkdir backend frontend
```

輸入完後，我們就會得到以下架構的資料夾
```md
| ecommerce-project
  |  backend
  |  frontend
```


## 2. 設定後端資料夾架構

```shell
$ cd backend
$ django-admin startproject ecommerce .
$ django-admin startapp api
```

Ps1. `startproject` 是產生基礎的文件
Ps2. `startapp` 是新增多種不同的 `app`

這兩個指令輸入後，你的資料夾架構會變這樣

```md
| ecommerce-project
  |  backend
   -- | api
      -- | migration
      -- | __init__.py
      -- | admin.py
      -- | apps.py
      -- | models.py
      -- | tests.py
      -- | views.py
   -- | ecommerce
      -- | __init__.py
      -- | asgi.py
      -- | settings.py
      -- | urls.py
      -- | wsgi.py
      -- | manage.py
   -- | manage.py      
  |  frontend
```

## 3. 設定前端資料夾架構

我這邊前端使用的是 `react`，因此等等會創建 react 的資料夾
```shell
$ cd ../frontend
$ npx create-react-app .
```
輸入以上指令後，檔案架構會變成這樣：

```md
| ecommerce-project
  |  backend
   -- | api
      -- | migration
      -- | __init__.py
      -- | admin.py
      -- | apps.py
      -- | models.py
      -- | tests.py
      -- | views.py
   -- | ecommerce
      -- | __init__.py
      -- | asgi.py
      -- | settings.py
      -- | urls.py
      -- | wsgi.py
      -- | manage.py
   -- | manage.py      
  |  frontend
   -- | node_modules
   -- | public
   -- | src
      -- | App.css
      -- | App.js
      -- | index.css
      -- | index.js
      -- | logo.svg
      -- | reportWebVitals.js
      -- | setupTests.js
   -- | package.json
   -- | package-lock.json
   -- | README.md
```


## 4. 安裝 Docker 文件

由於我們現在做前後端分離，因此會需要在前端和後端的資料夾位置，各設定 dockerfile，因此會像這樣

### docker-compose.yml

首先在 `最外層` 的地方，新增整個專案的 docker 檔案，我們最後會直接執行 `docker-compose up` 來建立 image 和 container：

```md
| ecommerce-project
  |  backend
      <!-- 忽略 -->
  |  frontend
      <!-- 忽略 -->  
  |  `docker-compose.yml`   -> 新增在這邊
```

* docker-compose.yml 檔案內容

```yaml
services:
  backend:
    build: ./backend
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - ./backend:/app
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      - DATABASE_URL=postgres://postgres:postgres@db:5432/ecommerce

  frontend:
    build: ./frontend
    command: npm start
    volumes:
      - ./frontend:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    depends_on:
      - backend

  db:
    image: postgres:13
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=ecommerce
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres

volumes:
  postgres_data:
```


### backend dockerfile

接著我們到 `backend` 的資料夾裡面新增 `dockerfile` 和 `requirements`：

```md
| ecommerce-project
  |  backend
   -- | api
      <!-- 忽略 -->  
   -- | ecommerce
      <!-- 忽略 -->  
   -- | manage.py
   -- | `Dockerfile`              -> 新增 dockerfile
   -- | `requirements.txt`        -> 新增 requirements
  |  frontend
      <!-- 忽略 -->  
  |  `docker-compose.yml`
```
* Dockerfile 檔案內容

```Dockerfile
FROM python:3.9

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 8000
```

* requirements 檔案內容

這個檔案是因為我們要安裝整個專案會用到的 package

```txt
Django>=4.2.2
psycopg2-binary>=2.9.3
```

### frontEnd dockerfile

接著我們到 `frontend` 的資料夾裡面新增 `dockerfile`：

```md
| ecommerce-project
  |  backend
   -- | api
      <!-- 忽略 -->  
   -- | ecommerce
      <!-- 忽略 -->  
   -- | manage.py
   -- | `Dockerfile`
   -- | `requirements.txt`
  |  frontend
      <!-- 忽略 -->  
   -- | `Dockerfile`              -> 新增 dockerfile      
  |  `docker-compose.yml`
```

* Dockerfile 檔案內容

```Dockerfile
FROM node:14

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm install

COPY . .

EXPOSE 3000
```




