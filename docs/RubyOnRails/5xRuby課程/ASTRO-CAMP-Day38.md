---
title: ROR/GIT(04)
sidebar_label: "DAY38. [ROR] Git-4"
description: GIT
last_update:
  date: 2024-06-18
keywords:
  - 5xRuby
  - Git
sidebar_position: 38
---


1、實際應用 GIT
------


### 情境一 - 新增 branch 前，忘記拉下 master 最新檔案

* 如果今天有一個自己的分支，先取名叫做 - `3471` 好了，因此我們先執行以下指令：

```shell
$ git checkout -b 3471
```

* 接著寫完一大堆 code 後，你想到自己在創新分支前，忘記把最新的檔案拉下來，這時候我們先切到 master 分支：

```shell
$ git checkout master
```

* 然後再拉下最新版本：
```shell
$ git pull
```

* 再來切回自己的分支：
```shell
$ git checkout 3471
```

* 接著進行融合：
```shell
$ git rebase master
```
   
ps. 這邊可能會遇到有衝突的地方，需要解衝突   
     
   
* 最後解完衝突後，在把分支推上去就好了
```shell
$ git push origin 3471
```

### 情境二 - 分支推上雲端後，master 有別的分支已經 merge 進去，導致自己的雲端分支落後於 master

* 如果今天有一個自己的分支，先取名叫做 - `3471` 好了，因此我們先執行以下指令：

```shell
$ git checkout -b 3471
```

* 接著寫完一大堆 code 後，你想到自己在創新分支前，忘記把最新的檔案拉下來，這時候我們先切到 master 分支：

```shell
$ git checkout master
```

* 然後再拉下最新版本：
```shell
$ git pull
```

* 再來切回自己的分支：
```shell
$ git checkout 3471
```

* 接著進行融合：
```shell
$ git rebase master
```
Ps. 這邊跟前面都差不多！接下來錯誤會出現在推上雲端的時候


* 推上雲端，發生以下錯誤
```shell
$ git push origin 3471

-----
 ! [rejected]        3471 -> 3471 (non-fast-forward)
error: failed to push some refs to 'github.com:Tagtoo/product-service.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```
Ps. 簡單就是說，你的本地分支落後於遠端分支


* 確保本地分支和遠端分支都是最新的

```shell
$ git pull origin 3471 --rebase
```

Ps. 這個指令會將遠端分支的更新拉下來，同時合併進本地的修改，這邊通常會有衝突，解完衝突就好


* 最後推上去就可以

```shell
$ git push origin 3471
```
