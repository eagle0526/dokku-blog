---
sidebar_position: 1
---


Docker 基礎架構
------

### Docker Daemon

Docker Daemon 負責監聽 Docker API 的請求，並且管理 Docker 的物件，像是映像檔(image)、容器(Container)、虛擬網路(Network)以及Volume，還可以和其他的 Docker Daemon 進行通訊，管理 Docker 整體服務。

### Docker Client

Docker Client 是大部分用戶和 Docker 互動的主要方式，當你使用 docker container run 指令的時候，Client 就會將這段指令透過 REST API 發送給 Docker Daemon，並由其執行背後程序，一個 Client 可以和一個以上的 Daemon 通訊。 

### Docker Registries

Docker Registries 是專門儲存映像檔的倉庫，DockerHub 則是任何人都可以用的公共倉庫(像是 GitHub )，Docker 本身預設就是在 DockerHub 上面尋找映像檔。

你也可以建立自己的私人倉庫，來存放屬於公司內部或是屬於自己的映像檔。當你使用 `docker pull` 或是 `docker container run` 指令時，所需的映像檔就會從設定好的倉庫中拉出，當你使用 `docker push` 指令時，映像檔就會被推送到你設定的倉庫中，沒有設定的情況下，倉庫都預設為 「DockerHub」


Docker 物件
------

因為 docker 就像樂高一樣，所以用物件來說明蠻適當的。先介紹一下有哪些物件

(1) 映像檔 - image
(2) 容器 - container
(3) 容積 - volume
(4) 虛擬網路 - networks


### 映像檔 image

映像檔本身是一個唯讀的樣板，搭配一長串的指令，在大部分情況下，一個映像檔是基於另外一個映像檔(大部分是使用官方映像檔)，並加上額外的一椰參數所建立。

舉例來說，你可以建立一個 Ubuntu 的映像檔，在裡面利用指令安裝任何你需要的套件，像是 VIM、GIT 等等，並打包成自己的映像檔，執行成容器時，就會有 VIM、GIT等套件的功能。

當然你也可以做專屬於你的映像檔，或是使用別人發佈在DockerHub上的映像檔，但要記得挑選有認證的，以防安全問題。

如果你要寫只屬於自己的映像檔，需要撰寫 `Dockerfile` ，後面會提到怎麼寫，利用一些語法來定義映像檔，以及執行成容器所需的步驟與工具。


### 容器 container

容器是映像檔的運作實體，可以透過 Docker Client 發送 API 來啟動、暫停、刪除容器，也可以將一個容器連接到一個以上的虛擬網路，甚至根據其當前的狀態，建制一個新的映像檔。

預設情況下，一個容器和其他容器及主機是相對隔離的，但可以透過控制容器的虛擬網路，來把其他容器加入相同的網路，改變之間的隔離程度。



### 容積 volume

volume 是一個非常重要的物件，本身運作於容器之外，確保容器刪除後的資料保存，而 volume 是儲存在主機上的，和容器本身的生命週期無關，這讓使用者可以輕鬆地在各個容器間共享檔案系統。

volume 有兩種使用方式，分別是 Volume 和 Bind Mount(掛載)，後面會說到。


### 虛擬網路 networks

Docker 強大的原因，在於容器間可以互相溝通，並將服務串連，亦或是將他們連接到非 Docker 的執行環境，容器本身甚至不需要知道自己是否被部署在 Docker 上面，都是靠著虛擬網路達成連線的功能。



Docker 指令格式
------


首先我們直接在終端機輸入 docker
```shell
$ docker

-------
Usage:  docker [OPTIONS] COMMAND
A self-sufficient runtime for containers

Options:
.....

Management Commands:
.....

Commands:
.....
```

主要會看到這幾個資訊：

### options 

options 主要是一些 Docker 的全域設定


### Management

我們用 `docker container run` 指令來解說：


```shell

$ docker container run ...

# 上面的 container 是被操作的物件，也就是 Management Command
# run 則是對物件進行操作，也就是 Command
```

Management Command 本身是可以被操作的 Docker 物件，Command 則是對該物件的執行動作，上面的例子就是執行一個容器。


```shell

$ docker network create ...

# network 是被操作的物件，也就是 Management Command
# create 則是對物件進行操作，也就是 Command
```

上面的例子就是建立一個虛擬網路


### docker helper

如果今天忘記一些指令，可以這樣輸入，這樣會有一大堆提示、該輸入哪些參數可以看
```shell
$ docker container run --help

-------
Usage:  docker container run [OPTIONS] IMAGE [COMMAND] [ARG...]
Run a command in a new container
Options:
.....
```


容器生命週期
------


### 啟動容器

```shell
$ docker container run --publish 80:80 nginx

------
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
ebc3dc5a2d72: Pull complete 
b8dc57082e5d: Pull complete 
0c203f625277: Pull complete 
5ff69ab3adbc: Pull complete 
......
```

### 退出非背景執行容器

按下 `Ctrl + C` ， 就可以退出容器



### 列出執行中的容器

```shell
$ docker container list

------
CONTAINER ID   IMAGE          COMMAND                  CREATED        STATUS         PORTS                    NAMES
19f0b5256750   mariadb:10.4   "docker-entrypoint.s…"   2 months ago   Up 6 minutes   0.0.0.0:3306->3306/tcp   summary-interview-db-1
```

Ps. 如果今天退出所有 docker 的話，這邊會是空的

### 列出包含退出狀態的容器

```shell
$ docker container list --all

-------
CONTAINER ID   IMAGE          COMMAND                  CREATED        STATUS         PORTS                    NAMES
19f0b5256750   mariadb:10.4   "docker-entrypoint.s…"   2 months ago   Up 6 minutes   0.0.0.0:3306->3306/tcp   summary-interview-db-1
02ab51e1fc28   nginx          "/docker-entrypoint.…"   About a minute Exited (0) ..                           cool_clarke
.....

```

### 啟動退出狀態的容器

如果今天容器是在退出狀態，我們不需要使用run，直接用start指令就好

```shell
$ docker container start {CONTAINER ID}

ex. docker container start 19f0b
```


### 退出背景執行的容器

如果今天是使用start來啟動容器，會發現用 Ctrl + c 會無法退出容器，因此要用這個指令來退出

```shell
$ docker container stop 02ab51e1fc28
```

### 刪除退出狀態的容器

刪除容器有兩種方法  

#### 先介紹如何刪除進入退出狀態的容器
```shell
$ docker container rm 02ab51e1fc28
```


#### 強制刪除容器

如果今天容器還在執行狀態，使用剛剛的刪除方法，會發現他不給你刪除，因此要改成這樣刪除

```shell
$ docker container rm --force 02ab51e1fc28
```


### 背景執行容器

一開始有提到，我們可以用 `docker container run` 來執行容器，那要怎麼在一開始執行容器的時候，就讓容器進入背景執行狀態呢？
```shell
$ docker container run --publish 80:80 --detach nginx
```



### 替容器命名

如果我們不替容器命名的話，每次產生的容器ID都是亂數產生的，這樣有點難精準控制我想要的容器，因此我們可以幫容器取名

```shell
$ docker container run --name good --publish 80:80 --detach nginx
```

這邊我們產生了一個叫做 `good` 的容器，我們來看看是否有生成這個容器

```shell
$ docker container list

------
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS          PORTS                    NAMES
f3bdbfbdbf14   nginx          "/docker-entrypoint.…"   8 seconds ago   Up 7 seconds    0.0.0.0:80->80/tcp       good
```


接著我們前面操作(停下、刪除)都是用容器ID來執行，現在我們用容器名字就可以達成了
```shell
# 停下容器
$ docker container stop good

# 刪除容器
$ docker container rm good  
```


### 觀看容器內的LOGS

如果今天容器是在背景執行，會看不到LOGS，因此我們可以這樣輸入來看LOGS：
```shell
$ docker container logs good
```
不過這樣只能單純把LOGS印出來，如果在開發時，想要持續追蹤LOGS發展，要這樣輸入：
```shell
$ docker container logs --follow good
```

進到這個狀況的時候，如果想要離開，只要按下 Ctrl + C 就可以離開。



### 啟動容器發生了什麼事

當我們在一開始輸入指令

```shell
$ docker container run --name good --publish 80:80 --detach nginx 
------
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
```

啟動nginx的時候，電腦就可以連上localhost了，究竟電腦做了哪些事情？

```md
這一行說明了，Docker在本地端找不到nginx:latest這個映像檔
> Unable to find image 'nginx:latest' locally

所以從 library/nginx 這裡拉取映像檔，也就是從預設的 DockerHub 拉取此檔案到本地端
> latest: Pulling from library/nginx
```


### 列出容器時的所有資訊

我們前面把所有容器列出來，有顯示很多資訊，我們來看一下細節
```shell
$ docker container list

-------
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS          PORTS                    NAMES
f3bdbfbdbf14   nginx          "/docker-entrypoint.…"   8 seconds ago   Up 7 seconds    0.0.0.0:80->80/tcp       good
```
(1) CONTAINER ID : 執行容器時，Docker 會賦予容器一個獨一無二的ID   
(2) IMAGE : 啟動時，指令中所指定的映像檔名稱   
(3) COMMAND : 容器啟動時的啟動指令，之後提到 `DOCKER映像檔` 時，會有更詳細的講解   
(4) CREATED : 何時啟動的容器   
(5) STATUS : 總共有 created、restarting、running、removing、paused、excited、dead等七種狀態，就是目前容器的狀態
(6) PORTS : 啟動時，指令中指定容器的PORT該對應到本機的哪個PORT
(7) NAMES : 容器名稱，如果沒取名，DOCKER會隨機分配取名


### 如何重新啟動容器，而不要在背景執行

我們剛剛用start開啟已經存在的容器時，系統會預設用背景執行
```shell
$ docker container start good
```

那要怎麼樣開啟時，想要直接顯示LOGS呢？我們可以用attach的方式
```shell
$ docker container start --attach good
```


### 容器生命週期演練


1. 在背景執行三個不同的服務，分別是nginx、postgres、httpd(apache)，並且分別給容器命名
2. nginx 要執行在 80:80、postgres 要執行在 5432:5432、httpd 要執行在 8080:80
3. 當啟動 postgres 容器時，需要給予環境變數 --env POSTGRES_PASSWORD=mysecretpassword，才能正確啟動
4. 使用 docker container logs 指令，來確認服務有正常啟動
5. 停止並刪除三個容器
6. 用 docker container list --all 確認所有都刪掉


```shell
$ docker container run --name good --publish 80:80 --detach nginx
$ docker container run --name bad --publish 5432:5432 --env POSTGRES_PASSWORD=mysecretpassword --detach postgres
$ docker container run --name normal --publish 8080:80 --detach httpd
```

查看該docker的log
```shell
$ docker container logs bad

$ docker container logs --follow bad
```


停止這個docker
```shell
$ docker container stop bad
```


刪除這個docker
```shell
$ docker container rm bad
```

一探容器內部
------

### 透過指令進步內部

透過下方的指令，我們會從一個終端機，進到另一個終端機，輸入後，我們會看到這個東西

```shell
$ docker container run --interactive --tty nginx bash
----

root@7b6dcbac069a:/# 
```

#### 可以再#號後面輸入Linux指令試試
```shell
root@7b6dcbac069a:/# ls

-----
bin  boot  dev	docker-entrypoint.d  docker-entrypoint.sh  etc	home  lib  media  mnt  opt  proc  root	run  sbin  srv	sys  tmp  usr  var
```




#### 離開容器內部
```shell
root@7b6dcbac069a:/# exit
```



### 進入運行中的容器

先確認有想進入的容器有沒有在運行(mystifying_ganguly這個有在進行)
```shell
$ docker container list

-------
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS          PORTS                    NAMES
7b6dcbac069a   nginx          "/docker-entrypoint.…"   4 minutes ago   Up 11 seconds   80/tcp                   mystifying_ganguly
19f0b5256750   mariadb:10.4   "docker-entrypoint.s…"   2 months ago    Up 25 hours     0.0.0.0:3306->3306/tcp   summary-interview-db-1
```

對於正在運作中的容器，我們要輸入 `exec` 指令，才能進入
```shell
$ docker container exec --interactive --tty 7b6dcbac069a bash
```

這樣就可以進入正在運作的容器中，並且一樣可以執行指令

```shell
root@7b6dcbac069a:/# ls
bin  boot  dev	docker-entrypoint.d  docker-entrypoint.sh  etc	home  lib  media  mnt  opt  proc  root	run  sbin  srv	sys  tmp  usr  var

root@7b6dcbac069a:/# exit
exit
```

### 新參數介紹

#### --interactive : keep STDIN open even if not attached

直接翻譯的意思是， `保持輸入模式` ，可以理解成容器之間保持互動狀態

#### --tty : Allocate a pseudo-TTY

直接翻譯的意思是， `分配一個虛擬的TTY` ，那TTY是什麼？   
由於古早的電腦非常貴，一台電腦要分配給很多個電腦操作，當多個用戶自然會需要多台打字機對電腦進行輸入，TTY就是 `Teletypewriter` 的縮寫，簡單來說可以簡單地把終端機想像成TTY。   


#### exec 

exec是一個允許在執行中的 Docker 容器執行任何指令的指令，白話一點來說，就是他可以把指令傳遞到執行中的容器內，並要求容器執行指令。

#### bash

我們先用下這一行指令，這樣可以看到run後面可以接哪些參數  
這樣我們會發現 `IMAGE` 後面可以接 COMMAND 的參數，這個 COMMAND 就是容器執行後，會執行的指令  
```shell
$ docker container run --helper

-------
Usage:  docker container run [OPTIONS] IMAGE [COMMAND] [ARG...]
```


不過我們先來看之前寫的這一行
```shell
$ docker container run --interactive --tty nginx bash
```



為什麼我們之前寫的是 bash，這是因為映像檔在建置的時候，都會給一個CMD參數來當作容器啟動時的指令，若沒有額外輸入COMMAND，就會使用映像檔預設的啟動指令。  
而預設的 nginx 指令就是 `nginx -g daemon off` -> 這一行是啟動nginx的意思。   
  
而這邊我們改成用 `bash` 取代 `nginx -g daemon off` 這一串啟動指令，變成容器在啟動時，執行bash這個命令處理器。



### 在容器安裝套件

先產生一個新的 `ubuntu container` 並先離開
```shell
$ docker container run --interactive --tty --name ubuntu ubuntu

$ root@7a206a77f57d:/# exit
```

再來我們重新啟動 container
```shell
$ docker container start --interactive ubuntu
```

接著我們來安裝curl工具，並且測試容器是否會保存我們安裝的東西
```shell
$ root@7a206a77f57d:/# apt install -y curl

------
Reading package lists... Done
Building dependency tree... Done
...略
```

離開 container
```shell
$ root@7a206a77f57d:/# exit
```

因為離開了，所以容器現在是退出狀態，接著我們再次啟動ubuntu容器，如果curl可以使用，代表容器是有保存的功能

```shell
$ docker container start --interactive ubuntu
```



如果今天在docker遇到ubuntu安裝curl的問題，像下面這樣
```md
apt-get install curl

Reading package lists... Done
Building dependency tree       
Reading state information... Done
E: Unable to locate package curl
```

解決方法輸入這個就可以 - It is because there is no package cache in the image, you need to run:

```shell
$ apt-get update
```

解決連結 - https://stackoverflow.com/questions/27273412/cannot-install-packages-inside-docker-ubuntu-image



確定安裝好curl之後，就可以先離開container，再進去之後，輸入
```shell
$ root@7a206a77f57d:/# curl google.com 

------
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```

如果有成功叫出HTML檔案，代表docker確實有保存安裝的插件



容器與虛擬機
------


### 什麼是虛擬機

理論上的說法是，在電腦上透過 `Hypervisor` 的軟體，將作業系統和硬體與應用程式分開，這樣就可以將自己劃分為數個獨立的 `虛擬機器`。

白話一點的說法是，可以想像成一個管家(Hypervisor)，幫你把一個房子分成好幾間套房，每一個套房都有自己的衛浴設備和供電設施，誰也不求誰，但還是佔掉整個房子的容積
，而且像是水龍頭的流水量，也因為多條水管牽線導致流水量不順暢。


虛擬機佔據的硬體比較多，啟動速度也比較慢，是因為每個虛擬機都可以獨立執行自己的作業系統和應用程式，同時還可以分配到從 `Hypervisor` 所管理的原始資源。
這些資源包括記憶體、RAM、儲存設備...等。


```md
> 虛擬機示意圖


-------------------  -------------------  -------------------
| Virtual Machine |  | Virtual Machine |  | Virtual Machine |
|-----------------|  |-----------------|  |-----------------|
|   App A         |  |   App B         |  |   App C         |
|-----------------|  |-----------------|  |-----------------|
|                 |  |                 |  |                 |
|   Guest         |  |   Guest         |  |   Guest         |
|   Operating     |  |   Operating     |  |   Operating     |
|   System        |  |   System        |  |   System        |
|                 |  |                 |  |                 |
|-----------------|  |-----------------|  |-----------------|

-------------------------------------------------------------
|                                                           |
|                  Hypervisor                               |
|                                                           |
|-----------------------------------------------------------|

|-----------------------------------------------------------|
|                                                           |
|                  Infrastructure                           |
|                                                           |
|-----------------------------------------------------------|
```


### 什麼是容器

容器其實就是一個抽象的應用層，把程式碼和相依套件打包在一起，多個容器可以在同一台機器上運作，並且和其他容器共享作業系統的核心(這和虛擬機完全不同)。   
虛擬機是每一個都有自己的作業系統，容器在Linux作業系統上作為獨立的執行程序相較於虛擬機，能夠佔用更少的空間，又能夠處理更多的應用程式。    


```md
> 容器示意圖


|----------| |----------| |----------| |----------| |----------|
|          | |          | |          | |          | |          |
|          | |          | |          | |          | |          |
|   App A  | |   App B  | |   App C  | |   App D  | |   App E  |
|          | |          | |          | |          | |          |
|----------| |----------| |----------| |----------| |----------|
  

----------------------------------------------------------------
|                                                              |
|                  Docker                                      |
|                                                              |
|---------------------------------------------------------------
|--------------------------------------------------------------|
|                                                              |
|                  Host Operating System                       |
|                                                              |
|---------------------------------------------------------------

|---------------------------------------------------------------
|                                                              |
|                  Infrastructure                              |
|                                                              |
|---------------------------------------------------------------
```





:::tip
Docker 容器只是在 Linux 作業系統上的執行程序，並不是什麼迷你的虛擬機   
他能夠獲取的資源有限，在程序結束時，容器就會進入退出的狀態     
:::



### docker 容器只是在 Linux 作業系統上的執行程序

為了驗證 docker 只是一個在Linux 作業系統上的執行程序，我們先來啟動一個容器

```shell
$ docker container start ubuntu
```

再來使用top，來查看該容器的執行程序
```shell
# 這個指令的意思是，列出該容器的所有執行程序，可以看到列出的PID，也就是執行程序本身的ID
$ docker container top ubuntu

---------
UID            PID              PPID             C                STIME            TTY              TIME             CMD
root           2244             2217             0                10:25            ?                00:00:00         /bin/bash
```


那要怎麼驗證容器本身只是一個 Linux 作業系統上的執行程序呢？我們可以在 Linux 作業系統的終端機輸入 `ps aux` (列出系統所有執行程序資料)
這樣會看到容器內的PID和Linux作業系統的PID相同，這樣就可以驗證，其實 Docker 是跑在 Linux 的作業系統上，並非虛擬機的概念
如果今天是虛擬機，在當前的作業系統，是看不到執行程序的！

### 為什麼只提 Linux

會一直提 Linux 的原因，是因為 Mac 和 Windows 上的 Docker，是執行在迷你虛擬機上，所以當你在上述兩者作業系統中，列出所有執行程序，是沒辦法看到容器的執行程序

那要如何在 mac 上看到容器的執行序呢？我們需要先連上迷你的虛擬機，只要按照下方的指令，就可以看到虛擬機內的執行程序：
```shell
$ docker run -it --rm --privileged --pid=host justincormack/nsenter1
```

容器的IP和PORT
------




### 檢查容器的PORT

```shell
# 先在背景啟動一個nginx
$ docker container run --detach --publish 80:80 --name nginx nginx  

# 查看port
$ docker container port nginx

-------
80/tcp -> 0.0.0.0:80
```


### 0.0.0.0 和 127.0.0.1 在 Docker 中有哪裡不一樣

在 docker container port 回應中有看到 `0.0.0.0:80` ，這是因為容器本身沒有設定 IP 位置， Docker 預設為 `0.0.0.0`，和我們一般本機開發的 `127.0.0.1` 不太一樣，這代表什麼呢？

在 docker 世界， `127.0.0.1` 代表 `這個容器的本身`，而不是 `這台機器`，如果從一個容器向外連接到 `127.0.0.1` ，對於容器本身來說就像連到自己一樣。

如果今天有兩個容器要互相溝通，不小心把容器綁定成 `127.0.0.1` ，會沒辦法向外溝通，而 `0.0.0.0` 代表網路街口，他能接受來自其他容器的連接，以及外部的連接都能成功到達容器內。


### 容器的IP位置


我們前面都還沒談論過容器的IP位置，或許你會認為容器和主機都以相同的IP的位置執行，但其實不是，我們可以透過docker container inspect 指令來看到容器的 IP 位置：
```shell
$ docker container inspect --format '{{ .NetworkSettings.IPAddress }}' nginx

------
172.17.0.3
```

### 接著查看本機的IP位置

inet 10.8.8.147，這一段就是我們的本機IP位置

```shell
$ ifconfig en0

-------
en0: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
	options=6463<RXCSUM,TXCSUM,TSO4,TSO6,CHANNEL_IO,PARTIAL_CSUM,ZEROINVERT_CSUM>
	ether 1c:57:dc:45:4b:19 
	inet6 fe80::18ea:d978:4c81:d9ea%en0 prefixlen 64 secured scopeid 0xc 
	inet 10.8.8.147 netmask 0xffffff00 broadcast 10.8.8.255
	nd6 options=201<PERFORMNUD,DAD>
	media: autoselect
	status: active
```


Ps. 因此可以確定，本機和容器的IP位置是不同的



Docker 虛擬網路
------

### 電腦防火牆

防火牆一開始的意思，是古人使用木頭建造房屋時，為了避免火災蔓延，將堅固的石塊堆砌在房屋周圍作為屏障。     
而現代網路的意意思，是指隔離本機網路，與外界網路的一道防禦系統，藉由控制過濾限制訊息，來保護內部網路資料的安全。   

Ps. 防火牆預設阻止所有從網際網路中進來的流量，提到防火牆，就是希望透過著方式更理解Docker如何向網際網路打開大門，並讓外部請求進到容器內部。



### 打開防火牆

正常情況下，Docker預設會用一個叫做 `bridge` 的虛擬網路，我們可以透過以下指令來看：

```shell
$ docker network list

-------
NETWORK ID     NAME                       DRIVER    SCOPE
9a64695ce968   bridge                     bridge    local
65abba938f22   external-service-network   bridge    local
86299be73d50   host                       host      local
bd3c20ec4d10   none                       null      local
```

這個虛擬網路幫我們橋接了本機的網路介面，讓我們可以透過簡單的 --publish 參數，快速啟動容器，並對應到防火牆該開啟哪個port。
若是沒有指定虛擬網路的情況下，Docker會預設使用這個 `bridge` 的虛擬網路，



這裡剛好可以提到，為什麼nginx的容器預設不會被分配到和本機相同的IP位置呢？
因為沒有指定網路的話，容器會以Docker的bridge虛擬網路優先，並連接上去，所以當然不同的網路環境會顯示不同的IP位置




我們可以透過以下指令，查看bridge這個虛擬網路的IP：


```shell
$ docker network inspect bridge

------
[
    {
        "Name": "bridge",
        "Id": "9a64695ce968d37f5b622ceeb9516eb02a3ac821e17278e17d18151edf1810cd",
        "Created": "2023-04-19T10:18:50.301027583Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        ......省略
    }
]
```


可以發現這一行："Gateway": "172.17.0.1"，還記得我們前面查看的nginx IP是多少嗎？
```shell
$ docker container inspect --format '{{ .NetworkSettings.IPAddress }}' nginx

------
172.17.0.3
```


就像前面說的一樣，沒有指定虛擬網路的情況下，Docker 預設使用bridge這個虛擬網路，而Docker則替我們在bridge網路內分配IP位置。
但是即使容器被分配到不同的IP位置，這又和網際網路的請求，能夠進入容器內部有什麼關係呢？



### NAT 網路位址轉換

Docker 透過 `NAT` 這項技術讓外部的請求進入 Docker，並順著虛擬網路找到容器。


NAT 這項技術被廣泛用在許多公司的內部網路，或是私人企業的內網中，主要原因是IPv4的位置稀少，很多企業或網路公司在只有少數的IP位置情況下，公司內部卻有太多電腦要連接網路，故採取共用IP的解決方法，就是讓一個IP位置給多台電腦使用。


使用者上網後，拿到一個IP位置，而IP分享器或無線基地台，則將一組專門給內部使用的私有IP分配給所有的內部電腦，內部每台電腦擁有一個 `192.168.0.X` 的IP位置，但無線基地台對外卻只有一個由網路公司賦予的IP位置。


通常，NAT將每一部電腦所用的IP，對應到共用IP，且NAT負責將進出封包的HEADER進行轉換，使得內部電腦可以輕鬆的用外部網路連線溝通。


用上述的概念，我們可以想像Docker本身就是負責分配IP的機器，而在Docker上運作的容器都是公司內部的電腦，被分配一個IP位置，但主要外面的入口還是要通過Docker來處理底層的網路技術。



### 操作 Docker 虛擬網路

#### list 指令

```shell
$ docker network list

------
NETWORK ID     NAME                       DRIVER    SCOPE
9a64695ce968   bridge                     bridge    local
86299be73d50   host                       host      local
bd3c20ec4d10   none                       null      local
```

如果沒有特別建立虛擬網路的話，一開始應該只會看到這幾個虛擬網路。

(1) 第一個 bridge 是我們前面介紹 Docker 預設的虛擬網路，藉由 NAT 技術潛伏在主機的防火牆後方
(2) 第二個是 HOST 虛擬網路，這是一個特殊的服務，他跳過了 Docker 的虛擬網路，直接把容器連接到主機的網路介面上，這麼做有好處、也有壞處。壞處是安全性降低了，好處是可以提高網路的效能
(3) 最後一個是 none 虛擬網路，連到這個虛擬網路 = 沒有連到任何虛擬網路，有時候我們想要斷開某些曝光在網路的服務，可以暫時先將容器連接到 none 虛擬網路。


#### inspect 指令

`inspect` 是一個非常萬用的指令，不論搭配Docker哪先物件，都可以使用inspect指令來查看資訊

```shell
$ docker network inspect bridge   

------
[
    {
        "Name": "bridge",
        "Id": "9a64695ce968d37f5b622ceeb9516eb02a3ac821e17278e17d18151edf1810cd",
        "Created": "2023-04-19T10:18:50.301027583Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "42c5762b9b63674ff28ebb2021d2f3b2b0d341f14017943fa8f470ac3c046709": {
                "Name": "nginx",
                "EndpointID": "c431653f7de60d8e4ed823e5e243f4f8dcd3c21dae5d892029aaf8f77a6f5767",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "7a206a77f57dbff19428b8fa7b2f44639d138560607fbf30358bd5039a706084": {
                "Name": "ubuntu",
                "EndpointID": "d630a41211bca33964c600481f69610e65971a5b25e058383259ff325f2f18bf",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        ......省略
]
```

上面可以看到，Containers裡面有兩個容器連在bridge這個虛擬網路上。

### 建立自己的虛擬網路

app中的名字，可以取自己喜歡的
```shell
$ docker network create {app}

ex. docker network create newNetwork
```

在把list列出來，會發現剛剛新創的虛擬網路有在裡面
```shell
$ docker network list

------
NETWORK ID     NAME                       DRIVER    SCOPE
9a64695ce968   bridge                     bridge    local
86299be73d50   host                       host      local
bfad8604bfd2   newNetwork                 bridge    local
bd3c20ec4d10   none                       null      local
```



### 添加原先的容器到虛擬網路中


如果我們想要把先前創建的容器，新增到剛剛創的虛擬網路，要怎麼做呢？使用 `--network 參數`

```shell
# 先清除舊有的所有容器
$ docker container rm --force $(docker container list --all --quiet)

# 新增一個容器
$ docker container run --detach --publish 80:80 --network {app} --name nginx nginx
------
4b4cd504eca96331f4c1740ff5fe906bd15031800817970c35a979fa8d2ca4d1
```

接著我們來確認一下剛剛那個指令，我們有沒有把容器連上剛剛創的虛擬網路
```shell
$ docker network inspect newNetwork

------
[
    {        
        ...省略
        "Containers": {
            "4b4cd504eca96331f4c1740ff5fe906bd15031800817970c35a979fa8d2ca4d1": {
                "Name": "nginx",
                "EndpointID": "f742988d816920ec25db21a6a6d5922b9e3b0a34a4e7ce61a1c732e948b13e50",
                "MacAddress": "02:42:ac:13:00:02",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            }
        },
        ...省略

    }
]

```

確實有連上虛擬網路！但是在食物上，我們會需要替正在運作的容器，加進新的虛擬網路，這時候中斷容器服務，再重新使用 `--network` 的方式不太合理。


### 添加運作中的容器到新的虛擬網路


先新增一個postgres容器，並且讓他連接到預設bridge網路
Ps. 什麼都不設定，就是連到bridge
```shell
$ docker container run --detach --name pg --env POSTGRES_PASSWORD=password postgres
```

接著吧PG虛擬網路，跟剛剛新創的虛擬網路連接上去
Ps. 這個指令不會有反應，因為他沒有response
```shell
# newNetwork是虛擬網路的name，pg是容器的name
$ docker network connect newNetwork pg
```

那我們要怎麼驗證，這個容器有同時連接到兩個虛擬網路呢？一樣使用inspect就好了
```shell
$ docker container inspect pg

------
"Networks": {
    "bridge": {
        "IPAMConfig": null,
        "Links": null,
        "Aliases": null,
        "NetworkID": "9a64695ce968d37f5b622ceeb9516eb02a3ac821e17278e17d18151edf1810cd",
        "EndpointID": "c793fb4e1580a8f443a076502b81fad38852fd1b649bab122ac44e720a5395f8",
        "Gateway": "172.17.0.1",
        "IPAddress": "172.17.0.2",
        "IPPrefixLen": 16,
        "IPv6Gateway": "",
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "MacAddress": "02:42:ac:11:00:02",
        "DriverOpts": null
    ,
    "newNetwork": {
        "IPAMConfig": {},
        "Links": null,
        "Aliases": [
            "fe3ee07b5174"
        ],
        "NetworkID": "bfad8604bfd275abf2c53a0a04074bd9f3d1c6876b539bbef5cc73ef1f11a6a2",
        "EndpointID": "ce8f04560baa29bf8e983e440fda8389ab9a06591bae63e03a8d135f73b0c7e3",
        "Gateway": "172.19.0.1",
        "IPAddress": "172.19.0.3",
        "IPPrefixLen": 16,
        "IPv6Gateway": "",
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "MacAddress": "02:42:ac:13:00:03",
        "DriverOpts": {}
```



這樣看此容器確實有連接到兩個虛擬網路！

### 中斷虛擬網路的連接

此指令可以中斷容器和虛擬網路的連接
Ps. 一樣不會有response
```shell
$ docker network disconnect newNetwork pg
```

### driver 是什麼

我們在列出虛擬網路的list時候，會發現有一個DRIVER的欄位
```shell
$ docker network list

------
NETWORK ID     NAME                       DRIVER    SCOPE
9a64695ce968   bridge                     bridge    local
86299be73d50   host                       host      local
bfad8604bfd2   newNetwork                 bridge    local
bd3c20ec4d10   none                       null      local
```

看到上面的資料應該會覺得有一個地方很神奇，就是 `newNetwork` 也就是我們新創的虛擬網路，他的 `DRIVER` 也是用 `bridge`。
原因就是如果今天沒有指定driver的話，docker預設的driver就是bridge。

使用bridge當作driver的虛擬網路，通常適用在單主機的容器們需要互相溝通的情況。為什麼強調是單主機呢？因為後面會提到另外一個driver，就是 `overlay`，這種虛擬網路可以跨平台溝通。


Docker 的 DNS
------

在 docker 的世界，我們可以忘掉IP位置這件事情，因為前面提到過，容器的IP會由Docker做分配，每次容器啟動的時機不同，都會導致IP位置不一樣，所以想要透過IP位置來做容器間的相互溝通，是不太實際的。

因此DNS就是Docker的解決方案，當我們對一個容器命名後(--name)，在Docker虛擬網路中，我們為他的命名就是這跟服物的DNS，我們可以透過他的名字，訪問到容器。


### 用 IP 位置來溝通

我們先清空所有容器
```shell
$ docker container rm --force $(docker container list --all --quiet)
```

#### 創建第一個container
最後一個參數後面才會提到，先不用理會
```shell
$ docker container run --publish 3000:3000 --detach  --name whoami --network newNetwork robeeerto/whoami
```

上面輸入好後，我們打開瀏覽器，並輸入 `http://localhost:3000/` 後，會看到下面的資訊
```md
> 容器名稱：a9292f6ceb25
> 容器的 IP 位置：172.19.0.2
> 環境變數 AUTHOR 是：robertchang
```

這裡的容器名稱，就是我們剛剛新啟動容器的ID，而IP位置就是Docker幫容器分配的IP位置，第三個嘖IMAGE才會提到

#### 創建第二個container



```shell
$ docker container run --publish 3001:3000 --detach  --name whoami-2 --network newNetwork robeeerto/whoami
```

上面輸入好後，我們打開瀏覽器，並輸入 `http://localhost:3001/` 後，會看到下面的資訊

```md
> 容器名稱：b129236860f0
> 容器的 IP 位置：172.19.0.3
> 環境變數 AUTHOR 是：robertchang
```


現在，我們新創的虛擬網路 `newNetwork`，有兩個容器連上去了(whoami、whoami-2)，接著我們來讓兩個容器進行溝通，這邊我們先用IP位置的方法進行溝通：

Ps. 使用curl工具

```shell
# 先進去第一個容器
$ docker container exec --interactive --tty whoami sh
```

進入第一個容器的環境後，輸入下面這行
```md
> /app # curl 172.19.0.3:3000

------
> 容器名稱：b129236860f0<br>容器的 IP 位置：172.19.0.3<br>環境變數 AUTHOR 是：robertchang/app # 
```

會發現有成功溝通了，但是我們今天試試不同的流程看看，也就是重新把剛剛的流程重走一次，並且這一次要在中間穿插一個服務


#### (1) 先清空容器
```shell
$ docker container rm --force $(docker container list --all --quiet)
```

#### (2) 新增一個whoami容器

```shell
$ docker container run --publish 3000:3000 --detach --name whoami --network newNetwork robeeerto/whoami
```

#### (3) 新增一個postgres容器

```shell
$ docker container run --detach --name pg --env POSTGRES_PASSWORD=password --network newNetwork postgres
```

#### (4) 新增一個whoami-2容器

```shell
$ docker container run --publish 3001:3000 --detach --name whoami-2 --network newNetwork robeeerto/whoami
```


現在 `newNetwork` 虛擬網路中，有三個容器了，並且我們在兩個中間穿插了一個PG容器，現在我們再來試試用IP位置進行容器溝通

(1) 先進到第一個容器中
```shell
$ docker container exec --interactive --tty whoami sh
```
(2) 溝通到whoami-2容器
```md
> /app # curl 172.19.0.3:3000

------
> curl: (7) Failed to connect to 172.19.0.3 port 3000 after 5 ms: Connection refused 
```

輸入同樣的IP位置後，會發現whoami-2的IP位置變了，因為這個IP位置已經被PG拿走。


### 用 DNS 來溝通

以剛剛的當作範例，我們直接透過DNS來進行溝通

(1) 先進到第一個容器中
```shell
$ docker container exec --interactive --tty whoami sh
```

(2) 溝通到whoami-2容器
```md
> /app # curl whoami-2:3000

------
> 容器名稱：c442b528e93f<br>容器的 IP 位置：172.19.0.4<br>環境變數 AUTHOR 是：robertchang/app # 
```

會發現用DNS溝通可以成功！


### 為什麼是 port 3000

有注意到的人應該會很好奇，明明前面我們新增容器的時候，whoami-2的port明明是-publish 3001:3000，但是為啥我們剛剛由容器1，溝通到容器2的指令是curl 172.19.0.3:3000。     

首先我們可以回想一下，左邊的port代表的是什麼意思，代表的是這台機器上對應的port，而不是容器本身打開的port 3001:3000，以這個例子來說，機器上打開3001，但容器本身還是3000。     


而假設今天在相同的虛擬網路，whoami容器要找到whoami-2容器，並不需要進到網際網路，再回到虛擬網路中，而是直接通過容器本身所在的虛擬網路內的容器名稱(DNS)，，加上打開的PORT進行連線即可。


### 為什麼不用 bridge Network 呢

如果今天不指定虛擬網路的話，Docker 不是預設會以 bridge 的虛擬網路為主嗎？為什麼還要特地自己建立虛擬網路呢？
因為預設的 bridge 虛擬網路沒有內建 Docker DNS 功能，所以若是使用預設的虛擬網路，會發現剛剛用 DNS 連線的範例是連線不到 whoami-2 的。


### 使用 --link 方式讓兩個容器在預設的虛擬網路做溝通

現在我們不要讓兩個容器在我們剛剛新建的虛擬網路內，而是讓兩個在預設的虛擬網路內


```shell
# 第一個容器
$ docker container run --publish 3000:3000 --detach --name whoami robeeerto/whoami

# 第二個容器使用--link
$ docker container run --publish 3001:3000 --detach --name whoami-2 --link whoami robeeerto/whoami
```

再來進到容器二裡面
```shell
$ docker container exec --interactive --tty whoami-2 sh

------
/app # curl whoami:3000
容器名稱：fcce7320b68f<br>容器的 IP 位置：172.17.0.2<br>環境變數 AUTHOR 是：robertchang/app # 
```

這樣確實成功讓容器一和容器二互相溝通




Docker 映像檔
------

什麼是 `映像檔`，簡單來說，就是 `Docker執行容器時的說明書，並附上工具包的一個檔案`。
映像檔本身是透過一個叫做 `Dockerfile` 的檔案建製而成，而在 `Dockerfile` 中，我們可以一步步告訴Docker：「黑！照著這些步驟去執行，中間有些必要的套件是必要的，幫我在執行容器時，一起放進去吧！」。
映像檔不一定是一個作業系統，他只是單純在作業系統上的執行程序。

映像檔可以非常的迷你，小到只是一個檔案，像是 `Golang` 的應用程式在編譯完後，就是一個靜態的執行檔案。他也可以非常巨大，像是完整的 Ubuntu 作業系統，或是 PHP 的執行環境等。

### 從 DockerHub 認識映像檔


#### docker official image 

官方把映像檔放在DockerHub上，其目的是提供基本的作業系統(例如：Ubuntu、CentOS)或服務(例如：postgreSQL、redis等)，希望可以作為大多數使用者的起點，也為現在流行的程式語言提供類似
「平台即服務」的概念，像是「Ruby、Golang、Node」等，都有官方認可的映像檔。


官方映像檔也同時是Dockerfile的最佳實踐範例，提供非常清楚的說明，讓其他使用者在撰寫Dockerfile的時候，有一個很好的參考，也確保安全性和更新速度，這非常重要，因為官方映像檔基本就是DockerHub上最受歡迎的映像檔。



#### verified publisher

為所有開發者提供官方的驗證，這些映像檔來自可信賴的來源，減少了從不安全的儲存褲中拉取危險映像檔的風險，基本上只要有這個徽章，就是可信賴又安全的映像檔，也可以透過嚴格的安全審查害DockerHub官方申請這個徽章。

### 映像檔標籤

我們進到DockerHub後，點選Redis，可以看到裡面有滿滿的版本、作業系統的後綴。

這邊要提到一個慣例，就是如果沒有指定要下載哪個版本，通常會拉 `latest` 的版本下來，那要如何拉指定版本呢？

我們來使用下列的指令，並且在指令最後加上標籤號：

```shell
$ docker image pull redis:7.0
```


那我們現在拉一個7.0.4版本
```shell
$ docker image pull redis:7.0.4
```

我們執行這個指令的時候，會發現一件事，就是下載速度超級快，這是怎麼回事呢？我們打開映像檔看看

```shell
$ docker image list

------
REPOSITORY               TAG       IMAGE ID       CREATED        SIZE
redis                    7.0       84ed4fbf28f6   6 days ago     111MB
redis                    7.0.4     84ed4fbf28f6   7 months ago   111MB
```


會發現兩個 `IMAGE ID` 完全相同，因為這兩個是同一份映像檔，只是用不同的標籤顯示而已，這才觸發了Docker的快取機制，導致根本沒有下載的感覺，因為畚箕已經有一份一樣的映像檔了。   
至於雖然兩個映像檔都是111MB，但是實際容量不是222，而是111，因為這兩個標籤都指向同一個映像檔，所以不會額外佔據本機空間。  


標籤最大的作用是 `穩定版本`，如果今天只有 `latest` 時，對於正式環境很不可靠，永遠都要擔心版本更換的時候，正是版本會不會有問題。



### 怎麼到處都有 alpine

如果是剛學習docker的新手，會常常聽到「alpine」這個名詞，這其實就是Linux的一個分支，和Ubuntu、CentOS是一樣的，那為啥在Docker世界中，alpine的討論度會這麼高呢？    
最大原因在於他非常小！我們來下載redis的alpine看看

```shell
$ docker image pull redis:7-alpine

------
REPOSITORY               TAG       IMAGE ID       CREATED        SIZE
redis                    7.0       84ed4fbf28f6   6 days ago     111MB
redis                    7.0.4     84ed4fbf28f6   7 months ago   111MB
redis                    7-alpine   d196fde608b2   6 days ago     30.4MB
```

列出來之後，會發現alpine的容量超級小，小也有小的犧牲，很多基主的套件在alpine是沒有提供的，如果需要要自己去安裝

### 層層堆疊的映像檔

透過以下指令，我們來查看層層堆疊的映像檔是什麼概念！

```shell
$ docker image history nginx:latest

------
IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
9e7e7b26c784   12 days ago   /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B        
<missing>      12 days ago   /bin/sh -c #(nop)  STOPSIGNAL SIGQUIT           0B        
<missing>      12 days ago   /bin/sh -c #(nop)  EXPOSE 80                    0B        
<missing>      12 days ago   /bin/sh -c #(nop)  ENTRYPOINT ["/docker-entr…   0B        
<missing>      12 days ago   /bin/sh -c #(nop) COPY file:e57eef017a414ca7…   4.62kB    
<missing>      12 days ago   /bin/sh -c #(nop) COPY file:abbcbf84dc17ee44…   1.27kB    
<missing>      12 days ago   /bin/sh -c #(nop) COPY file:5c18272734349488…   2.12kB    
<missing>      12 days ago   /bin/sh -c #(nop) COPY file:7b307b62e82255f0…   1.62kB    
<missing>      12 days ago   /bin/sh -c set -x     && addgroup --system -…   60.6MB    
<missing>      12 days ago   /bin/sh -c #(nop)  ENV PKG_RELEASE=1~bullseye   0B        
<missing>      12 days ago   /bin/sh -c #(nop)  ENV NJS_VERSION=0.7.11       0B        
<missing>      12 days ago   /bin/sh -c #(nop)  ENV NGINX_VERSION=1.23.4     0B        
<missing>      12 days ago   /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B        
<missing>      12 days ago   /bin/sh -c #(nop)  CMD ["bash"]                 0B        
<missing>      12 days ago   /bin/sh -c #(nop) ADD file:7b3c55926db26568f…   74.3MB  
```


可以發現每一個映像層，由下而上堆積起來，到最後形成一個映像檔，並且賦予ID - 9e7e7b26c784，這個歷史紀錄不是來自容器的歷史紀錄，而是這個映像檔案在建置時候的歷史紀錄，
在每一層都帶有不同的指令，有些是執行指令，有些是加入檔案，所以才會有0B的層級，也有74.3MB的層級。



### 映像檔快取的秘密

在映像檔中都有獨一無二的SHA所計算出來的ID，目的是幫助Docker去辨認是否已經有一樣的映像層。



### 自製一個帶有PHP程式語言環境的映像檔

Step1: 我們使用PHP的映像檔當做基底

```md
> 自製映像檔第一層

> |---------------------------------------------------------------
> |                                                              |
> |                           PHP                                |
> |                                                              |
> |--------------------------------------------------------------- 
```

Step2:撰寫Dockerfile。在這個第一層的PHP之上增加新的映像層，像是加入環境變數，或是COPY本機的資料到映像檔內等。

```md
> 堆疊而成的映像檔

> |---------------------------------------------------------------         
> |                                                              |  ------ |
> |                           COPY                               |         | 
> |                                                              |         |
> |---------------------------------------------------------------         |
> |---------------------------------------------------------------         |
> |                                                              |         | 
> |                           ENV                                |         |==> 自製映像檔
> |                                                              |         |
> |---------------------------------------------------------------         |
> |---------------------------------------------------------------         |
> |                                                              |         |
> |                           PHP                                |         |
> |                                                              |         |
> |---------------------------------------------------------------  ------ |
```

Step3: 我們想要做另外一個以PHP為基礎的映像檔，而Docker會用到前面提過的SHA創造的獨特ID，辨識出已經有相同的PHP映像檔在本機之中，並利用其已經存在的特性加快映像檔的建置，也一併減少整體電腦耗費的硬體容量，這是Docker最基本的快取機制。


```md
> 共用相同映像檔的兩個映像檔

> |--------------------------| |---------------------------------|
> |                          | |                                 | 
> |     COPY Staging         | |   COPY PRODUCTION               |  
> |                          | |                                 | 
> |--------------------------| |---------------------------------|
> |---------------------------------------------------------------
> |                                                              |
> |                           Port80                             | 
> |                                                              |
> |---------------------------------------------------------------
> |---------------------------------------------------------------
> |                                                              |
> |                           Nginx                              |
> |                                                              |
> |---------------------------------------------------------------
> |---------------------------------------------------------------
> |                                                              |
> |                           自製映像檔                           |
> |                                                              |
> |---------------------------------------------------------------
```


對上面的ENV、COPY、RUN指令不熟悉沒關係，先了解到映像檔是透過一層一層的映像層堆疊而成，而每個指令都會形成一個映像層，這是很重要的概念，之後會反覆利用這個概念來加快建置的速度和大小。



### 對相同應用程式建置兩個不同的映像檔

再舉一個例子，相同的一個應用程式可能會因為部署的環境不同，而分成Staging(接近正式)、Production(正式)兩種版本，並使用COPY指令來複製不同的設定檔案，進而建置兩個不同的映像檔。

```md
> Docker的快取特性

>                   |--------------------------| |---------------------------------|        
>                   |                          | |                                 |        
>            | ---  |          COPY            | |             EXPOSE              |  ---   | 
>            |      |                          | |                                 |        |
>            |      |--------------------------| |---------------------------------|        |
>            |      |--------------------------| |---------------------------------|        |
>            |      |                          | |                                 |        |
>  image1 == |      |           ENV            | |              RUN                |        |  == image2
>            |      |                          | |                                 |        |
>            |      |--------------------------| |---------------------------------|        |
>            |      |---------------------------------------------------------------        |
>            |      |                                                              |        | 
>            | ---  |                           PHP                                |  ---   | 
>                   |                                           基礎映像檔共用       |        
>                   |---------------------------------------------------------------
```


### 映像檔歷史紀錄中的missing

透過快取的機制更了解映像檔後，根據下圖由下往上看，就像是 `docker image history` 一樣，每一個歷史紀錄都代表了一個映像層，我們也可以透過歷史紀錄看到每一層最後一次更動的時間是啥時。   
而最前面的missing其實不是錯誤訊息，也不是有什麼檔案缺失，只是這些都同屬於一個映像檔的一部分，但又礙於ID並不是每一個映像層都需要，才用這種方式顯示。


```md
> 歷史紀錄示意圖

>            |---------------------------------------------------------------    
>            |                                                              |   
>  b459081   |                           Port80                             | -- |
>            |                                                              |    |
>            |---------------------------------------------------------------    |
>            |---------------------------------------------------------------    |
>            |                                                              |    |
>  missing   |                           Nginx                              |    | ====== 映像檔
>            |                                                              |    |
>            |---------------------------------------------------------------    |
>            |---------------------------------------------------------------    |
>            |                                                              |    |
>  missing   |                           自製映像檔                           |    |
>            |                                                              | -- |
>            |---------------------------------------------------------------

```



### 映像檔的唯讀性 

映像檔本身是唯讀的，意思是我們沒辦法更動映像檔內部的檔案系統，不過還記得前面我們曾經在容器裡面安裝 `Curl` 這個套件嗎？為啥會說不能更動呢？這邊我就要提到 `可寫層` 了！


### 可寫層

Docker 在容器啟動的時候，多加了一層可寫層在映像檔上方，所有對於檔案系統的更動都會記錄在可寫層上，且會隨著容器的刪除一起消失，若是更改的內容涉及映像檔原先擁有的檔案，Docker則會在可寫層上採用COPY、Write的方式，使得映像檔最初的檔案系統依舊保持一致，這麼做的理由很簡單，若是每次使用映像檔都會使其檔案系統受到污染，那就會大幅降低共同使用的方便性，啟動速度也會大幅降低


```md
> 利用可寫層來保持唯讀性

> |---------------------------------------------------------------     
> |                                                              |     
> |                           可寫層                              | <---｜
> |                                                              |     ｜
> |---------------------------------------------------------------     ｜  啟動容器時會加入一層可寫層
> |---------------------------------------------------------------     ｜
> |                                                              |     ｜
> |                           映像檔                              | ----｜  
> |                                                              |  
> |---------------------------------------------------------------  
```


### 檢視image細節

我們用 `inspect` 這個指令，來查看image的細節

```shell
$ docker image inspect nginx

------
    {
        ...省略

        "ExposedPorts": {
            "80/tcp": {}
        },
        
        ...省略

        "Env": [
            "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
            "NGINX_VERSION=1.23.4",
            "NJS_VERSION=0.7.11",
            "PKG_RELEASE=1~bullseye"
        ],
        "Cmd": [
            "/bin/sh",
            "-c",
            "#(nop) ",
            "CMD [\"nginx\" \"-g\" \"daemon off;\"]"
        ],
        "Image": "sha256:563125e602b67b1bbea9ae27e43ad6312d3243ff15aa7f498d20e5b189f59fa8",
        "Volumes": null,
        "WorkingDir": "",
        "Entrypoint": [
            "/docker-entrypoint.sh"
        ],

        ,,,省略

        "Os": "linux",


```


由於內容太多，我們只提幾個要注意的
#### (1) ExposedPorts

可以得知映像檔在建置的時候，便已經設好PORT了，這也是為什麼前面在操作容器的時候，我們不會動到右邊的PORT，因為如果更動的話，會和映像檔設定的不同，造成服務對不上
#### (2) ENV環境變數
他就是我們在建置映像檔的時候就可以放進去的變數，當我們啟動容器時，內部就會擁有這些環境變數。

我們來做個測試，先進入nginx執行的容器，並呼叫環境變數：

```shell
$ docker container run --interactive --tty nginx bash
---

root:/# echo $NGINX_VERSION

> 1.23.4
```
  

#### (3) CMD這一段 -> Docker 容器的啟動指令

這裡預設是啟動nginx的服務，回到前面說過的「唯讀性」，我們做的這些改變並不會影響到nginx映像檔本身，即使替換了啟動指令，也都是在可寫層做變化。     
利用 `docker image inspect` 指令查看到的映像檔，本身也沒有任何變化。     


#### (4) OS - Linux
最後是作業系統，可以看到這裡被設計執行在 Linux 上的作業系統，也呼應前面說的，macOS本身也是運行一個迷你的虛擬機還執行Docker，所以這裡看到的是Linux，而不是macOS


### 推送映像檔到 DockerHub

先在腦中的認知，映像檔應該要長這樣，先用nginx來示範

:::tip
nginx:latest #映像檔的名稱：標籤      
:::



接下來要在這個映像檔前面加上儲存庫的名稱，告訴他主要存到哪一個儲存庫之中，我們先打開映像檔列表，就可以看到從左至右分別是儲存酷的「名稱」、「標籤」、「映像檔ID」、「建立時間」、「大小」：
```shell
$ docker image list

------
REPOSITORY               TAG        IMAGE ID       CREATED        SIZE
redis                    7-alpine   d196fde608b2   7 days ago     30.4MB
nginx                    latest     9e7e7b26c784   13 days ago    135MB
redis                    7.0.4      f36d9597bf44   7 months ago   111MB
robeeerto/whoami         latest     3f1e79a452a0   5 months ago   276MB
```


上面「robeeerto/whoami」明顯跟其他人不一樣，，前面有提到說，官方的映像檔事沒有前綴的，而斜線前面的名稱，是dockerhub登入後右上角的名稱。
那要怎麼改變映像檔的名字呢？可以用docker image tag指定，把nginx:latest貼上一個不同的標籤：

```shell
$ docker image tag nginx:latest robeeerto/nginx:latest
```

Ps. 沒有反應是正常的

此時我們再看一次list，會發現貼標籤不會動映像檔做任何改動，可以看到除了名字，其他TAG、ID...等等都是一樣的
```shell
$ docker image list

------
REPOSITORY               TAG        IMAGE ID       CREATED        SIZE
redis                    7-alpine   d196fde608b2   7 days ago     30.4MB
nginx                    latest     9e7e7b26c784   13 days ago    135MB
robeeerto/nginx          latest     9e7e7b26c784   13 days ago    135MB
robeeerto/whoami         latest     3f1e79a452a0   5 months ago   276MB
```

最後我們把剛剛貼上標籤的nginx推上DockerHub

```shell
$ docker image push robeeerto/nginx:latest

------
The push refers to repository [docker.io/robeeerto/nginx]
ce6504827299: Preparing 
cbd644319450: Preparing 
7577f7ad3cd4: Preparing 
4bdc748e7c3d: Preparing 
c607b6f95cf7: Preparing 
0b9f60fbcaf1: Waiting 
denied: requested access to the resource is denied
```


不過會發現，被拒絕了，這是因為權限問題，我們先登入一下

### 用終端機登入DockerHub

```shell
$ docker login

------
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: YeeeChen
Password: 
Error response from daemon: Get "https://registry-1.docker.io/v2/": unauthorized: incorrect username or password
```

但是我們遇到了這個問題，這個原因是因為要申請個人的Token，因此我們現在到官網來申請：

依照此步驟：登入官網後，點擊右上角大頭貼，點擊「Account Setting」-> 點擊左邊「Security」 -> 點擊中間的「New Access Token」-> 填寫Token名稱 -> 點擊「Generate」，最後會跳出一個畫面，只要依照上面步驟在終端機輸入，就可以成功用終端機登入DockerHub了！

(1) Run `docker login -u yeeechen`
(2) At the password prompt, enter the personal access token.(貼上剛剛Token產生的密碼)

```shell
$ docker login -u yeeechen
Password: 

------
Password: 
Login Succeeded

Logging in with your password grants your terminal complete access to your account. 
For better security, log in with a limited-privilege personal access token. Learn more at https://docs.docker.com/go/access-tokens/
```


這樣我們就成功用終端機登入了，讓我們繼續來推上映像檔上去

### 推送映像檔

```shell
$ docker image push robeeerto/nginx:latest
```
這樣推送會錯誤喔！還記得前面輸的，/前面代表的是自己帳戶的名稱，所以這個標籤要改成自己DockerHub的名稱，因此我現在來新增一個我自己的標籤
```shell
$ docker image tag nginx:latest yeeechen/nginx:latest

------
The push refers to repository [docker.io/yeeechen/nginx]
ce6504827299: Mounted from library/nginx
cbd644319450: Mounted from library/nginx
7577f7ad3cd4: Mounted from library/nginx
4bdc748e7c3d: Mounted from library/nginx
c607b6f95cf7: Mounted from library/nginx
0b9f60fbcaf1: Mounted from library/postgres
latest: digest: sha256:2d7084857d5435dbb3468426444a790a256409885ab17c0d3272e8460e909d3c size: 1570
```

Ps. 雖然我的官網名稱是YeeeChen，但是因為指令標籤不能有大寫，因此要都改成小寫喔



DockerHub從另一個儲存庫 `library/nginx` 分享了映像層給我們，這樣做可以大幅減少 DockerHub 在儲存映像檔的容量問題，同時也能加快推送的速度。    
另外，上方的輸出還透露了另外一個訊息就是「官方映像檔並非真的沒有前綴，而是不顯示而已」，這裡確定有看到前綴是library了，代表他們屬於library這個儲存庫     




### 映像檔完全名稱

我們在推送映像檔的時候，會注意到有一行寫著：
```md
> The push refers to repository [docker.io/yeeechen/nginx]
```

後面的yeeechen/nginx說過了，那docker.io呢？這其實是DockerHub儲存庫的網域名稱，所以下方這段才是映像檔的全名：
```md
> docker.io/yeeechen/nginx
```



### 本地建立映像檔儲存庫

先來執行下面這一段：
```shell
$ docker run --detach --publish 5000:5000 --name registry --env REGISTRY_STORAGE_DELETE_ENABLED=true registry:2

------
Unable to find image 'registry:2' locally
2: Pulling from library/registry
547446be3368: Pull complete 
5145a9c19e2a: Pull complete 
9b36e14ae70d: Pull complete 
5fa9326efe1e: Pull complete 
544654b4f256: Pull complete 
Digest: sha256:8c51be2f669c82da8015017ff1eae5e5155fcf707ba914c5c7b798fbeb03b50c
Status: Downloaded newer image for registry:2
dbbe638afbcd8e3d7ade936b3abba50ff320fae913275afa5caa0f28165db63b
docker: Error response from daemon: Ports are not available: exposing port TCP 0.0.0.0:5000 -> 0.0.0.0:0: listen tcp 0.0.0.0:5000: bind: address already in use.
```

這樣我們就在本地端執行了一個映像檔儲存庫，讓我們試著把剛才重新貼標籤的映像檔再換一個新標籤
```shell
$ docker image tag yeeechen/nginx:latest localhost:5000/yeeechen/nginx:latest
```

接著把他推到本地執行起來的映像檔儲存庫內：
```shell
$ docker image push localhost:5000/yeeechen/nginx:latest 
```









本地建立映像檔儲存庫怪怪的


Dockerfile 內容解析
------


```Dockerfile
FROM ruby:3.1.2-alpine
ENV AUTHOR=robertchang

RUN apk add --update --no-cache \
    build-base \
    curl

WORKDIR /app

COPY . .

RUN gem install bundler:2.3.19 && \
    bundle install -j4 --retry 3 && \
    bundle clean --force && \
    find /usr/local/bundle -type f -name '*.c' -delete && \
    find /usr/local/bundle -type f -name '*.o' -delete && \
    rm -rf /usr/local/bundle/cache/*.gem

EXPOSE 3000

CMD ["bundle", "exec", "ruby", "whoami.rb", "-p", "3000", "-o", "0.0.0.0"]
```



### From指令

每一個映像檔，都必須以其他的映像檔為基底，這裡因爲執行的是RUBY所撰寫的應用程式，故選用「ruby:3.1.2-alpine」這個映像檔為基底，反之，如果你使用PHP撰寫，則會選用「PHP:zts-alpine」。
至於標籤的選擇，則取決你在開發這個應用程式時的版本限制，這就是自己在撰寫時所需衡量的部分，並沒有一定的公式可以套用，但是使用FROM作為Dockerfile的起手式，是一定會做的一件事。



### ENV指令

ENV指令適用來設定執行容器後的環境變數，以key=value的方式設定，以前面的例子來說，執行容器後，作業系統終究會存在一個 `$AUTHOR` 的環境變數，且值為 `robertchang`。

我們這邊來做的練習，因為nginx官方的Dockerfile中有一個環境變數，是 `ENV NGINX_VERSION`，我們來實際印出來看看
```shell
# 先進到容器中
$ docker container run --interactive --tty nginx bash  
root@2ed1759177bc:/# echo $NGINX_VERSION

------
1.23.4
```

可以看到印出 `1.23.4` 這一串數字，證明nginx中有這個環境變數可以用

### RUN指令

RUN指令是終端機執行的指令，以上面範例來説 ， `apk add --update --no-cache build-base curl` 為例，就是希望在接下來的容器環境中安裝 `build-base` 和 `curl` 這兩個工具。
指令不侷限在安裝工具，也可以是Linux系統常見的改變權限 `chown` 或是 `add group` 等。

只要是能夠被作業系統所接受的指令，都可以被寫在run裡面，能夠被作業系統所接受是指使用 `alpine` 作業系統時，就要使用apk套件管理工具，若是使用Ubuntu時，則會改用apt套件工具，如果適用macOS，則適用brew這個工具。


#### 為什麼run指令後面有 \ (反斜線符號)

如果今天不用反斜線符號，而是這樣分成兩行寫

```dockerfile
RUN apk add --update --no-cache build-base
RUN apk add --update --no-cache curl
```

這會導致映像檔的映像層變多，畢竟從一個映像層轉為兩個映像層，而在軟體開發中，要的就是又小又快，這樣不必要的浪費是不允許的。

#### 為什麼run指令中間有 && 符號

`&&` 符號在程式語言的世界中很常見，代表的是前面的指令執行結果沒有出錯，則接著執行後方的指令，而為什麼要用 `&&` 來串接指令呢？其實就和反斜線符號一樣，希望可以把指令濃縮到一個映像層中。

### WORKDIR指令

WORKDIR 指令式建立一個資料夾，並且以這個資料夾作為預設的工作目錄，有人會問，這個和我直接執行 `RUN mkdir app` 有什麼不同？

區別在於 RUN mkdir app 確實會建立一個 app 的資料夾，但預設的做目錄還是在根目錄，但以 WORKDIR/app 來說，做的是兩件事，第一件事就是 `mkdir app` 建立app資料夾，並且 cd app 進入這個資料夾內，以app資料夾作為工作目錄，緊接著 Dockerfile 內排在 WORKDIR 後面的指令，都是在 /app 這個資料夾內執行。



### COPY指令

COPY指令是從本機的檔案系統中複製資料到容器內的檔案系統，這裡的例子是 `COPY..` ，點符號代表的意思是「此處」的意思，所以這整個指令翻成白話文，就是從DockerFile身處的資料夾，複製所有的檔案到容器內部檔案系統的當前工作目錄。   

利用 `..` 的方式，一開始很容易搞混，這裡舉一個簡單的例子，下方是假設的檔案目錄，只有一個Dockerfile和txt檔：
```md
> .
> |------ Dockerfile
> |------ example.txt
```

我們可以這樣寫：
```DOCKERFILE
COPY example.txt example.txt

# 左邊是本機的檔案名稱，右邊則爲執行成容器後的檔案名稱
```

若是不希望他在容器內叫做「example.txt」，而是叫做「happy.txt」，我們也可以這樣寫：
```DOCKERFILE
COPY example.txt happy.txt
```

Ps. 重要的是檔案的內容，而不是檔案的名稱


### EXPOSE指令

EXPOSE 指令就是執行容器後預設打開的PORT，也是為什麼我們在剛開始學習使用容器時，都不會去更動右邊的PORT的原因，這個數值已經在撰寫映像檔的階段，就做好了設定。

上網查看nginx的DOCKERFILE中，就有寫到「EXPOSE 80」，意味著這個映像檔執行成容器後，預設打開 prot 80，其他的都是關閉的狀態，所以就算想要強制對應到容器的PORT，也是沒辦法的事。


### CMD指令

CMD指令是在映像檔執行成為容器時，所執行的第一個指令，這是關係到「容器是否進入退出狀態」的指令。


### 建置映像檔


我們用書本提供的檔案，來建置第一個映像檔，進入 `whoami.rb` 檔案後，直接輸入下面的指令：
```shell
$ docker image build --tag whoami .                                                                

------
[+] Building 32.1s (11/11) FINISHED                                                                                                            
 => [internal] load build definition from Dockerfile                                                                                      0.0s
 => => transferring dockerfile: 529B                                                                                                      0.0s
 => [internal] load .dockerignore                                                                                                         0.0s
 => => transferring context: 2B                                                                                                           0.0s
 => [internal] load metadata for docker.io/library/ruby:3.1.2-alpine                                                                      3.6s
 => [auth] library/ruby:pull token for registry-1.docker.io                                                                               0.0s
 => [1/5] FROM docker.io/library/ruby:3.1.2-alpine@sha256:05b990dbaa3a118f96e9ddbf046f388b3c4953d5ef3d18908af96f42c0e138d9                4.3s
 => => resolve docker.io/library/ruby:3.1.2-alpine@sha256:05b990dbaa3a118f96e9ddbf046f388b3c4953d5ef3d18908af96f42c0e138d9                0.0s
 => => sha256:fd602cf58768c68ce194e9b618f6a22fec351c543ab89c110be7b0373df82f44 224B / 224B                                                1.0s
 => => sha256:153f2f432a9b49d34b175ea8a718dbf634a73e1e60f560e6e68b520613891016 28.54MB / 28.54MB                                          3.6s
 => => sha256:05b990dbaa3a118f96e9ddbf046f388b3c4953d5ef3d18908af96f42c0e138d9 1.65kB / 1.65kB                                            0.0s
 => => sha256:2b67a90c792985cb5edb89dd50651d0ddf2ef942a25871b6031f5573c41d7fbc 1.36kB / 1.36kB                                            0.0s
 => => sha256:9ddeeebf05be21bc57aef47bccc1bc49b02879c8650df4a7a797b88562990dce 6.12kB / 6.12kB                                            0.0s
 => => sha256:e43d5165cf5412b14fdf30765f0d1823cd0d27446e34154b51275785a4e57fb6 3.82MB / 3.82MB                                            2.3s
 => => sha256:83cd339cf8137c30ead5359c8c7a280d2d901391514b71f7e457c5262d782b53 172B / 172B                                                1.9s
 => => extracting sha256:e43d5165cf5412b14fdf30765f0d1823cd0d27446e34154b51275785a4e57fb6                                                 0.3s
 => => extracting sha256:fd602cf58768c68ce194e9b618f6a22fec351c543ab89c110be7b0373df82f44                                                 0.0s
 => => extracting sha256:153f2f432a9b49d34b175ea8a718dbf634a73e1e60f560e6e68b520613891016                                                 0.5s
 => => extracting sha256:83cd339cf8137c30ead5359c8c7a280d2d901391514b71f7e457c5262d782b53                                                 0.0s
 => [internal] load build context                                                                                                         0.0s
 => => transferring context: 2.49kB                                                                                                       0.0s
 => [2/5] RUN apk add --update --no-cache     build-base     curl                                                                        10.5s
 => [3/5] WORKDIR /app                                                                                                                    0.1s
 => [4/5] COPY . .                                                                                                                        0.0s
 => [5/5] RUN gem install bundler:2.3.19 &&     bundle install -j4 --retry 3 &&     bundle clean --force &&     find /usr/local/bundle   12.9s
 => exporting to image                                                                                                                    0.7s
 => => exporting layers                                                                                                                   0.7s
 => => writing image sha256:9fa38c019fbc9a50aae4cc6cfb41737ae1388644ca6873ea0cc0b8736b1936db                                              0.0s
 => => naming to docker.io/library/whoami                                                                                                 0.0s
```

這樣我們就成功建置人生中的第一個映像檔了！！


### 建置映像檔的每個階段

#### 建置指令
我們現在來看一下，剛剛寫的指令和實際跑的東西在幹嘛

```shell
$ docker image build --tag whoami .   
```
--tag 的作用就是給予要建置的映像檔標籤名稱，而最後面有一個非常重要的「.」(半型句號)，是常常有人忽略而導致沒辦法建置映像檔的關鍵，這個「.」代表的是「這裡」的意思，也就是Docker會預設在現在的目錄中Dockerfile，並以其為主建置映像檔。


#### --file指令
如過今天要根據 `staging`、`production` 分別有好幾種不同的Dockerfile，該怎麼做呢？Docker提供了 `--file` 指令給你使用，例如：想要建置的是 `production` 的映像檔，則可以輸入如下指令：
```shell
$ docker image build --tag whoami:production --file Dockerfile.production .
```

要記得最後那個「.」半型句號還是要加上去，要讓Docker知道健在自己處於哪個路徑，例如：「./docker/」的相對路徑，總之不要讓Docker迷路，他會不知道自己在哪。



#### 第一階段 

根據整個映像檔的紀錄來看，第一個階段是從FROM開始，因為本地端沒有「ruby:3.1.2-alpine」這個映像檔，就從DockerHub上面抓取了官方的映像檔製作為基底。


至於Docker第二個動作，應該是ENV為首的指令，但為什麼沒有出現在建置過程中的紀錄中呢？這是因為ENV為首的指令雖然製作出一層映像層(空的 0kb)，但由於其本身的指令並不會對檔案系統有更動，故沒有出現在建置過程的記錄上。


:::tip
Docker中每一次指令都會製作一個新的映像檔，至於會不會出現在建置的紀錄中，端看這個指令對於檔案系統是否有更動     
例如：安裝套件、新增資料夾等動作，都會出現在建置的紀錄中，反之，如expose、ENV指令，就沒有出現在建置紀錄中，因為其並沒有對檔案系統有任何的更動  
:::


可以利用 `docker image history` 指令，來查看映像檔所有的映像層，透過 `docker image inspect` 指令，則可以看到映像檔的檔案系統層。



#### 第二~五階段

後面的階段，都是按照Dockerfile的說明執行動作，所以我才會說 `映像檔就是執行容器的說明書` 只要寫好步驟，就能夠保證應用程式順利的執行。


### 建置時的快取機制

我們在建置一個一模一樣的映像檔
```shell
$ docker image build --tag whoami .

------
[internal] load build definition from Dockerfile                                                                           0.0s
=> transferring dockerfile: 529B                                                                                           0.0s
[internal] load .dockerignore                                                                                              0.0s
...省略
> naming to docker.io/library/whoami                                                                                       0.0s
```


可以看到藉由docker優異的快取機制，現在整個建置過程超級快，整個建置過程大約節省了50倍的時間。

再來我打開剛剛的Dockerfile檔案，我們來把ENV中的AUTHOR變數，改成自己的英文名字，並且在重新建置一次映像檔

```Dockerfile
FROM ruby:3.1.2-alpine
ENV AUTHOR=yeeechen

RUN apk add --update --no-cache \
    build-base \
    curl

...略    
```

再來重新建置一次image檔

```shell
$ docker image build --tag whoami .

------

...省略
 => exporting to image                                                                               0.7s 
 => => exporting layers                                                                              0.7s 
 => => writing image sha256:8fdc17e6b7f03b899f371c6c4dff5bcf99ac5bc719bb439aa686f47003d40d3a         0.0s 
 => => naming to docker.io/library/whoami    
```

會發現快取機制好像不見了？原本的0.6s又變成了40多秒，這是因為改動映像層的關係(我們更動的ENV)，進而造成sha算出來的ID有異，使得映像檔找不到匹配的映像層，而是重新建置。

不過你可能會有個疑問，就是重新建置ENV那層映像層不就好了，怎麼還會跑這麼久？其實，這是Docker一個有趣的機制，若是上層的映像層重新建置，則以下的映像層會全部重新建置，在這個案例中，我們更動了ENV，因此下面的RUN、WORKDIR、EXPOSE...等指令，都重新建置了，所以才會這麼久。


### 重新整理Dockerfile的執行順序


為了把重新建置的副作用降到最低，我們需要調整Dockerfile的指令順序，`指令的順序並不會影響到容器的啟動`，所以不用擔心。


唯一不會動的就是FROM這個紙令，因為我們需要透過另外一個映像檔當做基底，所以FROM一定是最上面的，而當我們在思考如何調整指令順序時，要想一下哪些指令的變動頻率比較低，越低的要放在越上面，才可以讓重新建置的副作用降到最低。

#### CMD、EXPOSE

首先，變動頻率最低的一定是 `CMD` 和 `EXPOSE`，啟動應用程式的方式都還是大同小異，而 `EXPOSE` 則是在設定好後，就很少進行變動了，舉例來說，nginx也不會突然變成開 `678port`。

現在Dockerfile變成這樣：

```dockerfile
FROM ruby:3.1.2-alpine
ENV AUTHOR=yeeechen

EXPOSE 3000
CMD ["bundle", "exec", "ruby", "whoami.rb", "-p", "3000", "-o", "0.0.0.0"]

RUN apk add --update --no-cache \
    build-base \
    curl

WORKDIR /app

COPY . .

RUN gem install bundler:2.3.19 && \
    bundle install -j4 --retry 3 && \
    bundle clean --force && \
    find /usr/local/bundle -type f -name '*.c' -delete && \
    find /usr/local/bundle -type f -name '*.o' -delete && \
    rm -rf /usr/local/bundle/cache/*.gem
```

而ENV的變動頻率，要看專案而定，也可以透過在啟動容器的時候傳入(在一開始操作容器的時候有教學指令)


#### RUN apk、WORKDIR、COPY、RUN gem install

至於「RUN APK」和「WORKDIR」的取捨，肯定是「WORKDIR」放比較上面，因為我們可能會需要新的套件，所以「RUN apk」的異動頻率會蠻高的。(所以我們把WORKDIR往上移)

再來是「RUN APK」和「COPY」的取捨，常理來說COPY的異動頻率會比安裝套件還要來得高，因為開發初期，檔案會一直變動，導致雖然指令本身都是「COPY..」，但編輯過的檔案會導致算出來的SHA不同，進而觸發重新建置的副作用。

最後是「RUN gem install...」，他也是一個安裝套件的指令，但他比較特別，如果今天我們把它往上移動，像這樣移動到 `COPY..` 上面：

```dockerfile
FROM ruby:3.1.2-alpine
ENV AUTHOR=yeeechen

EXPOSE 3000
CMD ["bundle", "exec", "ruby", "whoami.rb", "-p", "3000", "-o", "0.0.0.0"]

WORKDIR /app

RUN apk add --update --no-cache \
    build-base \
    curl


RUN gem install bundler:2.3.19 && \
    bundle install -j4 --retry 3 && \
    bundle clean --force && \
    find /usr/local/bundle -type f -name '*.c' -delete && \
    find /usr/local/bundle -type f -name '*.o' -delete && \
    rm -rf /usr/local/bundle/cache/*.gem

COPY . .
```


接著我們執行看看，會發現安裝套件的時候出錯了！，錯誤訊息是 `Could not locate Gemfile`，也就是找不到可以去參照的檔案來安裝套件。
```shell
$ docker image build --tag whoami .  

------
#7 1.787 Successfully installed bundler-2.3.19       
#7 1.787 1 gem installed                                                        
#7 2.021 Could not locate Gemfile
---
executor failed running [/bin/sh -c gem install bundler:2.3.19 &&     bundle install -j4 --retry 3 &&     bundle clean --force &&     find /usr/local/bundle -type f -name '*.c' -delete &&     find /usr/local/bundle -type f -name '*.o' -delete &&     rm -rf /usr/local/bundle/cache/*.gem]: exit code: 10
```


而會發生這個錯誤的原因很簡單，就是因爲之前提過映像檔是一層一層疊起來的，下層會具備上層所擁有的檔案系統及安裝過的套件，而在「RUN gem install...」的當下，還沒有把本機的檔案COPY到建置的過程中，進而導致執行「RUN gem install」的當下，根本找不到參照的(Gemfile)。

#### 最終修訂

最終，Dockerfile修訂到最小的影響版本是這樣

```dockerfile
FROM ruby:3.1.2-alpine
ENV AUTHOR=yeeechen

EXPOSE 3000
CMD ["bundle", "exec", "ruby", "whoami.rb", "-p", "3000", "-o", "0.0.0.0"]

WORKDIR /app

RUN apk add --update --no-cache \
    build-base \
    curl

COPY . .

RUN gem install bundler:2.3.19 && \
    bundle install -j4 --retry 3 && \
    bundle clean --force && \
    find /usr/local/bundle -type f -name '*.c' -delete && \
    find /usr/local/bundle -type f -name '*.o' -delete && \
    rm -rf /usr/local/bundle/cache/*.gem
```

`FROM -> ENV -> EXPOSE -> CMD -> WORKDIR -> RUN -> COPY -> RUN`  

在這個情形下，有機會被更動到的只有COPY檔案，才會觸發重新建置的副作用，已經算是把副作用影響減到最低！





多階段建置映像檔
------

在Docker世界裡面，1G的映像檔可以說是非常的大，就連執行postgreSQL的映像檔都只有376MB，因此我們剛剛練習的作業 -> 做一個next.js的映像檔有1.08G可以說是不可思議的大(這檔案只有一行字)。

有一種方式可以幫映像檔瘦身，就是使用「多階段建置」的方式，整個多階段建置的精髓都是在「COPY --form」這段指令中，之前我們用COPY都是從本機複製檔案到映像檔的檔案系統中，而COPY --from則可以讓我們從另外一個映像檔複製檔案到映像檔的檔案系統中，而COPY --from則可以讓我們從另一個映像檔複製檔案到現階段的映像檔。

我們直接來看dockerfile範例：

```dockerfile
FROM alpine:3.16.2 AS builder # 建置階段
RUN echo 'Builder' > /example.txt # 建置階段

FROM alpine:3.16.2 AS tester # 測試階段
COPY --from=builder /example.txt # 測試階段
RUN echo 'Tester' >> /example.txt # 測試階段

FROM alpine:3.16.2 # 最終階段
COPY --from=tester /example.txt /example.txt # 最終階段CMD ["cat", "/example.txt"] # 最終階段
```

這邊將docker分成三階段，這裡有一個簡單的概念，只要是用到FROM作為開頭，就可以說是一個新的階段，而在第一個FROM到第二個FROM的之間的指令結果，都會停留在第一階段中。

### 建置階段

首先利用了「alpine:3.16.2」映像檔作為基礎，並且簡單執行了一個RUN指令，作用是把Builder這段文字寫入「example.txt」檔案，就結束任務了。

### 測試階段

這裡Dockerfile讀到了第二個FROM，所以就當作一個新的開始，而我們一樣使用「alpine:3.16.2」映像檔當作基礎，但不同的是，我們使用了「COPY --from=builder/example.txt /example.txt」這段指令。

對於Docker來說，要從builder階段複製一份「example.txt」到現在這個階段內，並命名為「example.txt」，此時docker會去找builder階段，但其實我們已經把第一階段命名好了，可以看到FROM的後面，我們用了AS這個語法，並將第一個階段命名為「builder」。接著我們把Tester這段文字也寫入example.txt檔案中，再來就遇到第三個FROM，並結束第二階段。

### 最終階段

來到最後階段，我們使用了「COPY --from=tester /example.txt /example.txt」，把tester這個階段的example.txt複製過來最終階段，並命名為「example.txt」，做得跟第二階段一樣，只是最後用CMD，並且去讀取example.txt這個檔案的內容。


最後，我們來建置這個映像檔：

```shell
$ docker image build --tag example .
```

接著利用我們學習到的Docker知識，猜猜把映像檔執行成容器會發生啥事？沒錯，就像這樣

```shell
$ docker container run example

------
Builder
Tester
```


印出來的內容，藉由複製前兩個階段的檔案一直傳遞到最後一個階段，並且讀取檔案中的內容，確實都還保留著前兩個階段所寫入的文字。

這代表我們能夠在前面的階段，將要安裝的套件以及安裝套件所需的編譯工具準備好，並且安裝完應用程式所需的套件，只把安裝好的套件複製到第二階段，這將會把第一個階段編譯所需要的工具都丟棄，大幅減少了映像檔大小

下面我們將實際看看多階段建置的應用！



### Golang 應用程式的多階段建置


### Express.js 應用程式多階段建置


### .dockerignore

在建置檔案的時候，可以看到一行紀錄：
```md
> => [internal] load .dockerignore
```


每一次建置的過程，其都會自動去讀取「.dockerignore」檔案，但在之前並沒有好好介紹過，其最主要功能在於「提前篩選掉不需要進入建置階段的檔案」，例如：常見的README.md，或是一些開發環境的設定檔，「對於建置階段以及執行容器沒有幫助的檔案」，都應該放進「.dockerignore」內。


### 清理本機容量

建置這麼多映像檔和容器後，電腦裡現在有一大堆的不必要的黨案，我們現在下指令來看一下有哪些：
```shell
$ docker system df

------
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          20        1         5.497GB   4.434GB (80%)
Containers      1         1         43.2MB    0B (0%)
Local Volumes   7         0         558.1MB   558.1MB (100%)
Build Cache     62        0         109MB     109MB
```


### 清理不必要的容器

有幾個方式可以清理容器，第一個是我們前面有使用過的：


```shell
$ docker container rm --force $(docker container list --all --quiet)
```

第二個是刪除停止執行的容器：
```shell
$ docker container prune

------
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Total reclaimed space: 0B
```

### 清理不要的映像檔

清理映像檔的指令和清理容器的指令很像：
```shell
$ docker image prune
```

#### 刪掉沒標籤的映像檔
Docker 有提示只會刪除 `dangling` 的映像檔，而 `dangling` 則意味著標籤被奪走的映像檔，以下示範什麼行為會製造出 `dangling` 的映像檔，這裡隨手建立一個 Dockerfile：

```dockerfile
FROM alpine:latest
CMD [ "echo", "Hi" ]
```

接著建置映像檔：
```shell
$ docker image build --tag dangling .
```

這時候列出所有映像檔，就可以看到他：
```shell
$ docker image list 

------
REPOSITORY            TAG        IMAGE ID       CREATED        SIZE
dangling              latest     3d2591fe726f   19 hours ago   13.7MB
```

這時候我們來修改一下dockerfile：

```dockerfile
FROM alpine:latest
CMD [ "echo", "Hi, my name is yeeechen" ]
```

再來使用相同標籤來製作映像檔：
```shell
$ docker image build --tag dangling .
```

接著把所有映像檔列出來，就會看到有一個映像檔，以 `<none>` 的方式顯示：
```shell
$ docker image list 

------
REPOSITORY          TAG        IMAGE ID       CREATED        SIZE
dangling            latest     3d2591fe726f   19 hours ago   13.7MB
<none>              <none>     d9ea7fb57d04   19 hours ago   13.7MB
```


這裡的 `none` 就是第一份建置的映像檔，因為標籤相同的關係，被第二次建置的映像檔搶走的標籤，轉變成 `dangling` 映像檔，而在使用 docker image prune 的指令時，就可以刪掉這些沒有標籤的映像檔


#### 映像檔全刪掉

有另外一個暴力的指令，就是把全部的映像檔，除了正在執行以外的映像檔都刪掉：

```shell
$ docker image prune --all

------
WARNING! This will remove all images without at least one container associated to them.
Are you sure you want to continue? [y/N] y
Deleted Images:
untagged: redis:7.0
...省略

Total reclaimed space: 1.948GB
```

最後可以看到我們清理了快2G的空間

再來看一下系統的狀況
```shell
$ docker system df

------
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          1         1         1.07GB    0B (0%)
Containers      1         1         43.2MB    0B (0%)
Local Volumes   7         0         558.1MB   558.1MB (100%)
Build Cache     62        0         1.485GB   1.485GB
```
可以看到image除了正在執行的外，都刪光光了

### 清理系統

接下的的指令，可以一次清除停止的容器、無名的映像檔、無名的快取、沒有用到的虛擬網路也會一起刪掉：
```shell
$ docker system prune

------
Are you sure you want to continue? [y/N] y
Deleted Networks:
external-service-network
newNetwork
...省略

Total reclaimed space: 1.485GB
```
這邊我們清理了快1.5G的空間  


再來看一下系統的狀況

```shell
$ docker system df

------
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          1         1         1.07GB    0B (0%)
Containers      1         1         43.2MB    0B (0%)
Local Volumes   7         0         558.1MB   558.1MB (100%)
Build Cache     7         0         0B        0B
```

最後看到除了正在執行中的，還有下面要講的「Volumes」，其他都刪乾淨了！



Docker Volume
------

本章的核心圍繞在「使用容器時如何保存資料」以及「保存資料為什麼會是一個問題」。   
要知道使用容器時通常有兩個核心，分別是「immutable」(不可變動的)、「ephemeral」(短暫的)，雖然有點難懂，但其實就是可以蕤譯刪除容器，並利用同一個映像檔再啟動一個相同的容器，也不會對整體的應用程式造成任何問題。


### 有/無 狀態的應用程式

#### immutable 的概念

目的是讓我們可以不斷地根據相同的映像檔啟動容器，且每次都是相同的，也可以說，容器本身是無狀態的運作環境。簡單來說，「無狀態」就是每一次執行都是獨立的，不會根據上一次的運作而有所改變。

#### ephemeral 的概念

其實「immutable」和「ephemeral」是密不可分的，因為容器本身是無狀態的，導致其生命週期非常短暫，而生命週期短暫的好處是「重新啟動的速度」以及「沒有任何副作用」。


#### 沒有狀態的話，那資料怎辦

既然沒有狀態，那我們的應用程式產生的資料要怎麼辦，不論是資料庫(MySQL、postgreSQL)或是快取(Redis、Memcached等)以及那些和映像檔的檔案系統分割開來的資料，該怎麼處理呢？
不過，當然，這些資料不可能存在映像檔裡面，畢竟放在裡面映像檔會直接被資料塞爆，這樣每次部署都要花很長的時間

而為了解決這件事，Docker提供了「Volume」及「Bind Mount」(掛載)的方式來做到保存狀態這件事情。

就算今天我們重新啟動容器、更新應用程式版本，這些資料都還是會存在，所以在Docker中，容器和Volume本身就是兩個模組，也是兩個不同的物件，這是Docker為了實踐有狀態的應用程式所給出的答案。


Volume是在容器磁碟空間外的一個儲存空間，換言之，可以像使用Docker虛擬網路時一樣，把Volume連接到任何想要連接的容器上，而在容器看起來，他不過就是磁碟空間中的一個路徑或是一個檔案目錄。

`Bind Mount` 則是將本機的檔案或檔案目錄掛載到容器內，對於容器本身來說，他不會知道這個檔案是不是掛載進來的，因為他就只是磁碟空間的一個路徑或是檔案目錄。

### 從DockerHub 看 Volume


我們從DockerHub來查看別人是怎麼使用volume的，首先到DockerHub上，搜尋mysql後，隨便選擇官方的任一版本映像檔，進入後，會看到官方映像檔在GutHub上面Dockerfile的原始碼，拉到最下面，可發現有一行使用 `Volume` 作為指令。

```md
> VOLUME /var/lib/mysql
```

以這個例子來說，MySQL的資料庫預設儲存路徑是放在「VOLUME /var/lib/mysql」位置，要知道資料庫雖然聽起來是一個獨立的存在，但在怎麼樣他還是磁碟空間中的一個檔案罷了，這段指令完整的語意就是告訴Docker，此映像檔執行成容器時建立一個Volume，並且連接到容器內「/var/lib/mysql」這個路徑的檔案，這表示所有放在這個volume中的資料，都是存在容器之外，除非手動刪除volume。


#### 檢查有無volume

拉取mysql映像檔看：
```shell
$ docker image pull mysql
```

用inspect指令看：
```shell
$ docker image inspect mysql

------
...省略
"Volumes": {

    "/var/lib/mysql": {}
```

Ps. 可以使用 `docker volume prune` 清空你的volume



### 執行帶有 volume 指令的映像檔

接著我們把帶有mysql執行起來
```shell
$ docker container run --detach --name mysql --env MYSQL_ROOT_PASSWORD=whatever mysql
```
建立好之後，我們來看一下這個container內部的資訊(用inspect指令)：

```shell
$ docker container inspect mysql   

------
"Mounts": [
    {
        "Type": "volume",
        "Name": "14be5e13a53e377b28b3cb014143c5071782cc192e4a393f81ccef06e0dcb400",
        "Source": "/var/lib/docker/volumes/14be5e13a53e377b28b3cb014143c5071782cc192e4a393f81ccef06e0dcb400/_data",
        "Destination": "/var/lib/mysql",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
],
```

可以看到它掛載了一個volume在裡面，Destination的部分是指容器內部路徑，Source則是指外部的volume，而這個volume路徑，若是使用Linux作業系統的人，可以直接透過CD指令進入到這過資料夾中。


現在我們來把存活的volume列出來：
```shell
$ docker volume list

------
DRIVER    VOLUME NAME
local     14be5e13a53e377b28b3cb014143c5071782cc192e4a393f81ccef06e0dcb400
```

這邊我們發現了，VOLUME NAME和容器的SHA值是一樣的，表示寫在映像檔中的volume指令會在沒有指定volume的情況下，自行建立一個以隨機SHA值命名的volume。

我們一樣可以用inspect指令來查看volume詳細資訊
```shell
$ docker volume inspect 

------
[
    {
        "CreatedAt": "2023-05-02T10:26:02Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/14be5e13a53e377b28b3cb014143c5071782cc192e4a393f81ccef06e0dcb400/_data",
        "Name": "14be5e13a53e377b28b3cb014143c5071782cc192e4a393f81ccef06e0dcb400",
        "Options": null,
        "Scope": "local"
    }
]
```

#### 如何知道哪個volume連到容器

現在突然發現一件事，我們可以透過容器知道現在連接的是哪一個volume，但沒辦法反過來從volume角度去看到現在連結的是哪一個容器。

若是開啟兩個MySQL服務：

```shell
$ docker container run --detach --name mysql2 --env MYSQL_ROOT_PASSWORD=whatever mysql
```

接著列出所有volume：
```shell
$ docker volume list

------
DRIVER    VOLUME NAME
local     14be5e13a53e377b28b3cb014143c5071782cc192e4a393f81ccef06e0dcb400
local     a2606fc0c21015cbf036fa140086897695d48c88631d94ab142ac01122a961de
``


發現怪怪的地方了，對吧！

從列出volume這件事情，我們沒有辦法得到任何有用的資訊，也沒辦法知道這是連接到哪一個容器，若是今天容器被刪除了呢？要知道「刪除容器並再次開始」是很容易的事情。
```shell
$ docker container rm --force mysql mysql2

------
mysql
mysql2
```

確認所有容器都刪乾淨了：
```shell
$ docker container list -all

------
CONTAINER ID   IMAGE          COMMAND                  CREATED      STATUS      PORTS                    NAMES
```

接著在列出volume，可以發現沒有被刪除：
```shell
$ docker volume list

------
DRIVER    VOLUME NAME
local     14be5e13a53e377b28b3cb014143c5071782cc192e4a393f81ccef06e0dcb400
local     a2606fc0c21015cbf036fa140086897695d48c88631d94ab142ac01122a961de
```

這跟我們前面提到的一樣，volume要另外再手動刪除，但是現在完蛋了，哪個volume對應到哪個容器呢？雖然資料都還在，但是不知道他主要是做啥的！


### 爲 volume 命名

因此，我們可以幫 volume 命名，這樣就不用再靠猜測 volume 裡面到底有哪些資料了！


回到一開始的啟動容器的指令，加入--volume指令，來告訴Docker要對應的volume名字是什麼：

```shell
$ docker container run --detach --name mysql --env MYSQL_ROOT_PASSWORD=whatever --volume mysql-data:/var/lib/mysql mysql
```

注意到了嗎？只需要在目的地的前方，加上volume的名稱，並用冒號進行連接，就能夠建立一個有名字的volume，並連接到「/var/lib/mysql」這個路徑。

接著列出所有的volume，看看是不是真的有效：
```shell
$ docker volume list

------
DRIVER    VOLUME NAME
local     14be5e13a53e377b28b3cb014143c5071782cc192e4a393f81ccef06e0dcb400
local     a2606fc0c21015cbf036fa140086897695d48c88631d94ab142ac01122a961de
local     mysql-data
```
可以看到確實有一個我們剛剛建置的volume，接著我們在使用inspect詳細地看volume：


```shell
$ docker volume inspect mysql-data

------
[
    {
        "CreatedAt": "2023-05-02T17:07:52Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/mysql-data/_data",
        "Name": "mysql-data",
        "Options": null,
        "Scope": "local"
    }
]
```

可以發現我們直接用 `inspect + volume名稱` 可以看到volume的詳細資料。

#### 第二種volume取名稱方式
另外，除了在容器啟動時輸入volume的名字，也可以手動提前建立好volume，並連接到容器上：

```shell
$ docker volume create whatever
```



### volume 的共用性

前面有提到可以把volume連接到任何想要連接的容器上面，所以這裡再次啟動一個mysql的容器，並且把剛建的`mysql-data`連接到新容器上。

先刪除現有容器：
```shell
$ docker container rm --force $(docker container list --all --quiet)
```

再次啟動一個mysql容器

```shell
$ docker container run --detach --name mysql --env MYSQL_ROOT_PASSWORD=whatever --volume mysql-data:/var/lib/mysql mysql
```

現在列出現有volume
```shell
$ docker volume list

------
DRIVER    VOLUME NAME
local     14be5e13a53e377b28b3cb014143c5071782cc192e4a393f81ccef06e0dcb400
local     a2606fc0c21015cbf036fa140086897695d48c88631d94ab142ac01122a961de
local     mysql-data
```

這邊可以看到因為我們在建立容器的時候有給volume名稱，Docker會發現已經有一個存在的 `mysql-data volume`，就不會再次用SHA隨機產生一個volume了。

利用 `docker container inspect` 指令，可以確定新的容器連接上的volume確實是 `mysql-data` 沒錯。

```shell
$ docker container inspect mysql

------
"Mounts": [
    {
        "Type": "volume",
        "Name": "mysql-data",
        "Source": "/var/lib/docker/volumes/mysql-data/_data",
        "Destination": "/var/lib/mysql",
        "Driver": "local",
        "Mode": "z",
        "RW": true,
        "Propagation": ""
    }
],
```
這邊可以看到，Name、Source都和mysql-data相符，也證明了volume的共用性，是可以在容器間共享的。

#### 另外一個指令可以讓兩個容器共用一個volume -> --volumes-from

--volume-from 後面接一個容器名稱，輸入後會讓這個容器的volume跟該容器名稱的volume一樣。
```shell
$ docker container run --detach --name mysql2 --env MYSQL_ROOT_PASSWORD=whatever --volumes-from mysql mysql
```


我們來檢查mysql2的細節
```shell
$ docker container inspect mysql2 

------
"Mounts": [
    {
        "Type": "volume",
        "Name": "mysql-data",
        "Source": "/var/lib/docker/volumes/mysql-data/_data",
        "Destination": "/var/lib/mysql",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
],
```
可以發現跟mysql一模一樣


### 另一種方式：Bind Mount

除了前面的那一種方式外，Bind Mount 是另一種把不屬於映像檔本身的檔案放入容器中的方法，這個功能本身的設計非常酷，第一次使用的時候真的有一種「啊！」的聲音出現！

如同字面上的意思，Bind Mount 就是單純把本機的檔案掛載到容器內，而這個指令沒有辦法寫在Dockerfile裡面，因為必須把一個真實存在的資料夾或是檔案掛載到容器內，所以只能在 `docker container run` 指令的時候加入。


而背後的執行原理，就是 `本機的路徑和容器內的路徑指向本機的同一個檔案`，使用起來的方式很像volume，容器的刪除並不會連帶刪除掉本機的檔案，兩個之間的優先度當然是本機獲勝。

接下來一樣使用nginx來示範Bind Mount的使用情境，不要覺得為什麼又是nginx，因為他最輕鬆把畫面呈現在瀏覽器上。

#### --mount 或是 --volume 都可以

我們進到「ch-05的nginx-example」資料夾

```shell
$ docker container run --detach --name nginx-volume --publish 80:80 --volume $(pwd):/usr/share/nginx/html nginx
```

接著打開 `http://localhost/`，會看到 `Hi, 你成功的使用了 Bind Mount 功能了！` 這一串字，我們是利用資料夾中本地的「index.html」去替換掉nginx原先設定好的index.html。

第一種方式一樣是使用 `--volume` 的指令，但是搭配的是本地端的絕對路徑，而非前面一節的volume名稱，而「$()」的用法，則是在Docker的指令中穿插終端機指令的做法，這裡的PWD在Linux以及macOS代表的是「此處意思」。

所以有兩種方式將volume連接到容器上，第一種是 `--volume 名稱`，並連接到容器上，另外一種是用 `Bind Mount` 將本機的檔案掛載到容器上。

再來我們嘗試第二種實現 Bind Mount 功能：
```shell
$ docker container run --detach --name nginx-bind-mount --mount type=bind,source=$(pwd),target=/usr/share/nginx/html --publish 8080:80 nginx
```
接著打開瀏覽器輸入 `http://localhost:8080/` 可以看到 `Hi, 你成功的使用了 Bind Mount 功能了！` 的畫面，證明確實取代了nginx預設的畫面。

我們來詳細解釋一下這個指令，--mount後面要接三個參數  
(1) type=bind   -> 這個是使用bind的方式      
(2) source=$(pwd) -> 這個是使用者的絕對路徑  
(3) target=/usr/share/nginx/html -> 這個是容器內的絕對路徑   


教完了兩種指令，都是將本地的資料掛載至容器內，接著我們進去容器看看是什麼樣子，由於兩個容器都在背景執行，所以去哪個都可以：
```shell
$ docker container exec --interactive --tty nginx-bind-mount bash

root@6d1cd48097ab:/# cd /usr/share/nginx/html
root@6d1cd48097ab:/usr/share/nginx/html# ls
Dockerfile  LICENSE  README.md  index.html
```

可以看到這個資料夾內的所有檔案，都掛載進來了，這是因為這裡用一整個檔案目錄去取代掉檔案目錄，要注意的地方是，我們不能夠用一個檔案去取代掉檔案目錄，後面會做一個錯誤示範來驗證這件事。     

現在要做的是 `Bind Mount` 最讓人驚艷的功能，現在我們還停留在 nginx-bind-mount 容器內，接著打開終端機的視窗，進入 docker-volume-nginx-example 資料夾，並且隨意新增一個檔案：

```shell
# 另一個終端機的視窗
$ cd docker-volume-nginx-example
$ touch text.txt

# 原本的終端機視窗
root@6d1cd48097ab:/usr/share/nginx/html# ls
Dockerfile  LICENSE  README.md  index.html  text.txt
```

非常有趣的事情發生了，剛剛我們在另一個終端機新增的檔案，即時新增到了容器內，而且完全不用重新啟動容器，也不用做任何指令，這讓Docker在本機開發變成是一件很輕鬆的事情。

只要啟動一個符合應用程式環境的容器，並且把開發的檔案掛載到容器內，一切就搞定了，也不必在容器內去做複雜的設定，因為需要的檔案還是存放在本機中。


Docker Compose
------


### 什麼是Docker Compose

前面每一章節都是在介紹獨立物件的使用，而Docker Compose則是一個整合容器、Volume、虛擬網路並構成應用程式的工具，要使用Docker Compose，須透過撰寫一個叫做「Docker-compose.yml」的YAML檔案，去針對每一個服務做設定，並且透過單一的指令 `docker compose up` 來一鍵啟動所有的服務。

### 為什麼需要 Docker Compose
先想想一個基本的WEB應用程式需要什麼呢？一個WEB SERVER(例如：Nginx、Apache、Traefik...)、一個App SERVER(Rails、Laravel、Django...)、一個資料庫(PostgreSQL、MySQL、MongoDB...)，就算是靜態頁面的產生器，也需要一個WEB SERVER來提供路徑和SSL的驗證。


目前我們確實可以單獨啟動一個容器，並且透過相同的虛擬網路(--network)來做到整合一個應用程式，但如果整個應用程式有6~7個容器要啟動，每一次啟動輸入的參數又是否能確保萬無一失呢？

這正是需要Docker Compose的原因，所有容器的啟動參數，如之前學過的 --publish、--env、--name，volume的名字、虛擬網路的名字等，都能夠被定義在一個文件中，以確保每一次的啟動都是相同的，而且統一在一個檔案中做更動，可避免誤植錯字而啟動失敗的尷尬窘境。


### 啟動 Wordpress

先打開作者幫我們設定好的檔案

```yaml
version: '3.9'

services:
  wordpress:
    image: wordpress:6.0.2
    ports:
      - 8080:80
    environment:
      - WORDPRESS_DB_HOST=db
      - WORDPRESS_DB_USER=admin
      - WORDPRESS_DB_PASSWORD=password
      - WORDPRESS_DB_NAME=wordpress
    volumes:
      - .:/var/www/html

  db:
    image: mysql:5.7
    environment:
      - MYSQL_DATABASE=wordpress
      - MYSQL_USER=admin
      - MYSQL_PASSWORD=password
      - MYSQL_RANDOM_ROOT_PASSWORD=1
    volumes:
      - db:/var/lib/mysql

volumes:
  db:
```



接下來我們來拆解這個檔案

```yaml
version '3.9'
```
這個非常好懂，Docker Compose經過多次版本迭代，自然而然會有很多新功能以及被取代的舊功能，就像是區分API的版本一樣，要注意的是，這個version單純是這份docker compose.yml的紀錄，讓編輯者了解到這份檔案是某一個Compose版本，而Docker Compose並沒有辦法做到根據檔案內撰寫的version來切換版本，最主要還是機器上安裝的Docker Compose支援什麼樣語法和功能。當然，Docker Compose在解析docker compose.yml時，遇到過時或比較新的語法，都會跳出提示來通知使用者。

接著是services，以這個例子來說，我們啟用了wordpress和db兩個service，概念其實和docker container run是一樣的，只是將docker container run需要的參數寫下來，讓docker compose去執行：

```yaml
services:
  wordpress:                            -> 服務名稱 / DNS名稱
    image: wordpress:6.0.2
    ports:
      - 8080:80
    environment:
      - WORDPRESS_DB_HOST=db
      - WORDPRESS_DB_USER=admin
      - WORDPRESS_DB_PASSWORD=password
      - WORDPRESS_DB_NAME=wordpress
    volumes:
      - .:/var/www/html
  db:                                   -> 服務名稱 / DNS名稱
    image: mysql:5.7
    environment:
      - MYSQL_DATABASE=wordpress
      - MYSQL_USER=admin
      - MYSQL_PASSWORD=password
      - MYSQL_RANDOM_ROOT_PASSWORD=1
    volumes:
      - db:/var/lib/mysql      
```



這裡分別是wordpress、db，標記了服務名稱的同時，也作為該服務在docker虛擬網路中的DNS名稱，就像在「介紹Docker虛擬網路」中替容器命名，以便作為DNS和其他容器溝通的手段。

再來是service內部的參數，所有的參數都是官方提供的選項，並不是自己想填什麼就填什麼，具有一定的規範：

```yaml
services:
  wordpress:                            
    image: wordpress:6.0.2
    ports:
      - 8080:80
    environment:
      - WORDPRESS_DB_HOST=db
      - WORDPRESS_DB_USER=admin
      - WORDPRESS_DB_PASSWORD=password
      - WORDPRESS_DB_NAME=wordpress
    volumes:
      - .:/var/www/html
```

* Image: 指定映像檔案的版本，就像在執行容器時一樣。
* Ports: 將指定的port對應到本機的port，如同執行容器時的 --publish 一樣。

有時候參數不只一個，可能會需要打開很多port，這實在YAML檔案的格式中，可以透過這種LIST的方式把需要的參數條列出來：
```yaml
ports:
  - 8080:80
  - 8080:80
  - 8080:80

# 上面的寫法在yaml中叫做List，用「-」作為前綴
```

* Environments: 將環境變數傳入容器中，如同執行容器時的 --env 參數是一樣的道理，同理，如果超過一個環境參數，就用List方式列出來
* Volumes: 以wordpress的例子來說，分別使用了Volume及Bind Mount兩種方法，就如同 --volume在執行容器時所帶入的參數一樣。

在wordpress的service中使用了Bind Mount，將此處綁定至容器中的/var/www/html;而db的service中，則用了Volume的方式建立一個容器外的Volume，作為資料庫的儲存空間。


```yaml
version: '3.9'

services:
    # ...省略
    volumes:                            -> 告訴Docker Compose這個應用程式要用倒的volume
      - .:/var/www/html
  db:                                   -> 我們在db這個service中有使用到volume
    # ...省略
```

這裡的volumes並非容器中的參數，而是整個docker-compose.yml最上層的選項。  
需要提前告知Docker Compose整個服務所使用到的volume，因為在db的服務中有寫入db:var/lib/mysql參數，所以要讓Docker Compose知道有這個volume存在，若是不寫，Docker Compose也會提醒你這個Volume不存在，要記得寫上去   


### 啟動所有服務

```shell
$ docker compose up --detach

------
[+] Running 4/4
 ⠿ Network wordpress-example_default        Created                                                             0.1s
 ⠿ Volume "wordpress-example_db"            Created                                                             0.0s
 ⠿ Container wordpress-example-wordpress-1  Started                                                             2.5s
 ⠿ Container wordpress-example-db-1         Started                                                             2.5s
```

打開瀏覽器前，我們解釋一下這段指令有什麼要注意的地方。
「 Network wordpress-example_default 」這行中，Docker Compose為了讓檔案中被標示的服務能夠執行在一起，預設會在啟動時建立一個虛擬網路，預設會在啟動時建立一個虛擬網路，如同在「Docker虛擬網路」提到的，要讓同一個虛擬網路中的容器，才可以透過容器名稱作為DNS溝通。

至於虛擬網路的命名，預設為「檔案目錄的名稱_default」，當然也可以提前告知Docker Compose要使用的虛擬網路名稱，在後面「部署WEB應用程式」的時候會提到。

接著是「Volume "wordpress-example_db"」這一行，道理如同虛擬網路一樣，因為在docker-compose.yml檔案中，有特別標示了需要volume db，所以docker compose在啟動時預設會用「檔案目錄的名稱_volume名稱」作為命名，如果不想要這麼長的volume名稱，當然也可以，之後會有範例提到，目前則以Docker Compose預設的行為做解釋。

再來是兩個容器的啟動，這裡就相對直覺很多，就是單純啟動容器，命名的規則是「檔案目錄的名稱-服務名稱-編號」。而為啥需要編號呢？因為Docker Compose為了擴展容器數量而預留空格，來做到基本的負載平衡。

最後我們打開瀏覽器，輸入 `http://localhost:8080/`，這裡就可以看到Wordpress網站建置好了。

### 停止所有服務

```shell
$ docker compose down

------
[+] Running 3/2
 ⠿ Container wordpress-example-db-1         Removed                                             2.5s
 ⠿ Container wordpress-example-wordpress-1  Removed                                             1.9s
 ⠿ Network wordpress-example_default        Removed                                             0.1s
```

這樣除了需要手動刪除的volume，容器和虛擬網路都刪除了。




### 深入 Docker Compose


```yaml
version: '3.9'

services:
  ui:
    image: robeeerto/todo-list-ui:latest
    container_name: ui
    restart: on-failure
    networks:
      - frontend
    ports:
      - 3001:3001
    environment:
      - NEXT_PUBLIC_CABLE_URL=${NEXT_PUBLIC_CABLE_URL}
      - NEXT_PUBLIC_API=${NEXT_PUBLIC_API}
    depends_on:
      - api

  api:
    image: robeeerto/todo-list-api:latest
    restart: on-failure
    container_name: api
    ports:
      - 3000:3000
    depends_on:
      - database
      - redis
    networks:
      - frontend
      - backend
    environment:
      - DB_HOST=database
      - DB_USER=${DB_USER}
      - DB_PORT=${DB_PORT}
      - DB_PASSWORD=${DB_PASSWORD}
      - RAILS_ENV=${RAILS_ENV}
      - REDIS_URL=${REDIS_URL}

  redis:
    image: redis:7-alpine
    container_name: redis
    restart: on-failure
    networks:
      - backend
    volumes:
      - redis-data:/data

  database:
    image: postgres:14-alpine
    container_name: database
    restart: on-failure
    networks:
      - backend
    environment:
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - database-data:/var/lib/postgresql/data
      
volumes:
  database-data:
    external: true
  redis-data:
    external: true

networks:
  backend:
    external: true
  frontend:
    external: true
```

上面這是一個基本的WEB應用程式，並且採用前後端分離的方式在本機執行，下面是整個架構：

```md
>     *************************************
>     *                                   *
>     *              Next.js              *        
>     *                                   *
>     *                                   *
>     *                                   *
>    |*-----------------------------------*|
>    |*                                   *|
>    |*                                   *|
>    |*            App  Server            *|
>    |*              (Rails)              *|
>    |*                                   *|
>    |*                                   *|       
>    |*                                   *|
>    |*************************************|
>    |                                     |
>    |      Redis           PostgreSQL     |
>    |                                     |
>    |-------------------------------------|

>  *線框是前端      -線框是後端        *-被包括起來的地方是APP SERVER
```

根據services的階層(從上面的yaml看)，能夠看出這個應用程式總共有四個服務，分別是前端的ui、後端的API、資造庫的DataBase，以及為了實踐WebSocket而用的redis。

這裡有幾個service內的參數是前面沒有提到的，這邊說明一下：


#### Restart
```yaml
api:
  restart: on-failure、always、unless-stopped、no (default)
```
Restart 共有四個參數可使用，分別是 `on-failure、always、unless-stopped、no`

* no: 意味著不論發生什麼情形，容器都不會重新啟動
* always: 除了容器被刪除以外，都將會重新啟動，舉例來說，如果這個容器的初始指令是印出一段文字，那他在進入停止狀態後，他將會被重新啟動
* on-failure: 容器因為非預期錯誤而進入停止狀態時，重新啟動
* unless-stopped: 若是容器結束工作而進入退出狀態，則不會重新啟動，算是僅次於always的一個設定

#### Container_name

```yaml
api:
  container_name: < 你想取什麼都可以 >
```

一般來說，Docker compose預設會以「檔案目錄的名稱 - 服務名稱 - 編號」來替容器命名，在進入容器或是觀看容器的LOGS時，要輸入一長串的容器名稱，所以為了省去麻煩，我們直接在這邊替容器命名就好。

#### networks
```yaml
api:
  networks:
    - frontend
    - backend
```


networks就像在執行容器時加入的 --networks 參數一樣，透過yaml的LIST寫法，也可以定義複數的虛擬網路

#### depends_on
```yaml
api:
  depends_on:
    - database    <- service 的名稱
    - redis       <- service 的名稱
```

depends_on 這個功能非常好用，有一個正式的 WEB應用程式中，很多時候我們需要等待另一個服務的啟動才有效果，上面的範例意味著API服務要等蛋REDIS及DATABASE容器都啟動後，才會進行啟動，這樣可以避免很多非預期的錯誤。

例如： API服務啟動太快，資料庫並沒有準備好，導致伺服器端出現500的錯誤，但其實不是程式碼出錯，而是啟動順序的問題




#### networks -> external: true

```yaml
volumes:
  database-data:
    external: true
  redis-data:
    external: true

networks:
  backend:
    external: true
  frontend:
    external: true
```

這邊要提到的是前面沒有提到的networks，同樣作為最上層的參數，使用方式其實和 volumes 相同，若是有在services內使用到networks參數，都要提前告知docker compose    

而這裡的重點是 `external: true` 參數，意味著該networks或是volume都屬於外部，也就是不隸屬docker-compose.yml之中，需要提前建立的意思，若是加入了 `external: true`，但卻沒有提前建立，Docker Compose將不會按到預設行為，自動建立被標記 `external: true` 的物件






### docker-compose.yml 中的環境變數

我們看一下下面這一段：

```yml
environment:
  - DB_HOST=database
  - DB_USER=${DB_USER}
  - DB_PORT=${DB_PORT}
  - DB_PASSWORD=${DB_PASSWORD}
  - RAILS_ENV=${RAILS_ENV}
  - REDIS_URL=${REDIS_URL}
```

這可能會讓人疑惑，這些看起來像是環境變數的值是從哪來的？Docker Compose有一個非常厲害的功能，是為了避免將機密資訊(資料庫的密碼)寫在docker-compose.yml內，所以在docker compose up時，他能夠自動比對當前資料夾內的.env檔案的環境變數，並且對應到docker-compose.yml內，如此就能夠讓我們安心把docker-compose.yml檔案上傳到GitHub等程式碼儲存庫。


下面這個資料夾來作為簡單的範例：

```md
# 資料夾結構
|----- .env
|----- docker-compose.yml

# .env
DB_USER = robert

# docker-compose.yml
version: '3.9'

services: 
  app:
    image: ...
    environments:
      - DB_USER=${DB_USER}  <- DB_USER=robert
```




### docker compose config 指令

這個指令可以讓 Docker Compose顯示出添加環境變數後的完成docker-compose.yml檔案：

```yaml
# docker-compose.yml  <- TodoList 的範例

$ docker compose config

------
name: todolist-example
services:
  api:
    container_name: api
    depends_on:
      database:
        condition: service_started
      redis:
        condition: service_started
    environment:
      DB_HOST: database
      DB_PASSWORD: ""
      DB_PORT: ""
      DB_USER: ""
      RAILS_ENV: ""
      REDIS_URL: ""
    image: robeeerto/todo-list-api:latest
    networks:
      backend: null
      frontend: null
    ports:
    - mode: ingress
      target: 3000
      published: "3000"
      protocol: tcp
    restart: on-failure
  database:
    container_name: database
    environment:
      POSTGRES_PASSWORD: ""
    image: postgres:14-alpine
    networks:
      backend: null
    restart: on-failure
    volumes:
    - type: volume
      source: database-data
      target: /var/lib/postgresql/data
      volume: {}
  redis:
    container_name: redis
    image: redis:7-alpine
    networks:
      backend: null
    restart: on-failure
    volumes:
    - type: volume
      source: redis-data
      target: /data
      volume: {}
  ui:
    container_name: ui
    depends_on:
      api:
        condition: service_started
    environment:
      NEXT_PUBLIC_API: ""
      NEXT_PUBLIC_CABLE_URL: ""
    image: robeeerto/todo-list-ui:latest
    networks:
      frontend: null
    ports:
    - mode: ingress
      target: 3001
      published: "3001"
      protocol: tcp
    restart: on-failure
networks:
  backend:
    name: backend
    external: true
  frontend:
    name: frontend
    external: true
volumes:
  database-data:
    name: database-data
    external: true
  redis-data:
    name: redis-data
    external: true
```



這個功能非常好用，可以幫助你在 `docker compose up` 之前，檢查所有參數是不是如同預期的一樣，上方可以看到docker compose config出來的結果，比我們所撰寫的還要嚴謹，而一般撰寫的docker-compose.yml已經是非常精簡的版本了。


### docker compose 所有指令

最常使用到的 Docker Compose 指令如下：

```shell
$ docker compose up 
# 根據 docker-compose.yml 的描述啟動理想的應用程式

$ docker compose up --detach
# 根據 docker-compose.yml 的描述啟動理想的應用程式，並且在背景執行

$ docker compose up --detach --build
# 跟上一個差異就在於，若是某個 service 有標記 build 以及存在 Dockerfile，將會建置映像檔在執行 Docker Compose

$ docker compose stop
# 使 docker-compose.yml 內的所有容器進入停止狀態

$ docker compose start
# 使 docker-compose.yml 內的所有停止狀態的容器啟動

$ docker compose down
# 刪除所有 docker-compose.yml 內的容器、虛擬網路 (external: true 不會被刪除)

$ docker compose down --volumes
# 刪除所有 docker-compose.yml 內的容器、虛擬網路、Volume (external: true 不會被刪除)
```


### 啟動應用程式前，先建置映像檔


還記得剛剛這個指令嗎？
```shell
$ docker compose up --detach --build
```

稍微解釋一下 `--build` 這個參數，在本端開發時很常用到它，本地開發時，常會有需要新增相依套件的情形發生，這樣導致每次都需要 `docker image build`、再 `docker image push`，最後才能應用在Docker Compose，這樣超級沒效率。


假設我們目前開發的專案是名為「app」的service，而在資料夾內應該也會有 Dockerfile 才是正確的情形，這時我們就可再 docker-compose.yml 檔案內這樣寫：

```yml
version: '3.9'

# 短寫法
services: 
  app:
    build: .     <- 自動找到當前目錄的 Dockerfile


# 長寫法
services:
  app:
    build:
      context: .    <- 路徑
      dockerfile: Dockerfile  <- 指定 Dockerfile 的檔案
```

* context : 為路徑，「.」的意思代表「此處」，路徑為現在這個資料夾
* dockerfile : 為建置映像檔所需的 Dockerfile，像是測試環境的 Dockerfile 可能就會叫做「Dockerfile.test」，這時若是我們想要利用不同的 Dockerfile 建置不同的環境，就可以特別標示，但若是採用短寫法的話，預設就是找檔名為「Dockerfile」的 Dockerfile。



### Docker Compose 的擴充欄位

說到 docker-compose.yml，還可以透過擴充欄位的方式，來降低許多重複的動作，以及寫錯字的隱藏錯誤，Docker提供的方式叫做 `x-labels`，結合yaml本身的 `<<:` 語法一起使用，能夠大幅減少重複寫法。

看下面的檔案，會發現 `networks` 的欄位是重複撰寫的，這樣在新增服務的時候如果不小心打錯字，會讓你找錯字找的很痛苦：

```yml
version: '3.9'

services: 
  app: 
    image: app
    networks: 
      - production
  db: 
    image: db
    networks: 
      - production
  redis: 
    image: redis
    networks: 
      - production
      
networks:
  production:      
```


這邊我們用 `docker compose config` 看可以看完整的資訊：

```md
> name: self-test
> services:
>   app:
>     image: app
>     networks:
>       production: null
>   db:
>     image: db
>     networks:
>       production: null
>   redis:
>     image: redis
>     networks:
>       production: null
> networks:
>   production:
>     name: self-test_production
```



不過我們可以利用 `x-labels`，統一在最上方管理重複撰寫的地方，並用 `<<:` 將其安插進去：


```yml
version: '3.9'

x-labels: &networks             <- &networks 是這個擴充欄位的命名
  networks:
  - production


services: 
  app: 
    image: app
    <<: *networks               <- 這裡是YAML寫法，像是安插變數一樣

  db: 
    image: db
    <<: *networks               <- 這裡是YAML寫法，像是安插變數一樣

  redis: 
    image: redis
    <<: *networks               <- 這裡是YAML寫法，像是安插變數一樣
      
networks:
  production:      
```

這樣改完之後，我們再用 `docker compose config` 看完整的資訊：

```md
> name: self-test
> services:
>   app:
>     image: app
>     networks:
>       production: null
>   db:
>     image: db
>     networks:
>       production: null
>   redis:
>     image: redis
>     networks:
>       production: null
> networks:
>   production:
>     name: self-test_production
> x-labels:
>   networks:
>   - production
```

可以發現 networks 如同原先的格式那樣被寫進去了，而且我們只要改 `x-labels` 的變數，就可以把檔案內有用到變數全都一起改。

### Docker compose 的複寫檔案

覆寫檔案這個功能，會應用在非常多的場景，大部分的應用程式都會分成不同階段部署，最基本來說要有 `開發環境`、`測試環境`、`正式環境`，以上述情況來說，我們會因此有三種檔案，「docker-compose.yml」(核心)、「docker-compose-dev.yml」(開發)、「docker-compose-production.yml」(正式)。

除了剛剛學過的擴充欄位，來減少撰寫重複欄位，還可以透過複寫檔案來更大幅度減少所需要的內容，因此我們來示範三個階段的檔案內容會有哪些：

#### docker-compose.yml 核心

作為最核心的「docker-compose.yml」，把應用程式會使用到的服務都放進去，而基本設定也都先設定好：

```yml
version '3.9'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: app
    restart: on-failure
    depends_on:
      - db
      - redis
  db:
    image: postgres:14-alpine
    container_name: db
    restart: on-failure
    volumes:
      - database:/var/lib/postgresql/data
  redis:
    image: redis:7-alpine
    container_name: redis
    restart: on-failure
    volumes: 
      -redis:/data

volumes:
  database:
  redis:
```

同樣的，透過 `docker compose config` 來確認整體環境開發：


```md
> name: docker-641
> services:
>   app:
>     build:
>       context: /Users/yee0526/Desktop/5xruby/Docker學習/Docker-Book-Example/docker-6.4.1
>       dockerfile: Dockerfile
>     container_name: app
>     depends_on:
>       db:
>         condition: service_started
>       redis:
>         condition: service_started
>     networks:
>       default: null
>     restart: on-failure
>   db:
>     container_name: db
>     image: postgres:14-alpine
>     networks:
>       default: null
>     restart: on-failure
>     volumes:
>     - type: volume
>       source: database
>       target: /var/lib/postgresql/data
>       volume: {}
>   redis:
>     container_name: redis
>     image: redis:7-alpine
>     networks:
>       default: null
>     restart: on-failure
>     volumes:
>     - type: volume
>       source: redis
>       target: /data
>       volume: {}
> networks:
>   default:
>     name: docker-641_default
> volumes:
>   database:
>     name: docker-641_database
>   redis:
>     name: docker-641_redis
```

確認核心檔案沒問題後，就可以來撰寫 `docker-compose-dev.yml` 開發環境的檔案。



#### docker-compose-dev.yml 開發

```yaml
services: 
  app:
    ports:
     -3000:3000
```

上面就是開發環境要撰寫的內容，非常少，可以透過 --file的指令來覆寫 `docker-compose.yml` 的檔案(我們順便用config指令來看檔案細節)：

```shell
$ docker compose --file ./docker-compose.yml --file ./docker-compose-dev.yml config
```

看一下config產生的資料

```md
> name: docker-641
> services:
>   app:
>     build:
>       context: /Users/yee0526/Desktop/5xruby/Docker學習/Docker-Book-Example/docker-6.4.1
>       dockerfile: Dockerfile
>     container_name: app
>     depends_on:
>       db:
>         condition: service_started
>       redis:
>         condition: service_started
>     networks:
>       default: null
>     ports:
>     - mode: ingress
>       target: 3000                          <- 多了我們在dev覆寫的內容
>       published: "3000"                     <- 多了我們在dev覆寫的內容
>       protocol: tcp
>     restart: on-failure
>   db:
>     container_name: db
>     image: postgres:14-alpine
>     networks:
>       default: null
>     restart: on-failure
>     volumes:
>     - type: volume
>       source: database
>       target: /var/lib/postgresql/data
>       volume: {}
>   redis:
>     container_name: redis
>     image: redis:7-alpine
>     networks:
>       default: null
>     restart: on-failure
>     volumes:
>     - type: volume
>       source: redis
>       target: /data
>       volume: {}
> networks:
>   default:
>     name: docker-641_default
> volumes:
>   database:
>     name: docker-641_database
>   redis:
>     name: docker-641_redis
```


可以看到上方的檔案，就是覆寫過後的結果，多了 port 的資料，而剛剛我們覆寫的指令，覆寫的的順序是由後方的覆蓋掉前方，以上方的開發環境例子來看，最前方的 `--file ./docker-compose.yml` 會是核心檔案，而後方的 `--file ./docker-compose-dev.yml` 則是開發環境要覆寫的檔案，在開發環境下，只要確認啟動的應用程式開啟port去對應到本機的port即可。



#### docker-compose-production.yml 正式

到了正式環境，就不會把易用程式的port(3000)打開到機器上，而是透過反向代理伺服器來做到發布外網的功能，以一個WEB應用程式來說，只需要開啟 port 80 以及 443，讓使用者能夠透過HTTP協定進入網站即可。

而在正式環境，應用程式也不再透過build方式來建置映像檔，而是透過已經建置好的映像檔來運作，這時候的映像檔會放在像是 DockerHub 之類的映像檔儲存庫上。

下方是 `docker-compose-production.yml` 的檔案內容，這裡的主要目的是試著練習檔案來覆寫功能，所以重點不在 `traefik` 的設定，後面會在說明這個反向代理伺服器。

```yml
services:
  proxy:
    image: traefik:v2.8
    container_name: proxy
    ports:
      - 80:80
      - 443:443
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

app:
  image: app:production
```

接著一樣用 `--file` 方式複寫檔案，並用 config 來看檔案細節：

```shell
$ docker compose --file ./docker-compose.yml --file ./docker-compose-production.yml config
```

config後的資訊：
```md
> name: docker-641
> services:
>   app:                                <- 在正式環境中，沒有再開啟port了
>     build:
>       context: /Users/yee0526/Desktop/5xruby/Docker學習/Docker-Book-Example/docker-6.4.1
>       dockerfile: Dockerfile
>     container_name: app
>     depends_on:
>       db:
>         condition: service_started
>       redis:
>         condition: service_started
>     image: app:production
>     networks:
>       default: null
>     restart: on-failure
>   db:
>     container_name: db
>     image: postgres:14-alpine
>     networks:
>       default: null
>     restart: on-failure
>     volumes:
>     - type: volume
>       source: database
>       target: /var/lib/postgresql/data
>       volume: {}
>   proxy:                              <- 新增的服務被覆寫上去了
>     container_name: proxy
>     image: traefik:v2.8
>     networks:
>       default: null
>     ports:
>     - mode: ingress
>       target: 80
>       published: "80"
>       protocol: tcp
>     - mode: ingress
>       target: 443
>       published: "443"
>       protocol: tcp
>     volumes:
>     - type: bind
>       source: /var/run/docker.sock
>       target: /var/run/docker.sock
>       bind:
>         create_host_path: true
>   redis:
>     container_name: redis
>     image: redis:7-alpine
>     networks:
>       default: null
>     restart: on-failure
>     volumes:
>     - type: volume
>       source: redis
>       target: /data
>       volume: {}
> networks:
>   default: 
>     name: docker-641_default
> volumes:
>   database:
>     name: docker-641_database
>   redis:
>     name: docker-641_redis
```



### 更多有關 docker-compose.yml 的參數

由於 docker-compose.yml 中可以加入的參數太多了，所以建議直接閱讀官方文件，這邊有一堆參數可以使用：

網頁連結：[https:docs.docker.com/compose/compose-file](https:docs.docker.com/compose/compose-file)

#### 為何不在核心檔案加入 port

因為port沒辦法被覆蓋掉，如果我們在核心檔案開啟 `port 3000`，用透過複寫的方式話，到了正式環境還是會開啟 `port 3000`，而因為這會牽扯到安全問題，一個WEB應用程式應該把port收斂到80、443。


#### 為什麼在正式環境還是有 build

這沒有辦法，通常在開發環境中，我們會透過build的方式，來因應添加套件或是檔案更動的情形，所以去重新建置映像檔是一件經常發生的事，但在正式環境中，因為我們有指定映像檔，docker compose會以映像檔為主，而不是去找 Dockerfile 來建置，所以這不會造成問題。




Docker Swarm
------

經過前面介紹，我們已經可以用Docker、Docker Compose來打包和執行應用程式，在正式部署應用程式前，我們需要了解如何建立一個高可用性且穩定的環境給正式環境的應用程式。   

在正式環境下，Docker Compose就會有一點無力，只能運作在單台機器的特性，導致當機器崩潰時，應用程式就會喪失所有服務，容器也將隨之消失，在一些小型的服務或是自己的SideProject，還可以接受這損失，但是如果今天是大型專案，就無法接受了。  

這是我們需要一個工具來幫助我們管理所有的機器和容器，Docker Swarm就是Docker給出關於容器調度的解答。


### Docker Swarm 模式

```shell
$ docker swarm init

------


```

### Docker Swarm 指令說明

```md
> Swarm initialized: current node (x0dvbi6j7r637ifq6l6nzaqhb) is now a manager.
```
Docker 告訴我們：「現在這個node是一個manager」，這裡出現了兩個在之前張別沒出現過的名詞

* Node(節點)：在Docker Swarm 中的機器，以目前的情況來說，這個Docker Swarm有一個節點，也就是你的電腦
* Manager(管理者)：擁有較高權限的節點，如同其名，他擁有管理整個 Swarm 的資格

再來看第二段
```md
> To add a worker to this swarm, run the following command:
> 
>     docker swarm join --token SWMTKN-1-213g0voehmedder5skxd489k2amz4thkhilpimhy6rea8zx4ko-ejmfaossjb3iq3yyepn3i3uh9 192.168.65.3:2377
```
Docker 這一段說：「可透過下列指令，把worker加入到現在這個Swarm中」，那worker又是什麼呢？

* Worker(工作者)：純粹拿來執行服務的節點，就是產線中的工作者，只接受管理者派下來的任務並執行

我們直接把這段貼到終端機看看
```shell
$ docker swarm join --token SWMTKN-1-213g0voehmedder5skxd489k2amz4thkhilpimhy6rea8zx4ko-ejmfaossjb3iq3yyepn3i3uh9 192.168.65.3:2377

------
Error response from daemon: This node is already part of a swarm. Use "docker swarm leave" to leave this swarm and join another one.
```

可以看到Docker回應：「目前這個節點已經是Swarm的一部分，如果使用 `docker swarm leave` 指令，就可以離開Swarm」。

先試著離開看看
```shell
$ docker swarm leave

------
Error response from daemon: You are attempting to leave the swarm on a node that is participating as a manager. Removing the last manager erases all current state of the swarm. Use `--force` to ignore this message.
```
又失敗了，Docker告訴我們：「因為這個節點是最後一個manager，離開Swarm的話，會造成目前Swarm的狀態遺失，所以如果要離開的話，請加入 --force 指令」。

加入 --force 先離開 Swarm：

```shell
$ docker swarm leave --force

------
Node left the swarm.
```

### Docker Swarm 的狀態

剛才要離開Swarm的時候，Docker說：「如果最後一個manager也離開的話，就會造成目前Swarm的狀態遺失」，這是什麼意思呢？還記得前面我們說過有關Docker的有狀態和無狀態嗎？最重要的是「資料的永續性」，以便讓某一個物件保持狀態。

我們可以看下面的圖：

上半部是manager的區塊，有著一個 Internal distributed state store (分散式儲存庫)，而每一個manager都和資料庫有著連接。

```md
> Docker Swarm 架構圖

>               |--------------------------------------------------------------|         
>               |                                         Raft consensus group |         
>               |    -----------------------------------------------------     |
>               |    |         Internal distributed state store          |     | 
>               |    ------|-------------------|--------------------|-----     |
>               |          |                   |                    |          |
>               |    ------|-----        ------|-----         ------|-----     |
>               |    |  Manager | <----> |  Manager | <-----> |  Manager |     | 
>               |    --|---|-----        --|-----|---         ------|-----     |
>               |      |   |               |     |                  |          |
>               |------|---|---------------|-----|------------------|-----------
>                      |   |               |     |                  |                        
>            |- - - - -|   | | - - - - - - |     | - - - - -|       | - - - |                
>            |             - | - - - - - - - |              |               |                
> |----------|---------------|---------------|--------------|---------------|---------------|
> |          |               |               |              |               |               |
> |    -----------     -----------     -----------     -----------     -----------          |
> |    |  Worker |     |  Worker |     |  Worker |     |  Worker |     |  Worker |          |
> |    -----------     -----------     -----------     -----------     -----------          |
> |                                                                         Gossip network  |
> |-----------------------------------------------------------------------------------------|
```


這個分散式儲存庫是拿來幹嘛的呢？其功能主要是紀錄整個Docker Swarm的詳細資訊，包含應用程式的預期狀態，worker 所回傳的資料(服務是否完成)、有哪些節點屬於這個Swarm、哪些估做要分配給哪些節點等...，可以說是整個Swarm保持狀態的核心，這也是為什麼Docker會警告最後一個manager離開時，會造成目前Swarm的狀態遺失，因為最後一個manager離開，整個Docker Swarm也會解散，之前儲存的資料也會被移除。    

接著，稍微提一下，架構圖的右上角，有一段 `Raft consensus group`，Raft本身是一種非常有趣的演算法(共識演算法)，目的是當一個叢集中有多個節點組成時，讓每個節點都維護同樣的狀態，這也是Docker Swarm如何讓不同的manager間可以達到一制性的原因，這裡就先不深入探討，只要知道，用Raft演算法，可以讓Manager Node之間的資訊保值一致。   

### Swarm 模式下的容器

前幾章節，我們都是透過 `docker container run` 來執行一個獨立的容器，而在 Docker Swarm 的模式之下，有了新的物件可以來執行容器


#### 可執行多個容器的 service 物件

我們先回到 Swarm 模式：
```shell
$ docker swarm init

------
Swarm initialized: current node (p8ye1z8ohg1m6od3zxz7gzrr1) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-4qgt8w3nfgmfudhnig5w0pkahpfvfo7qy78m3s3ky9l8334fhk-3m1sk2aou976uavq1oug7nj9r 192.168.65.3:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
這個新物件叫做 「service」(服務)，在前面的指令 `docker container run` 都只能夠一次啟動一個容器，缺乏彈性，而 `docker service create` 則是拿來解決這個問題的答案。
```

輸入下方指令：

```shell
$ docker service create --name nginx --publish 8080:80 --detach --replicas 3 nginx

------
gz64vyff82ss1ayuj4ptorvf4
```


接著使用 `docker service list` 指令來列出 service 物件：

```shell
$ docker service list

------
ID             NAME      MODE         REPLICAS   IMAGE          PORTS
o4o2zyv1564m   nginx     replicated   3/3        nginx:latest   *:8080->80/tcp
```

啟動service的時候，多了一個參數`--replicas`，他就是「副本」的意思，這裡要求要複製三個nginx的映像檔執行而成的容器：


```md
> Docker Service
>
>                                                    task                  container                     
>                                                     |                        |                         
>                                             --------|------------------------|--------------------     
>                                             |       |                --------|-------            |               
>                                =========>   |    nginx. 1            | nginx:latest |            |               
>                               /             |                        ----------------            |               
>      |----------------------|               ------------------------------------------------------                     
>      |      service         |                                    available node                                  
>      |  | - - - - - - - - | |                                                                                           
>      |  |                 | |                                                                                    
>      |  |    3 nginx      | |               ------------------------------------------------------               
>      |  |    replicas     | |               |                        ----------------            |               
>      |  |                 | |   ========>   |    nginx. 1            | nginx:latest |            |               
>      |  | - - - - - - - - | |               |                        ----------------            |                 
>      |      swarm manager   |               ------------------------------------------------------               
>      |----------------------|                                    available node                                       
>                              \                                                                                    
>                               \                                                                                   
>                                \            ------------------------------------------------------                 
>                                 \           |                        ----------------            |               
>                                   ======>   |    nginx. 1            | nginx:latest |            |               
>                                             |                        ----------------            |               
>                                             ------------------------------------------------------               
>                                                                  available node                                  
>                                                  
>                                                  
>                                                  
```

在Manager Node上執行了service指令，這個指令將會根據副本的數量雃拆分成數量的任務，以這個例子來說，就是建立了一個 Service，而這個 Service 裡面則有三個任務，每個任務裡面都是 nginx:latest映像檔執行而成的容器。

Manager Node 會想按照擁有的節點數量，想辦法分配到每一個節點上，而因為目前在自己電腦上執行，所以只有一個節點，這樣會把三個容器都座落在這個節點上。

我們把所有啟動的容器列出來：
```shell
$ docker container list

------
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
4885c19d0984   nginx:latest   "/docker-entrypoint.…"   18 minutes ago   Up 18 minutes   80/tcp    nginx.2.edvsbplc1pkj23ssuvej6uzda
8a846e414af7   nginx:latest   "/docker-entrypoint.…"   18 minutes ago   Up 18 minutes   80/tcp    nginx.1.hyzhmpwgxxopbvyyad5rdaict
f52e063ba963   nginx:latest   "/docker-entrypoint.…"   18 minutes ago   Up 18 minutes   80/tcp    nginx.3.kxu8fnvhxzrfl8khgpszsl3qb
```

可以發現有三個容器正在執行中，


### Docker Swarm 分配任務的流程

前面有提到 docker Swarm 會平均分配任務到每一個節點上，那他是如何做到的呢？我們看以下的圖片：


```md
> Swarm 分配的簡易流程圖

>       Docker Engine client                                                                                                                          
>       | - - - - - - - - - - - - - - - - - - - |                                                                                                                   
>       |        docker service create          |                                                                                                  
>       | - - - - - - - - - - - - - - - - - - - |
>                                                                                                                                             
>                                                                                                                                             
>                   Swarm manager                                                                                                                         
>                   | - - - - - - - - - - - - - |                                                                                          
>                   | RAFT                      |                                                                                  
>                   |   | - - - - - - - - - -|  |                                                                                                                 
>                   |   |        API         |  |     accepts command and creates service object 
>                   |   | - - - - - - - - - -|  |                                                                                                                 
>                   |                           |                                                                                  
>                   |   | - - - - - - - - - -|  |                                                                                                                 
>                   |   |    orchestrator    |  |     reconciliaction loop  that creates tasks for service objects
>                   |   | - - - - - - - - - -|  |                                                                                  
>                   |                           |                                                                                                                 
>                   |   | - - - - - - - - - -|  |                                                                                                                 
>                   |   |    allocator       |  |     alloactes ip addresses to tasks
>                   |   | - - - - - - - - - -|  |                                                                                                
>                   |                           |                                                                                                                 
>                   |   | - - - - - - - - - -|  |                                                                                                                 
>                   |   |    dispatcher      |  |     assigns tasks to nodes                                                                                           
>                   |   | - - - - - - - - - -|  |                                                                                                
>                   |                           |                                                                                                                 
>                   |   | - - - - - - - - - -|  |                                                                                                                 
>                   |   |    scheduler       |  |     instructs a worker to run a task
>                   |   | - - - - - - - -|- -|  |                                                                                                
>                   | - - - - - - - - - -|- - - |                                                                    
>                                        |                                                                                   
>                                        |                                                                                   
>                   worker node          |                                                                                   
>                   | - - - - - - - - - -|- - - |                                                                                          
>                   |    - - - - - - -   |      |                                                                                  
>                   |    | container |   |      |                                                                                  
>                   |    - - -|- - - -   |      |                                                                                  
>                   |         |          |      |                                                                                  
>                   |   | - - | - - - - -|- -|  |                                                                                                                 
>                   |   |     worker         |  |     connects to dispatcher to check for assigned tasks
>                   |   | - - - - - - - - - -|  |                                                                                                                 
>                   |                           |                                                                                  
>                   |   | - - - - - - - - - -|  |                                                                                                                 
>                   |   |    executor        |  |                                                                                                
>                   |   | - - - - - - - - - -|  |      executes tasks assigned to worker node                                                                          
>                   | - - - - - - - - - - - - - |
> 
```

在 client 端(自己的電腦)，把 `docker service create` 指令送到 Docker Swarm 中，而 `Manager Node` 會接受這個 Service 的預期狀態，以前面例子來說，預期狀態就是對應到本機的port 8080以及命名為nginx還要在背景執行，並且要有三個副本。

接著會進入到(Orchestrator 調度者)，負責建立 Service，不斷監視著其建立的 Service 物件，並且想辦法維持 Service 的預期狀態，假設今天某一個節點的nginx容器壞掉了，這時候Orchestrator就會發現，並且補上另外一個nginx容器，來滿足預期狀態中的三個副本。

下面可以實驗一下，看Orchestrator有沒有在偷懶，剛剛有確認過現在本機有三個容器，而 `docker service list` 也顯示 `replicas 3/3`，也就是滿足預期狀態的三個副本。

這時有一個推薦的工具叫做「watch」，可以自動刷新某一個指令的結果，macOS的讀者們可以透過 `brew install watch` 來安裝這個工具。

接著可以打開兩個終端機，其中一個輸入下方指令，就可以不用手動一直輸入 `docker service list` 來看狀態的改變：

```shell
# 第一個終端機

$ watch docker service list

------
ID             NAME      MODE         REPLICAS   IMAGE          PORTS
o4o2zyv1564m   nginx     replicated   3/3        nginx:latest   *:8080->80/tcp
```
另一個終端機則是列出所有的容器，並且用 Container ID 強制刪除其中一個：

```shell
# 第二個終端機

$ docker container list

------
CONTAINER ID   IMAGE          COMMAND                  CREATED             STATUS             PORTS     NAMES
4885c19d0984   nginx:latest   "/docker-entrypoint.…"   About an hour ago   Up About an hour   80/tcp    nginx.2.edvsbplc1pkj23ssuvej6uzda
8a846e414af7   nginx:latest   "/docker-entrypoint.…"   About an hour ago   Up About an hour   80/tcp    nginx.1.hyzhmpwgxxopbvyyad5rdaict
f52e063ba963   nginx:latest   "/docker-entrypoint.…"   About an hour ago   Up About an hour   80/tcp    nginx.3.kxu8fnvhxzrfl8khgpszsl3qb

============

$ docker container rm --force 4885c19d0984

------
4885c19d0984
```

這時候馬上看向第一個終端機，會發現變這樣，REPLICAS 變成 2/3 了，這代表 Orchestrator 沒有在偷懶，他有檢查到突然少一個副本：
```shell
ID             NAME      MODE         REPLICAS   IMAGE          PORTS
o4o2zyv1564m   nginx     replicated   2/3        nginx:latest   *:8080->80/tcp
```

不過大約3秒過後，終端機變這樣，副本變回 3/3：
```shell
ID             NAME      MODE         REPLICAS   IMAGE          PORTS
o4o2zyv1564m   nginx     replicated   3/3        nginx:latest   *:8080->80/tcp
```

這就是 Orchestrator 在做的事情，不斷檢查其建立的 Service 物件有沒有出現不符合預期的情況發生，一但發現，就會重複圖中 Manager Node 的流程，透過 allocate 來分配 IP位置給任務，然後dispatcher會把任務傳遞給Orchestrator分配好的節點，最後scheduler會告訴Worker Node該如何執行這個任務，而Worker Node除了乖乖執行任務外，還會回報任務的執行狀況給Manager Node讓整個Swarm保持在一個不斷同步資訊的情況下。


### Docker Swarm 指令

```shell
$ docker node list

------
ID                            HOSTNAME         STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
0vwlovz5h98riopke2i62za6h *   docker-desktop   Ready     Active         Leader           20.10.21
```

還記得我們一開始執行 `docker swarm init` 後，會產生一段特別的 Token，讓我們可以把其他的機器加入Swarm成為節點，但若不是在執行 `docker swarm init`時加入電腦，而是在運作一段時間後，才想加入新節點，要怎麼做呢？

透過 `docker swarm join-token` 指令，後方可以放 manager 或是 woker 兩個參數，就會產生相對應的token，讓我們可以產生新的機器到Swarm中：

```shell
$ docker swarm join-token manager/woker
```

還記得幾面我們在使用容器時，若是要更新容器的設定，都需要重新啟動容器，以便新的設定可以套用在容器中，而在Swarm模式下的service，已經不需要這麼做了，可以使用docker service update指令，讓 Swarm 幫我們更新設定和重啟容器，且中間完全不會有任何的空擋出現。


```shell
$ docker service update --env-add NAME=robert nginx

------
nginx
overall progress: 3 out of 3 tasks 
1/3: running   [==================================================>] 
2/3: running   [==================================================>] 
3/3: running   [==================================================>] 
verify: Service converged 
```

還可以透過 `docker service ps` 的指令，來看到 Swarm 替換容器的過程：

```shell
$ docker service ps nginx

------
ID             NAME          IMAGE          NODE             DESIRED STATE   CURRENT STATE                 ERROR                         PORTS
v4yvewcjklwo   nginx.1       nginx:latest   docker-desktop   Running         Running about a minute ago                                  
hyzhmpwgxxop    \_ nginx.1   nginx:latest   docker-desktop   Shutdown        Shutdown about a minute ago                                 
v855qs71j6lf   nginx.2       nginx:latest   docker-desktop   Running         Running about a minute ago                                  
1ix7gqnvj7ua    \_ nginx.2   nginx:latest   docker-desktop   Shutdown        Shutdown about a minute ago                                 
zo2wggoegybz   nginx.3       nginx:latest   docker-desktop   Running         Running about a minute ago                                  
kxu8fnvhxzrf    \_ nginx.3   nginx:latest   docker-desktop   Shutdown        Shutdown about a minute ago     
```

透過終端機畫面，可以很清楚看出有三個舊容器被替換掉了，當然也要來驗證看看環境變數是否有更改成功：
```shell
$ docker container list

------
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS              PORTS     NAMES
900fb311ed1c   nginx:latest   "/docker-entrypoint.…"   2 minutes ago   Up About a minute   80/tcp    nginx.3.zo2wggoegybz6egdm099ftn4g
44ef8ea1852b   nginx:latest   "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes        80/tcp    nginx.1.v4yvewcjklwo5fua0b3rtn588
9970ded6c649   nginx:latest   "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes        80/tcp    nginx.2.v855qs71j6lfdo43towcx6583


==============
$ docker container exec --interactive --tty 900fb311ed1c bash

------
root@900fb311ed1c:/# echo $NAME
robert
```

確實新增一個環境變數了，這是怎麼辦到的呢？


### Service 的滾動更新

Docker Swarm 預設更新 Service 的模式，是先從編號1容器開始關閉，並且啟動擁有新設定的編號1容器，接著才是編號2，以此類推，所以無時無刻都會有nginx服務的容器存在，服務才不會斷擋。   

不過今天是因為這個service有三個副本，假如今天只有一個副本呢？  

這時候可以使用 --update-order 參數，我們可以告訴 Swarm 所希望更新時的順序，這裡 Docker 官方有兩種設定，分別是「start-first」、「stop-first」。   

start-first 和前面提到的更新模式不同，不會先關閉編號1的容器，而是先製作一個擁有新設定的編號1容器，才關閉就的編號1容器，這裡一樣可以使用前面介紹的watch套件觀看，在 `docker container list` 時會看到，突然變遲四個nginx的容器，接著又變成三個，至於stop-first，則和預設的行為一樣，採取先關閉、後啟動的方式。   

其實，更新還有很多參數可以嘗試，之後我們會寫得更加詳細，這裡是希望大家對於Service更具有概念一點，在Swarm的模式下，可以想像成一個功能更強的容器。   

### 刪除 Service

我們來結束 service 的生命週期：

```shell
$ docker service rm nginx

------
nginx
```

最棒的是，Swarm會自動幫我們把容器加上 --rm 的指令，也就是我們在結束service後，不用擔心容器沒被刪除。  


正式建立叢集
------

這裡會用 Digital Ocaen 雲端架構供應商所提供的虛擬伺服器作為叢集的教學。


到 Digital Ocaen 網站，建立好三台 DROPLETS (3) 後，我們就會拿到三個`ip位置`，接著我們用ssh的方式連接到這三台機器：
Ps. 前提是前面建立機器的時候，是用ssh的方式作加密


```shell
# 連結的格式為 ssh + root(使用者名稱) + 伺服器IP位置(這個就是我們剛剛見好三台機器後，他們的IP位置)
$ ssh root@157.245.51.137 

------
The authenticity of host '157.245.51.137 (157.245.51.137)' can't be established.
ED25519 key fingerprint is SHA256:Nz6w7AIGjDDYjzIFCUIx6BX40wUVgfA0DiwSqOGfHJE.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '157.245.51.137' (ED25519) to the list of known hosts.
Welcome to Ubuntu 22.10 (GNU/Linux 5.19.0-23-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed May 10 05:17:34 UTC 2023

  System load:  0.04541015625     Users logged in:       0
  Usage of /:   17.1% of 9.52GB   IPv4 address for eth0: 157.245.51.137
  Memory usage: 38%               IPv4 address for eth0: 10.15.0.5
  Swap usage:   0%                IPv4 address for eth1: 10.104.0.2
  Processes:    97

0 updates can be applied immediately.



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# 
```

到這邊就正式進入到 DigitalOcean 的伺服器中。其他兩個伺服器也要比照辦理。

要在每一台伺服器上安裝Docker，就像回到一開始的「安裝 Docker」，因為這裡的作業系統都是 Ubuntu，所以我們直接使用腳本來安裝最新版本的Docker：
```shell
# ...省略
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# curl -fsSL https://get.docker.com -o get-docker.sh
# 輸入後沒有反應是正常的，這裡指下載了腳本


# 接著指令下載 docker
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# sh get-docker.sh
# Executing docker install script, commit: a8a6b338bdfedd7ddefb96fe3e7fe7d4036d945a
+ sh -c apt-get update -qq >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq apt-transport-https ca-certificates curl >/dev/null
Scanning processes...                                                                                                                                                                     
Scanning candidates...                                                                                                                                                                    
Scanning processor microcode...                                                                                                                                                           
Scanning linux images...                                                                                                                                                                  
+ sh -c mkdir -p /etc/apt/keyrings && chmod -R 0755 /etc/apt/keyrings
+ sh -c curl -fsSL "https://download.docker.com/linux/ubuntu/gpg" | gpg --dearmor --yes -o /etc/apt/keyrings/docker.gpg
+ sh -c chmod a+r /etc/apt/keyrings/docker.gpg
+ sh -c echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu kinetic stable" > /etc/apt/sources.list.d/docker.list
+ sh -c apt-get update -qq >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-ce-rootless-extras docker-buildx-plugin >/dev/null
Scanning processes...                                                                                                                                                                     
Scanning candidates...                                                                                                                                                                    
Scanning processor microcode...                                                                                                                                                           
Scanning linux images...                                                                                                                                                                  
+ sh -c docker version
Client: Docker Engine - Community
 Version:           23.0.6
 API version:       1.42
 Go version:        go1.19.9
 Git commit:        ef23cbc
 Built:             Fri May  5 21:18:32 2023
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          23.0.6
  API version:      1.42 (minimum version 1.12)
  Go version:       go1.19.9
  Git commit:       9dbdbd4
  Built:            Fri May  5 21:18:32 2023
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.21
  GitCommit:        3dce8eb055cbb6872793272b4f20ed16117344f8
 runc:
  Version:          1.1.7
  GitCommit:        v1.1.7-0-g860f061
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0

================================================================================

To run Docker as a non-privileged user, consider setting up the
Docker daemon in rootless mode for your user:

    dockerd-rootless-setuptool.sh install

Visit https://docs.docker.com/go/rootless/ to learn about rootless mode.


To run the Docker daemon as a fully privileged service, but granting non-root
users access, refer to https://docs.docker.com/go/daemon-access/

WARNING: Access to the remote API on a privileged Docker daemon is equivalent
         to root access on the host. Refer to the 'Docker daemon attack surface'
         documentation for details: https://docs.docker.com/go/attack-surface/

================================================================================
```

安裝好後，養成好習慣，使用 `docker version` 和 `docker compose version` 兩個指令確認安裝結果，確認這台機器是否有成功安裝 docker：
```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker version

Client: Docker Engine - Community
 Version:           23.0.6
 API version:       1.42
 Go version:        go1.19.9
 Git commit:        ef23cbc
 Built:             Fri May  5 21:18:32 2023
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          23.0.6
  API version:      1.42 (minimum version 1.12)
  Go version:       go1.19.9
  Git commit:       9dbdbd4
  Built:            Fri May  5 21:18:32 2023
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.21
  GitCommit:        3dce8eb055cbb6872793272b4f20ed16117344f8
 runc:
  Version:          1.1.7
  GitCommit:        v1.1.7-0-g860f061
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0

```

查看 docker compose 版本：
```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker compose version
Docker Compose version v2.17.3
```


第一台裝好，記得其他兩台也要做一樣的事情：

```shell
# 第二台ip
$ ssh root@157.245.51.139

# 第三台ip
$ ssh root@157.245.51.150


# 分別透過上方指令進入兩者機器後，安裝docker
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-02:~# curl -fsSL https://get.docker.com -o get-docker.sh

# 接著指令下載 docker
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-02:~# sh get-docker.sh
```


三台都安裝後，可以挑其中一台來執行 `docker swarm init`：

```shell
# 我挑第一台
ssh root@157.245.51.137

# 輸入 docker swarm init
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker swarm init
Error response from daemon: could not choose an IP address to advertise since this system has multiple addresses on interface eth0 (157.245.51.137 and 10.15.0.5) - specify one with --advertise-addr
```

不過輸入這個指令，會有錯誤訊息：「這個機器的網路介面上有太多的IP位置，叫我們綁定其中一個」，使用顯示在主控台上的IP位置，並且搭配 --advertise-addr 參數來使用：

```shell
# 加上這個指令  --advertise-addr 157.245.51.137 
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker swarm init --advertise-addr 157.245.51.137
Swarm initialized: current node (fxtjyolpe6vsitbr5zmi0lzeu) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-2irj647ax0c3wf13q0spyxce782pojy9swxx2x4prc30zc9x6h-bh2f6amfzqyljciu9vgn2qf0y 157.245.51.137:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```
這樣我們就成功開啟了 Swarm 模式。接著把其他伺服器也加入 Swarm 中，複製整段指令貼到另外兩台伺服器：

```shell
# 先在終端機各自進入兩台機器

# 第二台
$ ssh root@157.245.51.139 

root@ubuntu-s-1vcpu-512mb-10gb-sgp1-02:~# docker swarm join --token SWMTKN-1-2irj647ax0c3wf13q0spyxce782pojy9swxx2x4prc30zc9x6h-bh2f6amfzqyljciu9vgn2qf0y 157.245.51.137:2377
This node joined a swarm as a worker.

# 第三台
$ ssh root@157.245.51.150

root@ubuntu-s-1vcpu-512mb-10gb-sgp1-02:~# docker swarm join --token SWMTKN-1-2irj647ax0c3wf13q0spyxce782pojy9swxx2x4prc30zc9x6h-bh2f6amfzqyljciu9vgn2qf0y 157.245.51.137:2377
This node joined a swarm as a worker.
```

把另外兩台加入後，回到第一台：

```shell
$ ssh root@157.245.51.137

# 輸入 docker node list
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker node list

------
ID                            HOSTNAME                            STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
fxtjyolpe6vsitbr5zmi0lzeu *   ubuntu-s-1vcpu-512mb-10gb-sgp1-01   Ready     Active         Leader           23.0.6
ryxvpsy0fdcgwkq7b2qwjjyiq     ubuntu-s-1vcpu-512mb-10gb-sgp1-02   Ready     Active                          23.0.6
h10hi4yxdtp06wj0l4sy97088     ubuntu-s-1vcpu-512mb-10gb-sgp1-03   Ready     Active                          23.0.6
```


可以看到有三個 node 顯示出來，接著我們到第二台，輸入一樣的指令試試：

```shell
$ ssh root@157.245.51.139 

# 輸入 docker node list
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-02:~# docker node list
Error response from daemon: This node is not a swarm manager. Worker nodes can't be used to view or modify cluster state. Please run this command on a manager node or promote the current node to a manager.
```

會發現這個錯誤，原因是因為，這些被加入的節點不是 manager 的角色，所以不能夠柴看節點的列表， Docker 指示我們去 manager 的節點執行這個指令，或是把現在這個節點升級成 manager，那就讓我們來試著替這節點升級：

```shell
# 先回到第一個節點
$ ssh root@157.245.51.137

# 指令是 docker node promote 節點ID -> 下面那個是我們輸入 docker node list 時，有出現的 Node ID
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker node promote ryxvpsy0fdcgwkq7b2qwjjyiq
Node ryxvpsy0fdcgwkq7b2qwjjyiq promoted to a manager in the swarm.
```

再來，我們在第一台機器，打上node list指令：
```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker node list
ID                            HOSTNAME                            STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
fxtjyolpe6vsitbr5zmi0lzeu *   ubuntu-s-1vcpu-512mb-10gb-sgp1-01   Ready     Active         Leader           23.0.6
ryxvpsy0fdcgwkq7b2qwjjyiq     ubuntu-s-1vcpu-512mb-10gb-sgp1-02   Ready     Active         Reachable        23.0.6
h10hi4yxdtp06wj0l4sy97088     ubuntu-s-1vcpu-512mb-10gb-sgp1-03   Ready     Active                          23.0.6
```

發現剛剛promote的指令，MANAGER STATUS變成 `Reachable`，這個狀態和Leader差在哪呢？這和我們前面提過的Raft(共識演算法)有關，Raft演算法需要一個 Leader 來接受資訊並調度，之後再將資訊和其他節點同步，而其他的節點則為 Reachable 節點，但實際上他們的職位都是 Manager，可以做的事情也一樣，只是在底層運作的角色有些不一樣而已。

接著，我們再把第三台機器，也變成Reachable狀態：
```shell
# 因為剛剛第二台已經變成 maanger，所以第二台也有權限改變機器的狀態

ssh root@157.245.51.139

# 輸入 promote 指令
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-02:~# docker node promote h10hi4yxdtp06wj0l4sy97088
Node h10hi4yxdtp06wj0l4sy97088 promoted to a manager in the swarm.

# 再來輸入 node list 指令
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-02:~# docker node list

ID                            HOSTNAME                            STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
fxtjyolpe6vsitbr5zmi0lzeu     ubuntu-s-1vcpu-512mb-10gb-sgp1-01   Ready     Active         Leader           23.0.6
ryxvpsy0fdcgwkq7b2qwjjyiq *   ubuntu-s-1vcpu-512mb-10gb-sgp1-02   Ready     Active         Reachable        23.0.6
h10hi4yxdtp06wj0l4sy97088     ubuntu-s-1vcpu-512mb-10gb-sgp1-03   Ready     Active         Reachable        23.0.6
```

三台都升級好後，我們來建立 Nginx 的 service，並且透過 `docker service ps` 的指令，可以看到 `Swarm` 確實把每一個 Task 都分配到不同的節點上，不像畚箕印為沒有地方可以分配，而全部塞在一起：

```shell
# 先建立 Nginx 的 service ， 並有三個副本
ssh root@157.245.51.137  

root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker service create --publish 8080:80 --detach --name nginx --replicas 3 nginx
qbzquv4vsimi3575kf37g5frw


# 用 ps 指令看分配狀況
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker service ps nginx

ID             NAME      IMAGE          NODE                                DESIRED STATE   CURRENT STATE           ERROR     PORTS
j0gubq48z3yz   nginx.1   nginx:latest   ubuntu-s-1vcpu-512mb-10gb-sgp1-01   Running         Running 2 minutes ago             
ot04hax1imy1   nginx.2   nginx:latest   ubuntu-s-1vcpu-512mb-10gb-sgp1-02   Running         Running 2 minutes ago             
i33hteygoipd   nginx.3   nginx:latest   ubuntu-s-1vcpu-512mb-10gb-sgp1-03   Running         Running 2 minutes ago    
```

可以發現每一台container，都各自分配到一台node上！

如果在這台 Leader 上輸入 `docker container list`，會發現這node上只會有一個 container：

```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker container list
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS     NAMES
d8f3673d1003   nginx:latest   "/docker-entrypoint.…"   3 minutes ago   Up 3 minutes   80/tcp    nginx.1.j0gubq48z3yzrn95n97qdaubd
```

透過 Digital Ocaen 的IP位置，可以直接輸入網址「http://157.245.51.137:8080/」，會看到熟悉的nginx畫面。




### 內建的 Load Balance

現在我們認知到每一個節點都有一個nginx容器，而Docker Swarm其實會自動幫我們做 Load Balance，如果單一節點流量過大的時候，會把請求分配到比較閒的節點，我們要怎麼驗證這件事呢？   

首先，可以透過 `docker service logs --follow` 指令來觀看整個 Service 的 Log，也就是三個容器一次滿足的意思：


```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker service logs --follow nginx

------
nginx.2.ot04hax1imy1@ubuntu-s-1vcpu-512mb-10gb-sgp1-02    | /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
nginx.2.ot04hax1imy1@ubuntu-s-1vcpu-512mb-10gb-sgp1-02    | /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
nginx.2.ot04hax1imy1@ubuntu-s-1vcpu-512mb-10gb-sgp1-02    | /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
nginx.2.ot04hax1imy1@ubuntu-s-1vcpu-512mb-10gb-sgp1-02    | 10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
nginx.2.ot04hax1imy1@ubuntu-s-1vcpu-512mb-10gb-sgp1-02    | 10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
nginx.2.ot04hax1imy1@ubuntu-s-1vcpu-512mb-10gb-sgp1-02    | /docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
...以下省略
```
可以看到最前面是nginx.1 這個容器接受到的請求，但我們回想一下，這個IP位置的容器是nginx.2才對，怎麼回應的會是別的伺服器的容器，可以看下圖，可以知道原來是 ingress 這虛擬網路做的事：


```md
> ingress network 示意圖

>              192.168.99.100:8080                   192.168.99.101:8080                   192.168.99.102:8080       
>              my-web published port                 my-web published port                 my-web published port     
>                      |                                     |                                     |                     
>   | - - - - - - - - -|- - - - - - - - | | - - - - - - - - -|- - - - - - - - | | - - - - - - - - -|- - - - - - - - |
>   |           | - - -|- - - |         | |           | - - -|- - - |         | |           | - - -|- - - |         | 
>   |           |  Swarm      |         | |           |  Swarm      |         | |           |  Swarm      |         | 
>   |           |  load       |         | |           |  load       |         | |           |  load       |         | 
>   |           |  balancer   |         | |           |  balancer   |         | |           |  balancer   |         | 
>   |           | - - - - - - |         | |           | - - - - - - |         | |           | - - - - - - |         | 
>   |                                   | |                                   | |                                   | 
>   |                                   | |                                   | |                                   | 
>   |                                   | |                                   | |                                   | 
>   | 10.0.0.1:80        node1          | | 10.0.0.2:80        node2          | |                    node3          | 
>   | my-web/1           192.168.99.100 | | my-web/2           192.168.99.101 | |                    192.168.99.102 | 
>   | - - - - - - - - - - - - - - - - - | | - - - - - - - - - - - - - - - - - | | - - - - - - - - - - - - - - - - - |
```


上圖顯示出，即使輸入的IP位置沒有容器的存在，也會透過 `Docker Searm` 的 `Load Balance`，把請求導過去在同個port有服務的節點。

而我們例子則是透過 Load Balance，即使輸入的IP位置是nginx.2容器的節點，他也不像是傳統的一對一這麼單純，而是會自動導流，可以開個 `docker service logs --follow nginx`，並且不斷重新整理網頁，會看到Log不斷地切斷到不同的容器來接受請求。   

至於 ingress network 是什麼呢？

### overlay 虛擬網路

我們前面已經學到一個 Leader 在不同的伺服器(節點)上建立容器，值得開心的是我們已經脫離了本機，是真正活在網路的世界中。

還記得我們前面在說道 Docker 虛擬網路的時候，有提到Docker 預設的 driver 是 bridge，而在Swarm模式預設的driver就是overlay，我們用 docker network list看看：
```shell
$ sssh root@157.245.51.137 

root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker network list
NETWORK ID     NAME              DRIVER    SCOPE
be7cdc65929a   bridge            bridge    local
04feba965c2a   docker_gwbridge   bridge    local
4fab372dc165   host              host      local
rzdy0jjoa87z   ingress           overlay   swarm      -> 這個 
8c38f4f11d4f   none              null      local
```

我們現在來用 Docker Swarm 配 PostgreSQL 建立一個服務，因為是跨越節點的溝通，所以就像 Compose 章節一樣，先建立一個虛擬網路，但這次是指定用 overlay 當作 driver：

```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker network create --driver overlay dev
zxnliir6n36nqi7xglkiiyc4j
```

接著我們先啟動「postgres:14-alpine」的服務，不需要開啟port對外，這在之前就有強調很多次，在同一個虛擬網路內，並不需要從外部進行連線，所以不需要 --publish：

```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker service create --name pg --network dev --env POSTGRES_PASSWORD=password postgres:14-alpine

------
sisd8tjsk25nnyhhuwbcs0ujz
overall progress: 1 out of 1 tasks 
1/1: running   [==================================================>] 
verify: Service converged 
```

接著再啟動「drupal:7.92」這個服務，這次就要使用 --publish，因為這次我們對網際網路世界的接口，讓我們來啟動吧：

```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker service create --name drupal --network dev --publish 80:80 drupal:7.92

------
frkir4hg5d9dzkn1i2qtwc41x
overall progress: 1 out of 1 tasks 
1/1: running   [==================================================>] 
verify: Service converged 
```
我們可以透過伺服器的IP位置進入到服務中，就到了熟悉的Drupal設定畫面，這裡選用比較迷你的drupal版本，是因為考量到我們在DigitalOcean紹開的機器很小，怕負擔不過來

首先我們到瀏覽器輸入「http://157.245.51.137:80」，就可以看到畫面上Dripal設定畫面。

到了設定資料庫的部分，在postgres映像檔的預設情況下，資料庫名稱及使用者名稱都是postgres。

至於進階設定的 Database Host，在 Docker Swarm 的模式下，是使用 service 的名字作為 overlay 虛擬網路中的 Database Host，所以這裡會填入「pg」及「port 5432」。

後面的設定就跟之前一樣，最終會來到 Drupal 的主畫面，這時若是把 service 刪除，將不會保留任何資料，因為我們沒有使用 volumes 來儲存資料，後面，我們會來教學在 Docker Swarm 的模式下儲存資料。

### 如何在 Swarm 中儲存資料

不過這邊要先說到，如果你今天問，volume是否能跨越伺服器的鴻溝，答案是「不行」，因為 volume 只能儲存在在固定的節點上，根據我們前面所述，Swarm會自動分配容器到隨機節點上，那如何確保PG資料庫分配給所有的volume節點呢？

Docker給出的解答是「contraint」，也就是透過在服務上貼標籤的方式，限制該服務只能執行在某個節點，自然而然，容器也就只會在跟節點執行，也算是解決的volume的問題。

另外一個解決方案是，使用外部服務，例如：「AWS」和「GCP」都有提供資料庫的服務，很多時候正式環境都會採用這些現成的資料庫服務，因為這些服務都提供備份及監測的功能，又能確實解決Swarm下的資料分布問題，也算是另外一種解法。  

下面示範如何貼標籤：

首先我們移除前面建立的 service：
```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker service rm drupal pg
drupal
pg
```

並列出所有節點：
```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker node list
ID                            HOSTNAME                            STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
fxtjyolpe6vsitbr5zmi0lzeu *   ubuntu-s-1vcpu-512mb-10gb-sgp1-01   Ready     Active         Leader           23.0.6
ryxvpsy0fdcgwkq7b2qwjjyiq     ubuntu-s-1vcpu-512mb-10gb-sgp1-02   Ready     Active         Reachable        23.0.6
h10hi4yxdtp06wj0l4sy97088     ubuntu-s-1vcpu-512mb-10gb-sgp1-03   Ready     Active         Reachable        23.0.6
```

接著在目前的節點(有*這個)，建立一個volume，以面postgres的服務能夠有儲存空間：
```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker volume create pg
pg
```
啟動PG的service，並且利用constraint來強致其執行在有volume的節點：

```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker service create --name pg --mount source=pg,destination=/var/lib/postgresql/data --constraint node.id==fxtjyolpe6vsitbr5zmi0lzeu --network dev --env POSTGRES_PASSWORD=password postgres:14-alpine

gqte2oo5wfz6z5yz28tcj81tp
overall progress: 1 out of 1 tasks 
1/1: running   [==================================================>] 
verify: Service converged 
```

在swarm模式下，service的volume掛載參數只能用 --mount 的方式，並且要標明 source 及 target，source就是volume，而target則為容器內的檔案系統儲存位置。


```md
> --mount source=pg,destination=/var/lib/postgresql/data 
```

constraint的使用方式則可以分成很多種，這裡用最簡單的方式，告訴swarm只會把這個service部署到ID為「fxtjyolpe6vsitbr5zmi0lzeu」的節點，也就是我們目前所在的節點：
```md
> --constraint node.id==fxtjyolpe6vsitbr5zmi0lzeu
```

接著用最傳統的方式來確認postgres服務是否有執行在這個節點上：
```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker container list
CONTAINER ID   IMAGE                COMMAND                  CREATED          STATUS          PORTS      NAMES
7faf95ca19a7   postgres:14-alpine   "docker-entrypoint.s…"   28 minutes ago   Up 28 minutes   5432/tcp   pg.1.ove5v9pdjdg3r826j0ue7zamk
```

再來，部署durpal服務：

```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker service create --name drupal --network dev --publish 80:80 drupal:7.92

zlihzn4zyih2q1qbhclu9wlfm
overall progress: 1 out of 1 tasks 
1/1: running   [==================================================>] 
verify: Service converged 

```

這樣新增好後，就可以照著前面有做過的步驟，打開瀏覽器「http://157.245.51.137:80」，設定好drupal，並且新增一篇文章。   

再來刪除postgres服務，並重新執行，確認其有連接到volume，而文章也沒有因為postgres的服務刪除而消失不見：
```shell
# 刪除 service
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker service rm pg
pg

# 重新打開，確認文章是否不見
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker service create --name pg --mount source=pg,destination=/var/lib/postgresql/data --constraint node.fxtjyolpe6vsitbr5zmi0lzeu --network dev --env POSTGRES_PASSWORD=password postgres:14-alpine
Error response from daemon: rpc error: code = Unknown desc = constraint expected one operator from ==, !=
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker service create --name pg --mount source=pg,destination=/var/lib/postgresql/data --constraint node.id==fxtjyolpe6vsitbr5zmi0lzeu --network dev --env POSTGRES_PASSWORD=password postgres:14-alpine
yanf5s91uaajq1f7z3bpqpg3w
overall progress: 1 out of 1 tasks 
1/1: running   [==================================================>] 
verify: Service converged 
```


此時重新打開網站，會發現文章還在，證明用貼標籤的方法，可以讓資料被保存！

### 如何在 Swarm 中傳遞敏感資料

前面已經說明了如何在 Swarm 模式中儲存資料，不知道大家有沒有想過，要如何把敏感的資料(ex. 資料庫密碼)在Swarm模式中傳遞呢？畢竟是跨機器的，直接把敏感資訊在service啟動時寫入，也會因為存在bash的歷史紀錄中，不太安全，這時Docker 給出了「sercet」、「config」兩種解決方案，如同名字一樣，一個是拿來儲存機密資料，一個是設定檔。

#### 使用 sercet 物件

首先，一樣先移除掉前面做示範用的 drupal 和 postgres 服務：
```shell
$ docker service rm pg drupal
```

接著先用sercet做示範，在機器上隨意建立一個檔案，假裝裡面放著敏感的資訊：
```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# echo "I'm secret." >> secret.txt
```

建立一個secret物件：
```shell
# 建立 secret 物件
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker secret create my_secret ./secret.txt
m0g8oswy189u3yiv5pkt5v42r

# 把 secret 列出來
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker secret list
ID                          NAME        DRIVER    CREATED          UPDATED
m0g8oswy189u3yiv5pkt5v42r   my_secret             28 seconds ago   28 seconds ago
```

接著建立三個副本的nginx服務，並將剛剛創造的secret注入服務裡面，讓三個節點中都能夠拿到這個資訊：
```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker service create --name nginx --publish 80:80 --replicas 3 --secret my_secret nginx
yyagfolkl8hmgx54j6wylvuz2
overall progress: 3 out of 3 tasks 
1/3: running   [==================================================>] 
2/3: running   [==================================================>] 
3/3: running   [==================================================>] 
verify: Service converged 
```

再來就是確認是否三個節點都可以看到這一份檔案了，先拿到當前node的ID：

```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker container list
CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS              PORTS     NAMES
968b9a8d5e42   nginx:latest   "/docker-entrypoint.…"   About a minute ago   Up About a minute   80/tcp    nginx.3.w4hbdc8epoq2mv04vb6f39rb0
```

接著用 `exec` 查看

```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker container exec 968b9a8d5e42 cat /run/secrets/my_secret
I'm sercret
```

可以看到我們可以透過 `cat /run/secrets/my_secret` 指令，拿到檔案中的資料，所有的 `secret` 都會 `預設` 被注入到這個路徑。



#### 使用 config 物件

相較於 secret 物件， config 物件比較偏向非敏感資訊，可以拿來存放某些服務的設定檔。   
   
一樣先刪掉前面的 nginx 服務，改用 config 物件來把資料注入容器內：

```shell
# 刪除 service
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker service rm nginx
```

用剛剛同一個檔案來建立 config 物件，使用的方式和 secret 一樣：

```shell
# 用剛剛檔案創建 config物件

root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker config create my_config ./secret.txt
wx771zknkw23h8i9q1hgwjwzv
```


再來一樣建立三個副本，並用 --config 的方式把檔案傳入，這裡可以接輸入 config 物件的名稱，檔案就會出現在容器的根目錄內，並且一樣可以用cat來查看：

```shell
# 建立 service
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker service create --name nginx --publish 80:80 --replicas 3 --config my_config nginx
w721hq8d3hu1krrcz4zxuabru
overall progress: 3 out of 3 tasks 
1/3: running   [==================================================>] 
2/3: running   [==================================================>] 
3/3: running   [==================================================>] 
verify: Service converged 

# 查看檔案內資訊
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
d80dc6d0c9c6   nginx:latest   "/docker-entrypoint.…"   54 seconds ago   Up 52 seconds   80/tcp    nginx.1.p5kzsgkj1tor8sz9lmjqeqje7

root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker container exec d80dc6d0c9c6 cat ./my_config
I'm sercret.
```

這邊可以確定./my_config 路徑有我們剛剛放進去的檔案，但如果今天不想用這個預設路徑呢？我們來試試自創路徑：


一樣先移除 nginx 服務，再把 config 物件放到想要的路徑：

```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker service rm nginx
nginx
```
   
   
再次創造三個副本，並且這次要更改路徑：
```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker service create --name nginx --publish 80:80 --replicas 3 --config source=my_config,target=/custom-dir/my_config nginx
oiwa9pvjvyykt1r5pdaoru1q8
overall progress: 3 out of 3 tasks 
1/3: running   [==================================================>] 
2/3: running   [==================================================>] 
3/3: running   [==================================================>] 
verify: Service converged 
```

用 `cat` 確認剛剛新創的路徑是否正確：

```shell
# 先看當前容器
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker container list
CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS              PORTS     NAMES
0724dc170bb1   nginx:latest   "/docker-entrypoint.…"   About a minute ago   Up About a minute   80/tcp    nginx.1.yrsxs7yft0bmxhtzcmkgzevht

# 輸入剛剛新的路徑
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker container exec 0724dc170bb1 cat /custom-dir/my_config
I'm sercret.
root@ubuntu-s-1vcp
```

我們透過「souce=config 物件的名字，target= 容器內的路徑」，就可以把config物件的檔案放到自己想要的位置。  

這時可能會想說，使用secret道理能理解，那config呢？用config好處在於，如果今天要更新設定時，用「Bind Mount」方式的話，需要暫停服務並重啟，用config不用，讓我來試試，在不停止nginx服務的情況下更新config，我們來更新剛剛的檔案試試：

先更新原本的設定檔，並再次建立一個新的 config，之後要拿來覆蓋掉舊的設定：

```shell
# 先新增資訊在 secret.txt 檔案
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# echo "New Config" >> secret.txt

# 新增 config 檔
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker config create my_config_2 ./secret.txt
av8hkutneo6x6yhkv1zqol8nr
```


接著用這個指令 `docker service update --config-add` 的方式來更新 nginx 的服務，並且覆蓋掉原本的設定檔：






再來用cat看一下該檔案資訊：
```shell
# 覆蓋原本的檔案
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker service update --config-add source=my_config_2,target=/custom-dir/my_config nginx

# 再印出一次
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker container exec 7ff041677cf9 cat /custom-dir/my_config

I'm sercret.
New Config
```

這樣就可在不中斷服務的看況下更動設定擋案。

### 打包所有服務

用 docker-compose.yml的概念來撰寫 Swarm 的預期狀態檔案，許多的語法跟 compose 差不多，唯一不一樣就是加入 deploy 的參數。

我們來用前面教過的 drupal 搭配 PostgreSQL 試試寫成一個yaml檔案，並一鍵啟動服務：



```yml
version: '3.9'

services:
  drupal:
    image: drupal:7.92
    secrets:
      - source: my_secret
        target: /my_secret
    networks:
      - dev
    ports:
      - 80:80
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
  
  pg: 
    image: postgres:14-alpine
    configs:
      - source: my_config
        target: /my_config
    volumes:
      - pg:/var/lib/postgresql/data
    networks:
      - dev
    environment:
      - POSTGRES_PASSWORD=password
    deploy:
      replicas: 1
      restart_policy: 
        condition: on-failure
      placement:
        constraints: [node.id == 7ff041677cf9]

secrets:
  my_secret:
    external: true

configs:
  my_config:
    external: true

volumes:
  pg:
    external: true

networks:
  dev: 
    external: true
```

上面檔案的大部分參數都看過，這邊主要來介紹 `delpoy` 內的參數，deploy 內的參數只有在 Swarm 模式下有用，若是使用 `docker compose up` 去執行這份檔案，不僅會直接略過這些參數，還會因為 Docker compose 不支援 secret 和 config 物件而出錯。


`replicas` 是副本，前面也有提過，至於 `restrt_policy` 就是 Docker Compose 寫的 restart，只是在 Swarm 模式下比較囉唆，且要放在 deploy裡面。


### 如何一鍵啟動


在 swarm 模式下， docker compose up 就派不上用場了，要改成這個指令 `docker stack deploy`。


所以以我們照著下面步驟：

```shell
# 先進入第一個機器
$ ssh root@157.245.51.137 

# 建立一個 docker-compose.yml 檔案
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# touch docker-compose.yml

# 確認有建立檔案成功
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# ls

------這指令會顯示所有檔案
docker-compose.yml  get-docker.sh  secret.txt  sercret.txt  snap


# 用 nano editor 進入 檔案裡面，把我們剛剛上面寫的 yaml 檔案，寫上去， 用 control + x 可以儲存檔案並離開
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# nano docker-compose.yml

# 可以用 cat 指令確認是否有成功寫入資料
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# cat docker-compose.yml

------
version: '3.9'

services:
  drupal:
    image: drupal:7.92
    secrets:
      - source: my_secret
        target: /my_secret
    networks:
      - dev
    ports:
      - 80:80
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
  # ...以下略過
------

# 確認檔案有成功建立後，就可以輸入 docker stack deploy
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker stack deploy --compose-file docker-compose.yml dev
Creating service dev_drupal
Creating service dev_pg

```

--compose-file 為定義狀態的 YAML 檔案，不一定要叫做「docker-compose.yml」，而最後的參數則為這個 stack 命名，可以根據這個 stack 所做的事情來命名，可以看到 stack 最終的結果，還是建立了兩個 service。


到底什麼是 stack 呢？使用上很像 Docker-compose，就是把整個應用程式的預期狀態放入一個 stack 內，並且交由 Docker Swarm 全權處理，如下所示：


```md
> Stack 示意圖

>   |--------------------------------------------------------------|    
>   |                                                              |   
>   |    |- - - - - - -|            |- - - - - - -|                | -- |
>   |    |  service.1  |            |  secret     |                |    |
>   |    |- - - - - - -|            |- - - - - - -|                |    |
>   |                                                              |    |
>   |    |- - - - - - -|            |- - - - - - -|                |    | ====== stack
>   |    |  service.2  |            |  config     |                |    |
>   |    |- - - - - - -|            |- - - - - - -|                |    |
>   |                                                              |    |
>   |    |- - - - - - -|            |- - - - - - -|                |    |
>   |    |  service.3  |            |  network    |                |    |
>   |    |- - - - - - -|            |- - - - - - -|                | -- |
>   |                                                              |
>   |                               |- - - - - - -|                |
>   |                               |  volumes    |                |
>   |                               |- - - - - - -|                |
>   |                                                              |
>   |--------------------------------------------------------------|

```




### 查看 Stack 內的服務狀態

透過 `docker stack ps 「stack 名稱」` 的方式，能看到整個 stack 內的容器狀態，包含了執行在哪個節點上以及隸屬於哪個 service 的詳細資訊：

```shell
ID             NAME           IMAGE                NODE                                DESIRED STATE   CURRENT STATE            ERROR                     PORTS
dzpnk6pmaoe2   dev_drupal.1   drupal:7.92          ubuntu-s-1vcpu-512mb-10gb-sgp1-01   Running         Running 19 minutes ago                                      
rf1bm7nv705t   dev_pg.1       postgres:14-alpine                                       Running         Pending 20 minutes ago   "no suitable node (scheduling …" 
```

### 一鍵移除所有服務


只需要使用 `docker stack rm` 指令就能夠做到，刪除一個 stack，也代表刪除裡面包含的服務，是不是很像 `docker compose down` 呢？

```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker stack rm dev
Removing service dev_drupal
Removing service dev_pg
```

部署 WEB 應用程式
------


### 利用 Traefik 部署自己的映像檔儲存庫




```yml
# docker-compose.yml
version: '3.9'

x-networks: &network
  networks:
    - registry

x-restart: &restart-always
  restart: always

services:
  proxy: 
    image: traefik:v2.8
    container_name: traefik
    <<: [*network, *restart-always]
    ports:
      - 80:80
      - 443:443
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./acme.json:/acme.json:rw

    command: 
      - --entrypoints.web.address=:80
      - --entrypoints.websecure.address=:443
      - --entrypoints.web.http.redirections.entrypoint.scheme=https
      - --providers.docker=true
      - --providers.docker.exposedbydefault=false
      - --certificatesresolvers.letencrypt=true
      - --certificatesresolvers.letencrypt.acme.httpchallenge=true
      - --certificatesresolvers.letencrypt.acme.email=a034506618@gmail.com
      - --certificatesresolvers.letencrypt.acme.storage=acme.json
      - --certificatesresolvers.letencrypt.acme.httpchallenge.entrypoint=web
  
networks:
  registry:
    external: true
```

:::tip
設定多的預設變數，要用陣列包起來，要不然會發生錯誤喔   
`<<: [*network, *restart-always]`  
:::

這裡蠻多參數都說過了，提幾個比較有趣的。

#### volumes
(1) volumes 參數中：「/var/run/docker.sock:/var/run/docker.sock:ro」 代表的是 Traefik 也需要監聽 docker.sock 這個 Unix Socket 上的事件，來掌握同一個虛擬網路中是否有容器被建立或移除，最後面的「:ro」則代表了 read only，也就是把伺服器的 docker.sock 交給 Traefik 去監聽，但他只能讀取資訊，而不能修改內容。

(2) volumes 參數中：「./acme.json:/acme.json:rw」 則是 Traefik 一個非常厲害的功能，他會替你自動申請 SSL 的憑證，也就是在上網時安全的網站旁邊都會有一個小鎖頭，這個 acme.json 檔案需要自己手動建立，並將其權限設定為「600」，來讓 Traefik 能夠寫入憑證資訊，結尾的「:rw」則是「read & write 都可以」的意思。


#### 建立 acme.json 檔案

```shell
# 先進入第一個機器
$ ssh root@157.245.51.137 

# 建立一個 docker-compose.yml 檔案
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# touch acme.json

# 設定權限
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# chmod 600 acme.json
```



### Traefik 指令解釋


1. 建立一個 entrypoints 叫做 「web」，並且給予其 port 為 80：
```md
> --entrypoints.web.address=:80
```

2. 建立一個 entrypoint 叫做 「websecure」，並且給予其 port 為 443：

```md
> --entrypoints.websecure.address=:443
```

3. 此行作用為將 http 協定自動轉至 https，也就是從80轉到443：
```md
> --entrypoints.web.http.redirections.entrypoint.scheme=https
```

4. 因為 Traefik 提供的支援不是只有 Docker，故在此處我們需要讓他知道提供服務的平台是 Docker：

```md
> --providers.docker=true
```

5. 前面有提到，Traefik 會監聽 docker.sock 來檢查有沒有服務被建立，而這行的目的在於告訴 Traefik，有需要被 Traefik 接受的服務，我們會自己加入參數，而不需要 Traefik 自動追蹤，這麼做的好處是，有時我們並不需要所有的服務都被 Traefik 追蹤，我們只需要在被追蹤的服務加入「--traefik.enable=true」即可，後面有示範。


```md
> --providers.docker.exposedbydefault=false
```

6. Let's Encrypt 是一個提供免費 SSL憑證 的網站，這裡告訴我們 Traefik 我們要使用它：
```md
> --certificatesresolvers.letencrypt=true
```

7. Traefik 主要提供的三種不同的 SSL憑證申請，這裡告訴 Traefik 我們要走 httpchallenge 的憑證申請，至於不同的申請方式，大家可以到 Traefik 的官網查看：

```md
> --certificatesresolvers.letencrypt.acme.httpchallenge=true
```

8. 再來就是申請 SSL憑證 時需要附上的信箱，當憑證快到期時，就會發送 email通知你：
```md
> --certificatesresolvers.letencrypt.acme.email=a034506618@gmail.com
```

9. 這行告訴 Traefik，關於 Let's Encrypt 的憑證儲存檔案是 acme.json 檔案，而這個檔案我們前面已經建立好了，並且用 volume 的方式放到容器內：

```md
> --certificatesresolvers.letencrypt.acme.storage=acme.json
```

10. 這行告訴 Let's Encrypt 的 SSL憑證申請的入口是走「web」，也就是 port 80 的入口：
```md
> --certificatesresolvers.letencrypt.acme.httpchallenge.entrypoint=web
```

11. 這些都準備好的話，就可以來啟動 Traefik 服務了：

```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker compose up --detach
[+] Running 1/0
 ✔ Container traefik  Created 
```


最後我們再輸入這個主機的ip位置，也就是 「http://157.245.51.137/」，這樣我們就成功啟動網站了，不過現在看到的畫面應該是畫面上印出「404 page not found」，這代表的是 Traefik 其實有成功建立，只是目前沒有任何服務啟動，所以沒有任何內容回應。


12. 在 docker-compose.yml 的 service 內，繼續加入映像檔儲存庫的服務：


```yml
version: '3.9'

x-networks: &network
  networks:
    - registry

x-restart: &restart-always
  restart: always

services:
  proxy: 
    image: traefik:v2.8
    container_name: traefik
    <<: [*network, *restart-always]
    ports:
      - 80:80
      - 443:443
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./acme.json:/acme.json:rw

    command: 
      - --entrypoints.web.address=:80
      - --entrypoints.websecure.address=:443
      - --entrypoints.web.http.redirections.entrypoint.scheme=https
      - --providers.docker=true
      - --providers.docker.exposedbydefault=false
      - --certificatesresolvers.letencrypt=true
      - --certificatesresolvers.letencrypt.acme.httpchallenge=true
      - --certificatesresolvers.letencrypt.acme.email=a034506618@gmail.com
      - --certificatesresolvers.letencrypt.acme.storage=acme.json
      - --certificatesresolvers.letencrypt.acme.httpchallenge.entrypoint=web

  registry:
    image: registry:latest
    container_name: registry
    <<: [*network, *restart-always]
    volumes:
      - registry-data:/var/lib/registry
    labels:
      - traefik.enable=true
      - traefik.http.routers.registry-http.entrypoints=web
      - traefik.http.routers.registry-https.entrypoints=websecure
      - traefik.http.routers.registry-http.rule=Host(`commenthouse.com`)
      - traefik.http.routers.registry-https.rule=Host(`commenthouse.com`)
      - traefik.http.routers.registry-https.tls=true
      - traefik.http.routers.registry-https.tls.certresolvers=letencrypt
      - traefik.http.middlewares.https-only.redirectscheme.scheme=https
      - traefik.http.routers.registry-http.middlewares=https-only
      - traefik.http.routers.registry-https.service=registry
      - traefik.http.services.registry.loadbalancer.server.port=5000
      - traefik.docker.network=registry
      
networks:
  registry:
    external: true

volumes:
  registry-data:
    external: true
```


這次我們先專注在 registry 中的 labels 就好。

13. 這行指令是告訴 Traefik 這個容器是需要被追蹤的，前面介紹 Traefik 設定時提過：

```md
> - traefik.enable=true
```

14. 下面兩行指令中的「registry-http」 及 「registry-https」 是可以替換的命名，主要的用途是告訴 Traefik 這個服務的 「registry-http」 是走 「web」 這個 entrypoints，也就是走 port 80，而「registry-https」 則是走 「websecure」 這個 entrypoints，也就是 port 443


```md
> - traefik.http.routers.registry-http.entrypoints=web
> - traefik.http.routers.registry-https.entrypoints=websecure
```

15. 下面兩行則是延續上一段，告訴 Traefik 關於「registry-http」及 「registry-https」這兩個 router 的規則，後方可以填入自己購買的網域，也可以使用 subdomain，像是「registry-core.qqaazz.online」這樣的網址也可以

```md
> - traefik.http.routers.registry-http.rule=Host(`commenthouse.com`)
> - traefik.http.routers.registry-https.rule=Host(`commenthouse.com`)
```

16. 但我們需要先到管理 DNS 的網站，設定網域的 A Record 指向目前在部署的這台機器，以 Cloudflare為例：

因此我們到 `cloudflare` 把 DNS紀錄的a類型，改成指向 digialocaen 伺服器的 IP位置(157.245.51.137)，原本cloudflare是指向 `34.102.136.180`。


17. 下面是告訴 Traefik 我們有一個 middleware 叫做 「https-only」，並且我們把它掛在「registry-http」這個reouter前面，意思是「當我們今天通過port 80走 http協定時，會強制幫我們轉到https協定」，這是屬於 traefik 內建的 middleware，還有很多有趣的功能，大家可去玩玩看：

```md
> - traefik.http.middlewares.https-only.redirectscheme.scheme=https
> - traefik.http.routers.registry-http.middlewares=https-only
```

18. 下面兩行指令的意思是代表「registry-https」這個router因為是port 443，預設是執行https協定，所以需要ssl的憑證，這裡所用的憑證頒發則是一開始在 Traefik 就設定好的 letencrypt：

```md
> - traefik.http.routers.registry-https.tls=true
> - traefik.http.routers.registry-https.tls.certresolvers=letencrypt
```


19. 以下第一行中，我們替「registry-https」這個 router 命名了一個叫做「registry 的 serivce，第二行則是告訴 Traefik registry 服務的 port 開在 5000，最後一行則是告知 Traefik Docker 的虛擬網路名稱

```md
> - traefik.http.routers.registry-https.service=registry
> - traefik.http.services.registry.loadbalancer.server.port=5000
> - traefik.docker.network=registry
```


20. 我們再次輸入 dokcer compose up --detach 指令， Docker 就會自動在執行映像檔儲存庫的服務：
```shell
root@ubuntu-s-1vcpu-512mb-10gb-sgp1-01:~# docker compose up --detach
[+] Running 2/2
 ✔ Container registry  Started                                                    2.5s 
 ✔ Container traefik   Started                                                    2.5s 
```

這樣當我們連到ip位置時，就會自動觸發 Traefik





