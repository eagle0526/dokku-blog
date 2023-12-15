---
sidebar_position: 2
---


Docker 使用注意事項
------




### 根據 dockerfile 建立 image 檔

```shell
$ docker build -t check-expire-product:latest -f src/cronjobs/check-expire-product/Dockerfile .
```

1. -t : tag => 幫 image 取名字
2. -f : file => 如果今天專案有多個 dockerfile ，可以像這樣針對指定路徑的 `dockerfile`


### 根據 image 檔案建立 container

```shell
$ docker run -it --name check-expire-product -p 8080:80 check-expire-product:latest
```

1. -i : interactive => keep STDIN open even if not attached -> 直接翻譯的意思是， 保持輸入模式 ，可以理解成容器之間保持互動狀態
2. -t : tty => Allocate a pseudo-TTY -> 直接翻譯的意思是， 分配一個虛擬的TTY ，那TTY是什麼？
3. -p 8080:80 : publish => 將主機的端口 8080 映射到容器的端口 80。這樣，當你訪問主機的 8080 端口時，它會被映射到容器內運行的應用程序的 80 端口。
4. --name => 就是幫容器取名字

:::danger
啟動容器的時候一定要加上 `-i`，要不然容器的狀態會一直處於 `exited`，就算想要啟動成 `running`，還是會一瞬間變回 `exited`
:::



### 停止容器
```shell
$ docker container stop `{容器ID}`
```


### 開始容器
```shell
$ docker container start `{容器ID}`
```


### 進入容器內部

```shell
$ docker container exec --interactive --tty `{容器ID}` sh  
```
Ps. 最後面的 sh 之後再解釋


### 刪除容器
```shell
$ docker container rm --force 02ab51e1fc28
```


### 刪除 image

```shell
$ docker rmi -f 1e502fa756a3
```


## poetry 使用

### 確認 poetry 安裝的所有套件

```shell
$ poetry show
```

### 確認 poetry 使否有安裝指定套件

```shell
$ poetry show django
```


### 進入 poetry 虛擬環境
```shell
$ poetry shell
```


### 在虛擬環境中重新安裝套件

```shell
$ poetry install
```



## 本地開發

這個指令可以把 `shared` 這個資料夾複製到 `當下開發` 檔案位置，由於要把之後要使用 `docker bind` 來本地開發，如果沒有用此語法，會導致 `shared` 資料夾沒有複製過來

```shell
$ make cron_job_dir=$(pwd)/src/cronjobs/check-expire-product/ move-shared-to-cron-job
```




### 本機檔案更新 -> 影響 docker 內部文件的指令 -> bind mount

```shell
$ docker run -it --name check-expire-product-bind -p 8080:80 -v /Users/yee0526/Desktop/TAGTOO/cron-jobs-v2/cron-jobs-v2/src/cronjobs/check-expire-product:/srv check-expire-product:latest
```
