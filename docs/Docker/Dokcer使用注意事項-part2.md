---
sidebar_position: 3
---


Docker 使用注意事項 - part2
------


### 針對特定資料夾建立 image

```shell
$ image: docker build -t one:latest -f project-one/Dockerfile .
```


### 用剛剛的 image 建立 container

```shell
$ docker run -it --name project-one -p 8080:80 one:latest
```

### 進入 container 中

```shell
$ docker container exec --interactive --tty `{容器ID}` sh
```


### 刪除 container 
```shell
$ docker container rm --force 7a335820d67c
```


### 刪除 image
```shell
$ docker rmi -f 1e502fa756a3
```


## 不同內容的 image 建制出來的基礎 container，裡面的資料夾會有哪些不同

以下面兩種來做基本解析，分別用 `FROM nginx` 、 `FROM python:3.9-slim` 來查詢差異，分別以中兩種的 dockerfile 建立 image 後，並執行 container，接著分別進入容器中輸入 `ls`，可以發現兩者有不一樣的地方

### FROM nginx

```shell
test123: ls

---bin  boot  dev  docker-entrypoint.d  docker-entrypoint.sh  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

### FROM python

```shell
test123: ls

bin  boot  dev  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```


## WORKDIR 設定初始資料夾
可以使用此指令 `WORKDIR /srv` 來讓我們一開始就在 `srv` 這個資料夾


## COPY 指令介紹

這個指令分兩個部分，左邊路徑是本地端的檔案路徑，右邊的路徑是 docker 內部的檔案路徑

以此指令為例：`COPY project-one/index.html /srv/`，我要複製本地端 - project-one 資料夾裡面的 index.html 檔案，到 docker 裡面的 srv 資料夾中
另外一個範例：`COPY project-one/* /srv/`，我要複製本地端 - project-one 資料夾裡面的所有檔案，到 docker 裡面的 srv 資料夾中
最後一個範例：`COPY project-one/* .`，我要複製本地端 - project-one 資料夾裡面的所有檔案，到 docker 裡面的 跟目錄資料夾中 - ex. 如果今天你有先下這個指令 `WORKDIR /srv`，那本地端的所有資料夾都會匯到 `/srv` 中


## 如何使用 docker 在畫面上印出 hello world

設定好 dockerfile 後，明明指令都設定好，產生了 image 和 container ，打開 `http://localhost:8080/` 連結時，卻發現畫面上不是我們 `index.html` 的內容，原因是什麼呢？

### 路徑問題

我們先進到docker 裡面，並輸入此指令 `cat /etc/nginx/conf.d/default.conf`，這個指令會顯示目前 server 的一些設定   
只是找到裡面這一段，可以發現他預設的 root 跟我們的 root 路徑不一樣，因為我們現在的 root 已經改成 /srv 了，只要照著下面更改就可以     

```shell
test123: cat /etc/nginx/conf.d/default.conf

------
      server {
        ...
        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        } 
        ...
      }
      server {
        ...
        location / {
            root /srv;          // 改成這樣
            index index.html;
        }
        ...
      }
```

## 安裝 nano

由於 docker 正常來說是沒有 nano 或者 vim 這些編輯器，因此我們現在來在 docker 裡面新增他們

```shell
test123: apt update
test123:  apt install nano
```



### 使用 nano

進入檔案: 

```shell
$ nano /etc/nginx/conf.d/default.conf
```

進到檔案後，更改前面說的 `root` 就可以了！

### 重新啟動 nginx 

最後再重新打開container，並輸入指定的 port，就可以成功看到你的 index.html 內容

```shell
test123: service nginx restart
```

Ps. 要記得，由於 image 和 本地端是完全隔離的，所以以目前的設定，當你修改本地端的 index.html 時，是不會影響到 docker 裡面的 index.html 的

### nano 操作

#### 移動光標：

```md
使用箭頭鍵：上、下、左、右箭頭鍵可以移動光標。
使用 Ctrl + B：向後移動一個字符。
使用 Ctrl + F：向前移動一個字符。
使用 Ctrl + P：向上移動一行。
使用 Ctrl + N：向下移動一行。
```

#### 編輯文本： 

```md
直接輸入：直接使用鍵盤輸入文字。
使用 Delete 或 Backspace：刪除光標前的字符。
使用 Ctrl + D：刪除光標所在位置的字符。
使用 Ctrl + K：刪除光標所在位置到行尾的內容。
```
    
#### 保存和退出： 
    
```md
使用 Ctrl + O：保存文件。按下後，按 Enter 鍵確認文件名。
使用 Ctrl + X：退出 nano。如果文件已經被修改，nano 會提示你是否要保存。
```    

#### 搜索和替換： 
```md
使用 Ctrl + W：開始搜索。輸入要搜索的內容，按下 Enter。
使用 Ctrl + Shift + \\：開始替換。輸入要替換的內容，再輸入替換的內容，按下 Enter。
```


#### 其他操作

```md
使用 Ctrl + G：顯示幫助信息。
使用 Ctrl + C：顯示當前光標所在位置的行號和列號。
```
注意：在 nano 中，指令是以 Ctrl 鍵結合其他按鍵，例如 Ctrl + O 表示按住 Ctrl 鍵，然後同時按下 O。
完成編輯後，使用 Ctrl + X 退出 nano。如果有做過修改，nano 會詢問是否要保存。按下 Y 確認保存，然後按 Enter。    


## 如何讓 docker 檔案和本地端檔案保持同步更新

一定有人想要再開發的時候，讓本地端的檔案同步更新上 docker 中，這樣就不用一直重新建立 image，不過這樣要怎麼做呢？

### bind 語法

介紹以下指令，其他都差不多，只是多了這一段：-v '本地端檔案位置':'docker目錄位置' - -v /Users/yee0526/Desktop/docker/project-one:/srv

```shell
$ docker run -it --name check-expire-product-bind -p 8080:80 -v /Users/yee0526/Desktop/docker/project-one:/srv check-expire-product:latest
```

這樣就可以同步更新 index.html 檔案了
