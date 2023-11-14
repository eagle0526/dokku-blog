---
title: ROR/GIT(01)
sidebar_label: "DAY1. [ROR] Git-1"
description: GIT
last_update:
  date: 2022-10-03
keywords:
  - 5xRuby
  - Git
sidebar_position: 2
---


1、關於git - 版本控制系統
-------------

如果你問大部份正在使用 Git 這個工具的人「什麼是 Git」，他們大多可能會回答你「Git 是一種版本控制系統」   
專業一點的可能會回答你說「Git 是一種分散式版本的版本控制系統」  
但這個回答，對沒接觸過的新手來說，有講跟沒講差不多。到底什麼是「版本」？要「控制」什麼東西？什麼又是「分散式」？    
這個我們等等都會位大家解答  



### 1-1 什麼是版本

下面三個日期都有各自的資料夾，資料夾就代表三種版本

```markdown
>  
> 2016/2/8  檔案夾有三個檔案  
>   
> 2016/2/10 檔案夾有5個檔案(多兩個)  
>   
> 2016/5/8  更改五個檔案中其中的兩個檔案內容  
>  
```
  
*把每個版本以拍照比喻，每拍一張照，英文上叫做快照(snapshot)，也就是一個版本就是一個快照



### 1-2 什麼是分散


分散就是沒有經過一個主要伺服器，散落到每一台電腦裡面資料統整起來就會是完整檔案  
ps. CVS(Concurrent Version System)是一種集中式版控系統，使用這個，如果主要伺服器壞掉，就爆炸了



2、git簡介
-------

Git是目前業界最多人使用的版本控制系統之一，他就像時光機，可以讓檔案回到歷史紀錄的某一個時間點
  

### 2-1 git 優點    

- 同時支援本地及遠端操作(沒有網路也可以)    
- 容易與他人共同協做    


### 2-2 git 使用前的一些指令    

#### 2-2-1 檢視目前git的設定    
```markdown
> git config --list
```

#### 2-2-2 設定username 以及 email  
```md
> git config --global user.name "YeeChen"  
> git config --global user.email "a034506618@gmail.com"
```

#### 2-2-3 檔案初始化
初始化後會產生一個.git的目錄，這個就是整個git的精華
```md
> git init
```

#### 2-2-4 刪除某個指定檔案
這個只另可以山除某個指定檔案，不過這個指令有點危險，小心使用  
```md
> rm -rf .git               # 重複強制刪除.git這個檔案
>     r = recursive         # 遞迴重複的意思
>     f = force             # 強制的意思
```


:::danger 不小心在根目錄下指令 **git init**
如果今天不小心在整個電腦根目錄下了git init指令，造成整個電腦檔案都被板控         
解決辦法為 - 在根目錄的地方下 rm -rf .git        
可以刪掉在根目錄的 .git檔案      
:::


3、git commit 三個狀態
------

### 3-1 做專案的時候，git狀態的改變流程

(1) 產生檔案/修改檔案            
(2) 工作目錄    
(3) git add index.html  
(4) 暫存區域    
(5) git commit -m "add commit"  
(6) 儲存庫  



### 3-2 實際範例
這邊實際創立一個檔案，來看git的狀態改變 


#### 3-2-1 touch - 產生檔案、預設工作目錄
當你這個資料夾有再git的控制下，只要產生一個新檔案，這個檔案就會放一份在工作目錄裡面 
```md
> touch index.html
```

#### 3-2-2 git add - 把剛剛新增的檔案，加進暫存區域
```md
> git add index.html
```

#### 3-2-3 git commit - 把剛剛新增的檔案，加進儲存庫
```md
> git commit -m "first commit"
```



#### 3-2-4 完整資訊流程
```md
> 工作目錄(working directory)     | git add     -> 檔案再工作目錄要git add，才會往暫存區域移動
>  
> 暫存區域(staging area)          | git commit  -> 檔案再暫存區域要git commit，才會網儲存庫移動  
>
> 儲存庫(本地)  
>
> git add + git commit 完，算一個完整的commit(完整的拍一張照片)  -> 整個流程做完後，就是做一個"快照"
```

#### 3-2-5 git status - 確定目前git狀態
這個可以確認目前檔案是在三個狀態鐘的哪一個
```md
> git status
```

#### 3-2-6 git add .

::: tip
這個意思是把全部東西都放進暫存區域   
不過盡量不要使用此功能，除非很了解自己所有的檔案進度到哪      
:::


#### 3-2-7 git log - 看過往commit歷史紀錄
```markdown
> git log             看所有紀錄  
> git log --online    看最新歷史紀錄  
> git reflog          查看HEAD所有移動紀錄
```
