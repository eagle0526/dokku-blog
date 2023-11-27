---
title: JavaScript/What Is Event-Loop
sidebar_label: "2. [JavaScript] What Is Event-Loop"
description: What Is Event-Loop
last_update:
  date: 2022-09-24
keywords:
  - JavaScript
  - Event-Loop
sidebar_position: 2
---



:::tip
**前情提要**
本篇文章要說解釋的是事件循環(event loop)是什麼？？？？
要說明此東西，要先知到什麼情況下，會遇到、觸發事件循環(event loop)
:::
  
  
  
我們這邊先來舉個例子，今天我想印出一個HI，並且過五秒後，印出There，最後再印一個JS  
```md
> code                         Console(印出順序)
> 
> console.log(`hi`)             Hi
> setTimeout(function(){        JS
>     console.log(`There`);     There
> }, 5000);
> 
> console.log(`JS`)
```
  
  
承接上面印出的東西，會發現明明理想中的答案是HI > There > JS，為啥實際印出來不太一樣，有人可能會疑問，
大概是setTimeout設定太久的關係，那我們把原本的5s，改成0s
```md
> code                         Console(印出順序)
> 
> console.log(`hi`)             Hi
> setTimeout(function(){        JS
>     console.log(`There`);     There
> }, 5000);
> 
> console.log(`JS`)
```

結果會發現，疑答案還是跟5s的時候一樣欸，到底為啥會這樣呢？  
這邊就是我們今天要介紹的，事件循環是什麼？他幫我們做了哪些事？  
本篇文章提到的重點會有以些幾點   
1. Stack是什麼
2. 單執行序(one thread)是什麼
3. 非同步回調(Async Callback)是什麼
4. webAPIs & queue
5. 事件循環是什麼




Stack 是什麼
------

首先我們要來介紹的，就是Stack拉，這個東西在瀏覽器是一個非常重要的東西
如果今天我們呼叫一段JS函式，就會被放到Stack裡面
  
```markdown
>             JS
>    heap             stack
>   口                btn()
>      口             name()
>    口               cool()
>       口            OK()
> 
> 左半部是記憶體分配     右半部是執行函數的地方
> memory allocation   execution context
```

- heap是記憶體，就是進行記憶體分配的地方
- call stack是一種資料結構，他能夠紀錄幾種情況下，我們在程式中的哪些位置，比如說有一段函式，我們呼叫/使用他，這一段函示會被存放到stack裡面，有很多組函示的話，就會一層一層照執行順序堆疊到stack上，當回傳完函示裡面的東西後，該函式就會從stack裡被移除(依照先進後出的順序)

:::danger
但是!  
如果今天你用setTimeout 或是 DOM 或是 HTTP 並不會在這  
Ps. 說不在這不太精確，等等會提到setTimeout所在位置  
:::


單執行序 one thread
------

再繼續提到下一個東西前，我們要先了解一下單執行序是蝦咪？  
單執行緒的也就是程式執行時，所跑的程式路徑（處理的東西）是連續順序下來的，必須前面的處理好，後面的才會執行到。  
Ps. JS就是單執行序的程式語言  
  
  
既然JS是單執行序，就會有一個問題，如果今天在一堆程式碼中，中間有一段要執行很久的程式，那不就全部阻塞在那邊了嗎？  
如果今天進入一個網站，他要讀一個檔案讀超級久，在該檔案沒讀完前，整個網站都不能動，那肯定是一個大災難  
  
#### 阻塞 blocking    
什麼情況會造成執行速度變慢 => 其實blocking沒有很明確的定義，只要知道會造成執行速度變慢，是因為有一段執行很慢的程式碼 

ex. 你執行了一段while很長的迴圈，while迴圈後面還有其他程式，不過那些程式都要等到while迴圈跑完，才會執行，這邊這個有點塞住的狀況，就是blocking  
  
  
Ps. 那要如何解決blocking呢？JS使用到了 **Async Callback** 方法，後面會提到  
  
  
:::tip
one thread == one call back == one thing at a time  
單執行序 = 一次只有一個call = 一次只做一件事  
:::




非同步回調 Async Callback
------

再來就提到JS怎麼解決這個狀況了，大家在使用瀏覽器的時候應該都有注意到  
瀏覽器是不會 blocking 的(要不然使用者應該會發瘋，每動一步都要等一下才能做下一步)  
  
ps. node也不會blocking  
  
  
#### 實際範例
這邊又可以在把一開始那段程式碼，再拿出來一次
```markdown
> code                         Console
> 
> console.log(`hi`)             Hi
> setTimeout(function(){        JS
>     console.log(`There`);     There
> }, 5000);
> 
> console.log(`JS`)
```

可以注意到，There雖然被寫在第二段，照剛剛blocking提到的狀況  
應該要 **印出Hi -> 過5s後，印出There -> 最後印出JS才對**，不過為啥這邊反而是There會是最後出現。  
  
原因就是**非同步回調**的關係，如果今天有一段程式要耗時的時間比較久  
瀏覽器會跟這一段程式說：你先繼續執行，不過在你還沒把結果跑出來的這段時間  
我先去執行下面的程式，等你回全跑完之後，我再來印出你的結果  
  
  
Ps. 這邊可能會有人很好奇，那這一段在stack上的出現順序是怎麼樣的，這個因為要在提到另外個東西的關係，後面會提到。

:::tip
JavaScript Runtime 一次只能做一件事，但是瀏覽器會提供我們其他的東西  
這些東西是我們無法取得的執行緒，我們只能呼叫他們  
:::



webAPIs & queue
------
接下來就要來說明，剛剛的setTimeOut，在瀏覽器上到底是怎麼運作的  
首先先記住這一張圖  
  
Ps. webAPI是瀏覽器額外提供的東西

```markdown
>             JS                      webAPIs
>    heap             stack   
>  |口                btn()  |     | DOM          |
>  |   口             name() | --> | AJAX(fetch)  |  
>  | 口               cool() |     | setTimeOut   |    
>  |     口           OK()   |     |              | 
>  --------------------------      ----------------
>             |
>         event loop
>             |
>  ------------------callback queue------------------------
>     
>     onClick、onLoad、onDone
>  ------------------callback queue------------------------
```

### 非同步回調在stack、webAPI、queue、console.log上實際發生的狀況
我們現在再來跑一次剛剛的setTimeout

```markdown
> code                         Console
> 
> console.log(`hi`)             Hi
> setTimeout(function(){        JS
>     console.log(`There`);     There
> }, 5000);
> 
> console.log(`JS`)


> **文字註解**
> 1.當setTimeout被執行時，一開始會在stack，但是由於他要執行一段時間，就會轉移到webAPI 
>
>     stack            webAPIs
>    
>  |             |     |              |         
>  |             | --> |              |  
>  |             |     |              |    
>  |  setTimeout |     |              | 
>  ---------------     ----------------
>      |
>  event loop
>      |
>  -------callback queue---------
>     
>     
>  -------callback queue---------

> =========================================

> 2.此時stack中其他程式會繼續執行，而setTimeOut一樣在webAPI中繼續跑
> 而當他跑完時，不會直接放到stack(這樣會導致setTimeout隨機出現在程式碼中)
> 而是會跑到另一個地方 queue
>
>     stack            webAPIs
>    
>  |             |     |              |         
>  |             | --> |              |  
>  |             |     | setTimeOut   |  > 在webAPI持續執行中
>  |             |     |              | 
>  ---------------     ----------------
>      |
>  event loop
>      |
>  -------callback queue---------
>     
>     
>  -------callback queue---------

> ========================================

> 3.這裡就是任務佇列、回調佇列發揮作用的地方，所有在webAPI跑完的函式，
> 都會自動送到queue中排隊
>
>     stack            webAPIs
>    
>  |             |     |              |         
>  |             | --> |              |  
>  |             |     |              |  
>  |             |     |              | 
>  ---------------     ----------------
>      |
>  event loop
>      |
>  -------callback queue---------     
>     setTimeOut                   -> 在 queue 中等待執行如果stack是空的，就會
>                                     把 queue 的第一個任務，丟回
>  -------callback queue---------     


> ========================================


> 4.EventLoop執行
>
>     stack            webAPIs
>    
>  |             |     |              |         
>  |             | --> |              |  
>  |             |     |              |  
>  | setTimeOut  |     |              | 
>  ---------------     ----------------
>      |
>  event loop -> 如果stack是空的，就會把 queue 的第一個任務，丟回stack
>      |
>  -------callback queue---------     
>     
>  -------callback queue---------     
```
  
  
看完上面整個圖解流程後，應該可以了解seTimeOut在瀏覽器執行的流程，有了這個流程，很好了解決JS單執行序的問題  
並且還多了一些很有趣的應用(ex. GoogleMap的實際應用)
  
  
事件循環到底是什麼
------
最後，我們回來一開始的問題，事件循環到底是什麼呢？？  
  
他就像是上面這整個方程式中，最單純的小拼圖，他只有一個非常簡單的小工作  
他的任務是查看 stack、queue，如果 stack 是空的，他就會將queue的佇列中，第一個東西放回 stack上，讓堆疊能執行  

:::tip
**如果今天有其他的AJAX請求，都會以wevAPI這套相同方式去運作**  
ex. 今天我想要用爬蟲去爬取別人的資料、讀取別給我的的JSON檔案  
這些程式執行時都會先轉移到webAPI，等到stack清空時再執行  
:::


