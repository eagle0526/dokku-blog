---
title: ROR/GIT(02)
sidebar_label: "DAY3. [ASTROCamp] Git-2"
description: GIT
last_update:
  date: 2022-10-05
keywords:
  - 5xRuby
  - Git
sidebar_position: 4
---


1、git restore - 修復檔案
----------

如果今天不小心把檔案或目錄刪掉了，可以使用這個指令

```markdown
> **修復檔案實作** 
>
> git status                 > 刪除檔案後，確認目前git狀態
> --------------
>    終端機資訊   
>    刪除：     hello.html
>    刪除：     index.html
>    刪除：     welcome.html
> ---------------
>
> git restore hello.html     > 救回指定檔案
> git restore .              > 全部檔案救回(盡量不要用此功能，以免把修訂狀態的檔案一起還原)
> 
> 另一個回覆檔案的指令  
> git checkout .            
>
```


2、git checkout - 切分支、救檔案
------
(1) 有救回檔案、切換分支兩個功用    
(2) 新版的git把checkout換成下面ㄋ兩個新指令         


### 2-1 切分支發生的事  

1、HEAD移動  
2、檔案回復到指定分支狀態  

:::tip
**先創造新分支、並創造兩個新的commit**  
如果此時我們切到新分支，HEAD會跟著新分支移動，commit也會跟在新分支上  
如果此時再切回舊分支，剛剛兩個新commit的檔案，就會不見    
:::


### 2-2 新版拆成兩個指令

:::tip
git restore  ->  救回檔案   
git switch   ->  切換分支    
:::





3、git blame - 查看檔案更改資訊
----------

如果今天遇到系統壞掉了，或者是突然多了一行程式碼？想查找這一行是誰寫的？可以使用這指令

```markdown
> git blame index.html                > 確認這個檔案的所有行數修改資訊
> --------------
> 終端機資訊   
> abb4f438 (Eddie Kao 2017-08-02 16:49:49 +0800  1) <!DOCTYPE html>
> abb4f438 (Eddie Kao 2017-08-02 16:49:49 +0800  2) <html>
> abb4f438 (Eddie Kao 2017-08-02 16:49:49 +0800  3)   <head>
> abb4f438 (Eddie Kao 2017-08-02 16:49:49 +0800  4)     <meta charset="utf-8">
> abb4f438 (Eddie Kao 2017-08-02 16:49:49 +0800  5)     <title>首頁</title>
> abb4f438 (Eddie Kao 2017-08-02 16:49:49 +0800  6)   </head>
> abb4f438 (Eddie Kao 2017-08-02 16:49:49 +0800  7)   <body>
> 657fce78 (Eddie Kao 2017-08-02 16:53:43 +0800  8)     <div class="container">
> 657fce78 (Eddie Kao 2017-08-02 16:53:43 +0800  9)     </div>
> abb4f438 (Eddie Kao 2017-08-02 16:49:49 +0800 10)   </body>
> abb4f438 (Eddie Kao 2017-08-02 16:49:49 +0800 11) </html>
> ---------------
>
>
```

### 3-1、列出特定行數

下面的指令，可以列出index檔案的1~3行  
```md
> git blame index.html -L 1,3    
```

:::tip
git help 指令 -> 可以找出目前git有的指令  
:::


4、git branch - 建立分支、切換分支、刪除分支
------

### 4-1、git branch 確認目前分支
```markdown
> git branch               > 確認目前分支所在
> --------------
> 終端機資訊   
> * main
> ---------------
```

### 4-2、git branch cat 開cat新分支
```md
> git branch cat           > 在 “HEAD"所在位置，開新分支  
> git branch               > 開分支後，再確認一次目前分支  
> --------------
> 終端機資訊
>   cat   
> * main
> ---------------
>
```

### 4-3、git branch -d cat 刪除cat分支
```md
> git branch -d cat       > 刪除指定分支
> ---------------
> 終端機資訊
> 已刪除分支 cat（曾為 e12d8ef）。  
> ---------------
>
```

:::tip
如果head指向main就可以說，現在正在main分支上  
:::


5、git merge 合併分支
------


### 5-1、先切到想要到的分支
```markdown
> git switch/checkout cat    > 移動目前分支(checkout是舊版用法，switch是新版)  
> ---------------  
> 終端機資訊  
> 切換到分支 'cat' 
> ---------------
```

### 5-2、選擇想要合併的分支
```md
> git merge main             > 合併分支(現在所在分支，合成到指定分支)，現在是cat分支，比 main 還新
> ---------------  
> 終端機資訊  
> 已經是最新的
> ---------------
```

### 5-3、反過來合併也可以(會快進到比較新的那個分支)
```md
> git switch main            > 切到main分支
> git merge cat              > mian分支 合併 cat分支
> ---------------  
> 終端機資訊  
> Fast-forward               > 原本main分支，‘快進’到cat分支進度
> file1.html | 1 +
> file2.html | 1 +
> ---------------
```


6、分支互換
------

如果今天想要交換兩個分支的標籤，可以像下面這樣做
```md
(1) git branch -> 確認目前分支
>   cat      <-> 目標變成main
> * main     <-> 目標變成cat
 
(2) git branch -m cat2 -> 更改目前分支名稱
git branch
>   cat   
> * cat2

(3) git switch cat -> 切換到原本的分支
git branch
> * cat   
>   cat2

(4) git branch -m main -> 把此分支改成想要的分支名稱
git branch
> * main   
>   cat2

(5) git switch cat2 -> 再切回剛剛的分支
git branch -m cat   -> 再把分支名改掉，就完成分支名稱交換了
git branch
>   main   
> * cat
```


7、把git小耳朵叫出來
------
```md
> git switch main         > 先切到舊分支
> git merge cat --no--ff  > 融合到新分支的時候，不要使用fast-forward功能
```



8、外星分支(第三個分支跟第二分支合併)
------

### 8-1、指令介紹
兩個額外的分支 dog & cat 各做出不同的檔案，今天想要讓這兩個分支合併，就使用下面這個  
Ps. 後面要 -m "merge cat" 的原因，是因為要給讓兩個分支合併後，要給他一個新的commit，以免其他人不知道這個兩個合併是要幹嘛    
```md  
> git merge cat -m "merge cat" 
```



### 8-2、實際操作
```markdown
> git switch cat                           > 先切到第二個cat分支
> git merge dog -m "cat merge to dog"      > 讓cat分支融合第三個dog分支
>                                            融合後產生新的點(包含融合commit)
```

### 8-3、A 合併 B 或 B 合併 A 差異在哪
差在兩個合併後新產出的更改檔案，最後新的commit是被合成的那個檔案

```md
>  ex. dog 分支有 dog1.html、dog2.html
>      cat 分支有 cat1.html、cat2.html
> 
>  此時，cat分支合併到dog分支，新的分支上的檔案是
>  dog1.html、dog2.html
> 
```



9、git 大哉問
------

### 9-1、已經合併的分支可以刪掉嗎

Ans.可以    
  
分支單純只是一個貼紙，兩個分支已經合併了，另外再把分支刪掉也是沒問題的，他就只是一張貼紙  
ps. 不用擔心檔案不見  


### 9-2、還沒合併的分支可以刪掉嗎

Ans. 可以，不過比較不容易回復  
  
如果末端的枝節沒有貼紙的話，到原本貼紙的地方的commit，都會不見，假設不小心刪掉了，在原本的地方把貼紙加回去就好  
  
```markdown
> 刪掉的時候，會給一串數字，只要用建立新分支的語法加上去，就可把原本刪掉的分支加回去  
> git branch new_cat b174a5a  
```
Ps. 如果忘記該數字或找不到數字 - 使用git reflog，git 會給你過往HEAD的移動紀錄


### 9-3、合併發生衝突(conflict)了，怎麼辦
如果合併的時候，同時改了同檔案、同行的code，就會產生衝突，不過只要修改後，再繼續add、commit就可完成更改  

```markdown 
> git merge A.html -m "B merge to A"
> --
> 終端機資訊
> 自動合併 index.html
> 衝突（內容）：合併衝突於 index.html
> 自動合併失敗，修正衝突然後提交修正的結果。
> --
> 
> 這樣之後vr code會跳出衝突檔案，只要把該檔案修好(衝突的地方改掉)
>  
> ----------------------
>  
> git status                   > 修改好後，確認目前檔案狀態
> 
> ---
> 終端機資訊
> 位於分支 member
> 您有尚未合併的路徑。
>（解決衝突並執行 "git commit"）
>（使用 "git merge --abort" 終止合併）
> 
> 要提交的變更：
>      新檔案：   payment.html
> 
> 未合併的路徑：
>  （使用 "git add <檔案>..." 標記解決方案）
>     雙方修改：   index.html 
> 
> ---
> 
> git add .                    > 全部檔案加進暫存區
> git commit -m "fix conflict" > 修改衝突檔案，並提交新的commit
>  
```


10、git rebase - 另一種合併分支的方式
------

此合併方式，可以消掉兩個外星分支合併產生的點(新commit)

### 10-1、實際操作
```markdown
> rebase = re(變換) + base(根基)
> 
> git branch cat                 > 切到cat分支
> git rebase dog                 > 讓cat分支，以dog分支當作根基融合
> 
> ---
> 終端機資訊
> 成功重定基底並更新 refs/heads/cat。
>
> ---
```


### 10-2、rebase 合併好處
(1)、看起來比較乾淨 

### 10-3、rebase 合併壞處
(1)、兩條支線被融合了，線全部融合成一條，歷史紀錄會不清楚   
(2)、兩個檔案發生衝突的時後，因為線融合在一起，有可能不知道原本錯誤的檔案在哪   




11、git reset - git如何回到上一步
------

對reset指令誤解，把reset的中文翻譯成become會比較好

### 11-1、實際操作
```markdown
> git reset sha-1 -- hard (某個指定的標籤) 
>  
> mixed > 工作目錄(untracked) - git reset 預設狀態
> soft  > 暫存區
> hard  > XX(整個刪掉)
> 
> git reset sha-1 --mixed   > 這樣會導致退到某sha-1前的資料，那些資料都丟到工作目錄
>                 --soft    >                                           暫存區
>                 --hard    >                                           全部刪掉
>
> 
> 如果不想打 sha-1 那一串數字，也可以打“分支”
> ex. git reset cat --hard (還原到cat分支所在的地方，其他檔案都刪掉)
```


### 11-2、^ 和 ~ 代表的意思

:::info
^ -> Caret => 倒退一步的意思  
~ -> Tilde => 倒退N步的意思   
  
git reset HEAD^    => 退回上一個狀態  
git reset HEAD^~10 => 退回前10個狀態  
:::


### 11-3、有些shell會遇到沒有git reset HEAD^語法 

```markdown
> git reset HEAD^ --hard
> ----
> 終端機資訊
> zsh: no matches found: HEAD^
> ---- 
> 
> 只要加上一個反斜線就可以解決
> git reset HEAD\^ --hard
>   
```



12、git reflog - 呼叫出head的移動軌跡表
------

此指令可以呼叫出過往所有HEAD的移動狀況
```markdown
> 
> git reflog                > head的移動軌跡表
> 
> 終端機資料
> ---
> d89ec2f (HEAD -> lion) HEAD@{0}: reset: moving to HEAD^
> 08516db HEAD@{1}: commit (merge): 234
> d89ec2f (HEAD -> lion) HEAD@{2}: commit: add lion
> b69eb62 (dog) HEAD@{3}: checkout: moving from dog to lion
> b69eb62 (dog) HEAD@{4}: checkout: moving from cat to dog
> 8690e0d (cat) HEAD@{5}: commit: add cat5
> 0e4af52 HEAD@{6}: commit: add cat4
> b69eb62 (dog) HEAD@{7}: reset: moving to HEAD^
> 053fb21 HEAD@{8}: reset: moving to HEAD^
> f140e21 HEAD@{9}: reset: moving to HEAD^
> ---
```

### 12-1 reset sha-1
```md
> git reset sha-1 --hard   > 這可以回到指定的狀態，最原始資料夾也可以
```




13、git 注意事項
------

### 13-1、融合分支要注意的事情

```markdown
> 如果今天是用rebase融合兩個分支，再使用git reset HEAD\^ --hard，會無法直接回復融合前狀態，
> 要使用git reflog 找尋融合前的狀態才行。
```

### 13-2、git reset ORIG_HEAD的強力回覆手段
特殊用法，如果今天有做危險操作(融合分支)，ORIG會幫你記錄**一個**訊息  
```md  
> git reset ORIG_HEAD --hard 

```


### 13-3、git線上練習網站

:::tip
git線上練習網站：[https://learngitbranching.js.org/?locale=zh_TW](https://learngitbranching.js.org/?locale=zh_TW)  
:::


### 13-4、刪除.git的指令
:::danger
rm -rf .git 可以把整個git歷史紀錄刪掉  
:::