---
title: ROR/GIT(03)
sidebar_label: "DAY35. [ROR] Git-3"
description: GIT
last_update:
  date: 2022-11-29
keywords:
  - 5xRuby
  - Git
sidebar_position: 36
---


1、ssh-keygen設定
------

設定GitHub的SSL

### 1-1、ssh-keygen指令
首先在終端機打上
```md
> ssh-keygen
```

:::tip
這樣打後，會產出一份public、private的key  
通常放的路徑是在這/home/aaaaaa/.ssh/id_rsa    
如果想看到實際資料夾的位置，可以在mac搜尋bar打 **~/**     
這樣打後選擇進自己使用者的資料夾位置，然後打  
command + shift + . ，這樣就可以看到/ssh資料夾    
:::

:::tip
如果不會用終端機進到.ssh的位子，可以照著下面做         
(1) 開啟 Finder        
(2) 對著"下載項目"按右鍵   
(3) 點選"顯示上層資料夾"   
(4) 最後鍵盤同時按下，shift + command + .  
(5) 這樣就可以成功看到.ssh的資料夾了   
:::


### 1-2、查看金鑰
那要怎麼看自己金鑰呢？只要切到使用者的資料夾，並輸入以下指令
```md
> cat id_rsa.pub -> 可以印出你的鑰匙
```


!!印出鑰匙後，可以把鑰匙放到GitHub上!!





2、Git push
------

接下來到github建立空的repository，並在終端機輸入以下的指令，就可以把本機的檔案推到github上      

:::tip
如果想要知道指令，可以打下面的指令    
```shell
$ git help branch   
```
:::


### 2-1、把本機檔案推上遠端
輸入以下指令，可以把本機檔案推上去
```md
> git remote add origin git@github.com:eagle0526/git3-test.git    # 在遠端加分支
> git branch -M (current_branch) main                             # 強制把目前的分支改成main分支
> git push -u origin main                           # 我想要把畚箕的main分支的進度，推一份origin上
```

### 2-2、查看遠端節點
到終端機下這一行，可以看到目前有的遠端節點，不過現在會發現啥都沒有(因為啥都還沒加上去)  
```md
> git remote -v
```

### 2-3、新增遠端節點分支
第一行的origin可以改，這一行的意思就是在遠端加上一個tiger分支    
```md
> git remote add tiger git@github.com:eagle0526/git3-test.git
```


接著我們在下這一行，這時候就會發現，遠端有一個tiger的節點
```md
> git remote -v     # 印出遠端節點
```

### 2-4、更改遠端節點
更改遠端節點，把目前的tiger節點，改成dragon節點    
```md
> git remote rename tiger dragon
```
### 2-5、刪除遠端節點
刪除遠端節點，把目前的dragon節點刪掉
```md
> git remote remove dragon
```

### 2-6、把本機分支推上遠端
最後下push指令，就可以把目前本機的main分支，推一份到遠端節點上面
```md
> git push -u origin main
```



### 2-7、git push upstream

:::tip
如果現在重新在檔案加上新的檔案，並git add . 、 git commit，此時直接推上去，會發生上游衝突，他會說起前的main沒有上游分支  
:::

因此我們這樣打，給main一個上游分支，像下面這樣打，把本機的分支跟遠端的分支做掛鉤，origin是遠端也就是上游，把main當作預設的本地分支    
```md
> git push -u origin main

> ps. 這個-u是upstream的意思
```

:::tip
設定好上游分支後，之後就可以只打git push，就都會預設幫你把main分支的東西一起推上去了   
:::

如果你之後想改預設的本機分支，可以這樣改
```md
> git push -u origin feature/abc
```
這樣的意思是，我要把本機的feature/abc設定為預設分支，並且推上去

:::tip
github禁止直接推到某個分支上    
branch protection rules   
:::





3、Git push 的進階用法
------

### 3-1、把本地分支推上去改成別的分支
今天如果想把本地的main推一份到遠端的origin分支，並形成一個main分支  

這是一個簡寫    
```md
> git push origin main
```

上面那行指令的完整型態  
```md
> git push origin main:main
```

如果知道完整型態的寫法的話，可以修改成下面這樣  
這行的意思是，把一份本地的main，推一份到遠端的origin，並在線上行程一個cat分支  
```md
> git push origin main:cat
```

### 3-2、Heroku範例
之後做專案會用到的Heroku，我們會下這段指令，推一份本地的main分支，到遠端的heroku上面，並建立一個main分支     
```md
> git push heroku main
```

你只要推上一個main分支上去，heroku就會自動幫你執行一大段東西  
如果你今天在做feature的member分支，你可以直接這樣打，這樣就可以直接讓heroku吃到main分支  
```md
> git push heroku feature/member:main
```

### 3-3、把遠端的cat分支刪掉
用這種方法把遠端分支刪掉，他的原理是    
:前面的是一個空分支，用空分支推到遠端的origin，並形成cat分支    
```md
> git push origin :cat
```


4、Git Pull vs. Git Fetch vs. Git Merge
------
剛剛講完了如何把本機檔案推上git，現在來說明一下如何把檔案從GitHub載下來  

### 4-1、git clone 下載檔案
首先隨便開一個資料夾，cd到裡面後，下指令
```md
> git clone git@github.com:eagle0526/git3-test.git
```

### 4-2、git remote -v 查看遠端分支
抓下來後，可以看一下遠端分支現在在哪    
會發現clone下來的時候，還會順便幫你遠端節點設定好    
```md
> gti remote -v
```



### 4-3、git fetch 線上抓下新檔案

如果今天線上進度比本機還新，要怎麼把檔案抓下來呢
```md
> 本機                github
> 
>                     口   git在這   
> 
> 口 本機進度在這       口
> 
> 口                  口
> 
> 口                  口
```

接下來用fetch指令，他會在本機抓線上最新的檔案回來，並且在本機形成一個遠端分支，origin/main為啥叫這個名字呢。因為遠端分支是origin，本機分支是main     

```md
> 本機                github
> 
> 口 本機要更新到這     口   git在這   
>    origin/main
> 口                  口
> 
> 口                  口
> 
> 口                  口
```


把遠端分支最新的檔案抓下來到本機
```md
> git fetch origin main
```

### 4-4、git merge 線上融合本機分支

把剛剛線上fetch下來的檔案，跟本機的進度merge    
已下指令就可以把本機的main跟origin分支融合    
```md
> git merge origin/main
```


### 4-5、git pull 拉下遠端分支、融合本地分支

pull的指令是一次做完fetch、merger在做的事
```md
> git pull = git fetch + git merge
```

:::tip
如果今天git pull有衝突，是git merge的衝突喔，因為純粹fetch拉下來，不會有錯誤  
錯誤是發生在東西合併
:::





5、多人協作
------

如果今天A先推一份上去到github，這時候B也想推上去，會造成B推不上去   
因為A已經推新的上去了，B需要先拉一份檔案下來，再推上去  

```md
> Ａ人              main分支            B人               
>                        
> Ｏ  --> 先推上去     Ｏ    <-- 推不上去  X
> 
> 口                  口   git原先在這   口
>    
> 口                  口                口
> 
> 口                  口                口
>  
> 口                  口                口
```


解決方法，B先拉一份下來，再推上去
```md
> Ａ人                github            B人               
>                       
>                    X + O  <--推上去   X + O  
> 
> O  --> 先推上去      O                X + O (先拉下來)
> 
> 口                  口   git原先在這   口
>    
> 口                  口                口
> 
> 口                  口                口
>  
> 口                  口                口
```


如果今天都在main分支上做，會讓main分支很混亂
所以通常會開其他分支，確定好後再推到main分支上    






6、PUll Request(PR) 
------

### 6-1、發PR的人
如果你今天對某個專案感興趣，並且想要給他修改建議，你可以先fork一份他的專案到你自己的GitHub上    
接著你就可以更改這個專案的內容了，改好之後，就可以點選Pull Request，然後寫下你這次的改動    


#### 6-2、收PR的人
假設原作同意收下這個pr，下面會有一個Merge pull request的按鈕可以按，按下去之後，檔案就會直接更動    

Ps. 原作者在收的時候，可以在右邊的部分，對這個PR評論，並且如果覺得他寫得太爛，可以直接退還給他

:::tip
之後團隊合作，都是用PR的方式來協作    
:::


### 6-3、Finish your review
看完PR後有三個選項可以選
1. comment - 單純評論   
2. approve - 認同這則改動(如果到一定數量的認同，就會開啟可以點擊receive的功能)  
3. request change - 寫太爛，請改過  



### 6-4、三種收pr的方式
假設原本的commit數量有2個
```md
> create a merge commit -> 會有一個新的節點(對方發的commit有兩個，收完之後變五個)
> rebase and remote     -> 不會有節點(對方發的commit有兩個，收完之後變四個)
> squash and merge      -> 全部搜集成一個commit(對方發的commit有兩個，收完之後變三個)
```

:::tip
如果PR衝突怎麼辦，主辦人退回給寫PR的人，叫他改過再來  
:::

:::danger
什麼時候會用到squash？就是發PR的人寫太多無用訊息  
:::



:::tip **Approve的效用**  
可以設定，每一個PR有多少人Approve，才可以merge進去專案裡面   
所以在做專案的時候，如果團隊有五個人，一定要設定approve是四個   
這樣代表大家都要同意這件事，才能讓專案推上去      
:::



7、Git世界觀 - 11/29 - 下午筆記還沒做
------
寶藏都在.git目錄裡面(git init會產生的檔案)

blob代表檔案
tree代表目錄
commit
tag



### 7-1、git的blob物件計算
```md
> #引用sha1
> require "digest/sha1"

> 寫一段內容
> content = "hello, 5xRuby"

> 計算公式
> input = "blob #{content.length}\0#{content}"

> 可以把blob後的結果印出來試試
> puts Digest::SHA1.hexdigest(input)
```


14:10

這個東西 -> "|" 可以把後面的指令，倒給前面
因此我們可以下這個指令，來印出index檔的sha1
```md
> git index.html | git hash-object --stdin

> 印出e90bdb3679f7cfd0278b8d1aa9e8626c46eedf21
```


上面那個結果，會被塞到.git檔案裡面的.objects資料夾
並且取e9做一個資料夾，後面其他值會變成裡面的檔案的名字


:::tip
在git的世界，空的資料夾是無法被版控的
裡面至少要有一個檔案，就算檔案是空的也沒關係      
:::



可以看某一串sha-1的型態
```md
> git cat-file cda3d2e -t
```


可以看該commit的詳細資訊
```md
> git cat-file cda3d2e -p
```


這個commit裡面有一個tree，叫做417e
Ps. 每一個commit底下都會有個tree


繼續往下追
```md
> git cat-file 417e -p
```
會發現裡面有一個tree、一個blob

第一次commit，會指向




### 7-2、第二次commit

先把某個檔案的內容改過(index.html)

當你git add、commit 後，他又生成了另外一個commit

叫做b7c7

```shell
$ git cat-file b7c7aad
```


會發現裡面有一個tree(1480)、parent(cda4 -> 指向前一個commit)

```shell
$ git cat-file 1480 -p
```


裡會有一個e6928，他就是剛剛的database的tree



:::tip
如果今天有兩個檔案內容一樣，標題不一樣，會被算同一個blob   
:::



### 7-4、第三次commit

產生出ba90的blob


git cat-file ba90

裡面有一個123c的commit、前一個的tree







14:46 git世界觀的動畫



:::tip
git設計概念   
DAG 有相無還圖 -> 有方向但是不會造成循環  
:::



### 7-5、切換分支的時候發生了什麼事

首先HEAD會移動，並且整個專案的資料都會回到當時檔案的狀態





:::tip
git不是在做差異備份   
git是整個一起備份  
:::



### 7-6、git完整備份的好處
雖然完整備份比較比較浪費空間，但是還原速度很快   
因為我是整個檔案拉下來，不用去比較差異   



### 7-7、既然git只記錄檔案內容，那怎麼知道他叫什麼名字？
blob物件決定檔案的內容   
tree物件決定檔案的樣子   

blob是在add時候會產生    
tree是在commit之後才會產生的     



### 7-8、git add指令可以把檔案加到暫存區，為什麼有些資料被說是加到索引
當我們git add的時候，會建立一個份資料到index那邊


Ps. 暫存區又稱為索引index區


### 7-9、關於分支

開一個新分支，會在res/head裡面，新增一個檔案
如果今天直接把檔案名改掉，分支的名稱就會變了
或者是你把分支裡面的sha1值換掉，那個分支會直接換到新改的sha1




如果今天下此指令，會在head那邊新增一個aaa的資料夾，裡面有bbb的檔案
```md
> git branch aaa/bbb
```


:::danger
如果今天先開一個feature的分支，然後再建立一個feature/aaa      
這樣會報錯喔，原因是因為檔案名相撞了  
:::




### 7-10、HEAD是什麼東西
內容 = 某個commit的sha1或者是某個分支的名稱



### 7-11、關於標籤
通常會打一個標籤，有可能是因為有一個里程碑
如果今天有新增tag，會出現在refs的tags裡面


:::tip
**輕量化標籤 vs 普通標籤**    
普通標籤是有註記的標籤，如果今天去cat-file這個標籤，會發現裡面有藏一個物件     
它的功用是可以讓這個標籤有更清楚的內容     
輕量化標籤就單純只有標籤上面的名字    
:::




### 7-12、修改commit歷史

用reset修改，讓最新的的
```md
> git reset HEAD \^ --soft
> git commit -m "add fish"
```

用"--amend"，可以對最後一個commit做修改
```md
> git  commit -m "add fish" --amend
```




我要從現在的所在地，指定的sha1，做互動式的修正
不過這個要到VIM介面修改
```md
> git rebase --i sha1

> 在終端機輸入，就會跳出VIM的介面
```




如果今天老闆要你改歷史紀錄，但是因為內容改掉了，整個sha1會重設，會導致其他人都推不上去   
所以如果真的要重改歷史紀錄，要記得通知其他協作人員，並且請他們重新clone這一份檔案    
```md
> git push -f           # 強推上github
```


### 7-13、revert
再多人協作的情況下，可以對某個commit，對他下revert指令   
這樣當其他人收到clone下來時，他不會拿到被revert的那個檔案    



### 7-14、Flow
多人協作一定會用到的流程

