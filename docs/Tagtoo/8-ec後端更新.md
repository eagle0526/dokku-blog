---
title: tagtoo/ec
sidebar_label: "7. [tagtoo] ec"
description: ec 是關於爬蟲後端更新的檔案
last_update:
  date: 2023-11-08
keywords:
  - Muffet
  - ec
tags:
  - Muffet
sidebar_position: 7
---


:::tip
如果今天客戶的 muffet 裡面的 insert 是使用後端更新，就要到 ec 這個 repo 來修改，資料庫吃的是這個 repo，最後輸出成 op 要用 feed   
https://github.com/Tagtoo/ec   
:::    






前端更新和後端更新的差異
------

前端更新也就是 muffet，是使用 JavaScript 來去爬取客戶的網站，而後端更新是使用 Python 爬取客戶的網站，最大的差別在後端更新(python)會主動去爬取客戶網站，而前端更新需要顧客使用網站時，才會爬取資料


### 如何判斷 前/後端 更新
判斷是前端更新還是後端更新是以下面這兩個來區別：

```js
// 前端更新
insertPixel(`//track.tagtoo.com.tw/up?t=u&${product.toQueryString()}`)

// 後端更新
const api_tagtoo = `//api.tagtoo.com.tw/v1/refresh_product_item?url=${encodeURIComponent(location.href)}`
insertPixel(api_tagtoo)
```

### muffet 和 前後端更新的關係

muffet 檔案是設定像素追蹤，而前後端更新是當我們會送到資料庫的東西，最後 feed 產出的東西就是從我們雲端資料庫存的東西整理出來的



## 本機開發 - 流程大綱

1. 準備虛擬環境
2. 拉下 ec repo
3. 安裝 ipython 和 進入 ipython 環境
4. 安裝相關套件 (bs4、requests)
5. import 檔案 - import ec.site
6. 導入你想要看的產品 - 使用 site.get_item_by_url("產品連結")


Ps. 這個專案要用 python2 來實作，用 python3 會失敗喔


## 本機開發 - 詳細步驟
### 步驟一：準備虛擬環境

為了避免污染電腦的環境，所以我們準備一個虛擬環境，等等安裝的套件都會裝在我們的虛擬環境裡面

1. 安裝 virtualenv
```shell
$ pip install virtualenv==16.7.10
```
Ps. 要指定 virtualenv 版本的原因是因為 太新的版本不支援 python2


2. 確認你的 python2 目前安裝在哪

要先確認位置的原因是因為我們等等要用

```shell
$ where python2

-------
/Users/yee0526/.pyenv/shims/python2
```

3. 產生虛擬環境，並且指定這個虛擬環境的 python版本 要用 python2


```shell
# 產生資料夾
$ mkdir python2-vir
$ cd python2-vir

# $ virtualenv -p '你python2安裝的位置' '產生資料夾的名稱'
$ virtualenv -p /Users/yee0526/.pyenv/shims/python2 myenv
```


4. 進入虛擬環境

* 對於 Linux/macOS 系統：
```shell
$ source myenv/bin/activate
```

* 對於 Windows 系統：
```shell
$ myenv\Scripts\activate
```

### 步驟二：拉下 repo

```shell
$ git clone git@github.com:Tagtoo/ec.git
```


### 步驟三：安裝 ipython 相關套件

1. 確認目前這個虛擬環境裡面有下載哪些東西

會發現我們的環境目前只有一些東西
```shell
$ pip list

------
Package    Version
---------- -------
pip        20.3.4
setuptools 44.1.1
wheel      0.37.1
```

2. 安裝 ipython

```shell
$ pip install ipython
```
我們來看一下有沒有成功安裝：

```shell
$ pip list

------
Package                            Version
---------------------------------- -----------
appnope                            0.1.3
backports.functools-lru-cache      1.6.6
backports.shutil-get-terminal-size 1.0.0
decorator                          4.4.2
enum34                             1.1.10
ipython                            5.10.0
ipython-genutils                   0.2.0
pathlib2                           2.3.7.post1
pexpect                            4.8.0
pickleshare                        0.7.5
pip                                20.3.4
prompt-toolkit                     1.0.18
ptyprocess                         0.7.0
Pygments                           2.5.2
scandir                            1.10.0
setuptools                         44.1.1
simplegeneric                      0.8.1
six                                1.16.0
traitlets                          4.3.3
typing                             3.10.0.0
wcwidth                            0.2.6
wheel                              0.37.1
```

會發現我們環境裡面突然多了一堆東西，可以確認我們有成功安裝

3. 進到資料夾裡面並啟動 ipython

```shell
$ cd ec/
$ ipython

Python 2.7.18 (default, Jun 16 2023, 16:18:26) 
Type "copyright", "credits" or "license" for more information.

IPython 5.10.0 -- An enhanced Interactive Python.
?         -> Introduction and overview of IPython's features.
%quickref -> Quick reference.
help      -> Python's own help system.
```


Ps. 這邊要提醒一件事，因為原本我的本機環境也有安裝 ipython，不過本機環境是預設 python3，所以導致虛擬環境裡面雖然我一開始有預設是python2，但是還是被預設成 python3，雖然不知道是不是這個原因，但是當我把全域的 ipython 刪掉後，再次啟動虛擬環境的 ipython，就正常吃到 python2 了。






### 步驟四：安裝其他套件


進到 ipython 後，我們來 import 我們的檔案吧！

Ps. 我們現在在剛剛拉下來的 repo 裡面喔
```python
# python2-vir/ec
import ec.site

# ...省略
ImportError: No module named requests
ModuleNotFoundError: No module named 'schema'
ImportError: No module named bs4

```

安裝剛剛遇到的所有沒安裝套件：

```shell
$ pip install requests
$ pip install schema
$ pip install bs4
```


### 步驟五：import 檔案


1. 引入變數

```python
# 這個成功引入後，會發現我們的環境裡面有存入一個 ec
In [1]: import ec.site

# 可以用這個指令看
In [6]: whos

Variable   Type      Data/Info
================================
ec         module    <module 'ec' from 'ec/__init__.pyc'>
```

2. 確認變數

```python
In [11]: from ec.site import site
In [12]: whos


Variable   Type       Data/Info
================================
ec         module     <module 'ec' from 'ec/__init__.pyc'>
site       EC_Site    <ec.ec_site.EC_Site object at 0x1183fd148>

```


### 步驟六：指定產品連結查看資訊

方法解釋：
```md
> site.get_item_by_url("產品連結")
```

正式輸入：
```python
In [13]: site
Out[13]: <ec.ec_site.EC_Site at 0x1183fd148>


In [14]: site.get_item_by_url("https://www.s3.com.tw/TC/PDContent.aspx?yano=S00014739")

/Users/yee0526/Desktop/muffet/good/myenv/lib/python2.7/site-packages/urllib3/connectionpool.py:1063: InsecureRequestWarning: Unverified HTTPS request is being made to host 'www.s3.com.tw'. Adding certificate verification is strongly advised. See: https://urllib3.readthedocs.io/en/1.26.x/advanced-usage.html#ssl-warnings
  InsecureRequestWarning,
Out[15]: 
{'advertiser_id': 163,
 'category_path': u'\u7f8e\u9ad4\u4fdd\u990a > \u8eab\u9ad4\u4fdd\u990a > \u8eab\u9ad4\u4e73\u6db2\uff5c\u4fdd\u6fd5\u971c\uff5c\u6309\u6469\u6cb9',
 'description': u'\u53bb\u9664\u7570\u5473\uff0c\u5ae9\u767d\u808c\u819a \u542b\u690d\u7269\u8403\u53d6\u6210\u5206\uff0c\u6eab\u548c\u4fdd\u6fd5',
 'expire': datetime.datetime(2023, 7, 22, 4, 14, 58, 557444),
 'extra': {'google_product_category': '', 'variants': []},
 'image_url': u'https://photo.s3.com.tw/yaImg/S00014739/S00014739-01.jpg?1686124785083',
 'keywords': u'None',
 'link': 'https://www.s3.com.tw/TC/PDContent.aspx?yano=S00014739',
 'live': True,
 'price': 135,
 'product_key': 's3:product:S00014739',
 'recommends': [],
 'store_price': 199,
 'title': u'\u6cf0\u570b ISME~\u7da0\u8336\u814b\u4e0b\u5ae9\u767d\u4e73(15g) - \u5c0f\u4e09\u7f8e\u65e5 | \u7f8e\u599d\u3001\u4fdd\u990a\u3001\u751f\u6d3b\u8cfc\u7269\u7db2'}
```


這樣我們就可以在本機觀看目前爬蟲的狀況了！！！！
