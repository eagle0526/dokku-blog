---
title: ROR/JavaScript(05)
sidebar_label: "DAY13. [ROR] JavaScript-5"
description: JavaScript
last_update:
  date: 2022-10-21
keywords:
  - 5xRuby
  - JavaScript
sidebar_position: 14
---



1、API(Application Programming Interface)
------

抓取網路資料或是拿取一些別人的程式套件，就是用API來取得

### 1-1、舉個例子

當A程式需要另一個B程式幫他做某件事或拿某些資料的時候，這中間就會透過API(我們稱C)來幫忙溝通。   
這時A並不需要知道B做了什麼，怎麼做的。他只需要知道三件事:      
(1) 向C要求做這件事之前需要提供什麼資料?       
(2) 成功的話C會回復給我什麼?      
(3) 失敗的話C會回復給我什麼?    

上述三點也就是我們說的API的規格，至於這些規格要從哪邊取得呢?    
通常都是跟開發API的工程端拿，或是大一點的網站，如Facebook、google，或是政府機關網站都會有對應的API規格文件可以參考。    



2、RESTFul
---- 
(1) REST = 表現層狀態轉換   
(2) 網頁資料取得的幾個動詞，這之後會在提到  

如果我的網址的CRUD都符合上述方法，就稱為 符合RESTFul API設計
```markdown
> GET
> POST
> PUT
> PATCH
> DELETE
```

### 2-1、PUT Vs. PATCH

假設今天要更新使用者的基本資料(有姓名、大頭貼、描述)    

#### 2-1-1、PUT
用put方法送出更新，不只使用者的姓名會更新，連帶的連其實沒有要更新的大頭照以及描述都會在更新一次   

#### 2-1-2、PATCH   
用patch方法送出更新，則僅會將使用者姓名更新 ，沒有動到的大頭照以及描述並不會也一起更新一次    



3、資料傳輸格式
------
### 3-1、XML 
是一種延伸標記語言(跟HTML一樣都是標記語言) - 裡面的標籤可以自己定義，不過也只有你自己看得懂

### 3-2、CSV 
逗號分隔值，所有資料被逗號分格(!==excel檔)

### 3-3、json (JavaScript Object Notation)
json - 他跟JS很像的關係，因為他是用JS格式寫出來的 (不過要記得，網路上拿到的東西不是物件或是字串的格式，**都是文字喔**)



4、AJAX(Asynchronous JavaScript And XML)
------
(1) AJAX不是一種技術或語法，而是一種術語    
(2) 什麼是非同步 - 假設今天你傳簡訊給同事，他過一陣子才會回你訊息，這就叫非同步     
(3) JS是一種 **單執行緒** 的程式語言 (一個廚房只有一個廚師)     


### 4-1、今天執行AJAX是什麼意思
前面正在執行JS，後面在送資料到你的電腦，也就是在畫面執行時，偷偷送資料進來  


### 4-2、AJAX應用

#### 4-2-1、google map
(1) 一開始只能載入一小部分的圖資，在你拖拉畫面的時候，他會一直持續load新圖資進來  
(2) 滑鼠移到景點名稱時，才跳出名稱，也是AJAX應用  

#### 4-2-3、Fetch/XHR
(1) 網路爬蟲
(2) 抓取別人的Json檔案



5、同步執行、單執行緒的問題
------

### 5-1、同步執行有什麼問題嗎
會比較慢，因為JS是單執行緒，今天載入一張圖片時，如果這張圖片很大，那你要等這張圖片載完，才能做其他事情


### 5-2、單執行緒
單執行緒的也就是程式執行時，所跑的程式路徑（處理的東西）是連續順序下來的，必須前面的處理好，後面的才會執行    



6、使用 XHR (XMLHttpRequest) 寫網路爬蟲
------
(1) 資料來源 - API Application Programming Interface    
(2) jsonplaceholder這個網站可以加入我的最愛(free fake API)，假的API網站，可以拿來練習   


### 6-1、forEach寫法
```markdown
> const api = "網址"
> const req = new XMLHttpRequest();
> 
> 
> req.addEventListener("Load", () => {
>     const posts = JSON.parse(req.responseText)
>     // 處理posts顯示(用json.parse處理這個文字檔)
> 
>     const lists = document.querySelector(".lists") -> 抓li物件
> 
>     posts.forEach((post) => {
>         console.log(post.title)                 // 印出陣列所有的title
>         const el = `<li>${post.title}<li>`      // 增加一段HTML
>         
>         // 把抓出來的title放到li的後面
>         lists.insertAdjacentHTML("beforeend", el)
>     })
> })
> 

> **open後面接的動詞不能改POST，取用API的時候。方法對，action錯，還是一樣取不到答案**
> req.open("GET", api)
> req.send()
```

### 6-2、reduce寫法
```markdown
> const rawData = req.responseText
> const posts = JSON.parse(rawData)
> 
> const result = posts.reduce((acc, {id, userId, title}) => {
>     return acc + `<li>${id} ${userId} ${title}</li>`
> })
> 
> lists.insertAdjacentHTML("beforeend", result);
> });
```

:::tip
在迴圈外定義一個變數，在裡面一直持續用他，通常有更好的寫法    
:::  



7、fetch寫爬蟲
------
```md
> const api = "網址"
> 
> const api = "https://jsonplaceholder.typicode.com/posts";
> 
> fetch(api)
>   .then((e) => {
>     console.log(e.json());       -> 印出 promise
>   })
>   .then((data) => {
>      console.log(data)           -> 印出json解析出來的資料    
>   })
>   .catch((err) => {
>      console.log(err)  
>   })
```

:::tip
所有的promise都可以用catch、then接住      
:::      



8、Event loop
------

有另外寫一篇完整的文章
[點我觀看完整個事件循環介紹](https://eagle0526.github.io/javascript/2022-09-24-What-Is-Event-Loop.html)


9、Stack VS Web api vs Queue 
------

來看瀏覽器真正在執行的細節

### 9-1、stack
(1) 瀏覽器堆疊的概念    
(2) 先進後出(FILO)  

### 9-2、Queue
(1) 先進先出(FIFO)  



### 9-3、呼叫堆疊

現在我們設定三個函式，並且在函式裡面呼叫另外個函式，看看觸發順序的狀況
```markdown
> function a() {
>     b()
> }
> 
> function b() {
>     c()
> }
> 
> function c() {
>     console.log("hi")
> }
> 
> a();

> a() 先執行先疊第一層，再來b()，再來c()
> 結束的時候，c() 先抽出來，再來b()，再來a()
> |            |
> | function c |
> | function b |
> | function a |
> --------------
```

:::info
**Stack Overflow名稱來源**  
stack -> stack 是存放執行code的地方  
overflow -> overflow 是溢滿的意思  
Stack Overflow -> 程式堆疊滿出來  
:::



### 9-4、重點提醒
(1) setTimeOut or Fetch 等等需要時間的函數  
(2) 使用的時候，會在webAPI排隊，接著會依照函數的執行順序，進到queue     
(3) queue的特性是先進先出   
(4) stack 裡面的東西要完全結束，Queue執行列裡面的東西才會執行   
(5) function結束時就會離開stack     
(6) webAPI沒有順序問題，把他想像成一個廣場      
(7) 監聽器 addEventListener("click") 會一直待在 webAPI 裡面     


### 9-5、vip Queue、normal Queue

(1) vip通道進到stack要看stack是否清空   
(2) 一般通道進到stack，要確認vip是否為空，再確認stack是否為空才能進去   

:::tip
**queue也有分VIP、normal**  
一般的queue  
setTimeout  
  
vip的queue  
fetch是vip  
:::


Ps. [可以去loupe網站看看+操作](http://latentflip.com/loupe/?code=JC5vbignYnV0dG9uJywgJ2NsaWNrJywgZnVuY3Rpb24gb25DbGljaygpIHsKICAgIHNldFRpbWVvdXQoZnVuY3Rpb24gdGltZXIoKSB7CiAgICAgICAgY29uc29sZS5sb2coJ1lvdSBjbGlja2VkIHRoZSBidXR0b24hJyk7ICAgIAogICAgfSwgMjAwMCk7Cn0pOwoKY29uc29sZS5sb2coIkhpISIpOwoKc2V0VGltZW91dChmdW5jdGlvbiB0aW1lb3V0KCkgewogICAgY29uc29sZS5sb2coIkNsaWNrIHRoZSBidXR0b24hIik7Cn0sIDUwMDApOwoKY29uc29sZS5sb2coIldlbGNvbWUgdG8gbG91cGUuIik7!!!PGJ1dHRvbj5DbGljayBtZSE8L2J1dHRvbj4%3D)




10、async寫法 - await記得要在最外圍加上async
------

```markdown
> async function getPost() {
>     const resp = await fetch(api)
>     const posts = await resp.json()
>     renderPosts(posts)
> }
> 
> getPost()
```


11、跨來源資源分享 CORS Cross  - 面試會問
------

(1) 如果對方沒有開 CORS policy 是抓不回來的     
(2) 如果未來碰到業主網站CORS的問題，打電話叫他們開      
(3) CORS policy test 可以檢測對方CORS有沒有開   

:::tip
**CORS是針對瀏覽器的行為，而不是針對JavaScript**  
這個行為就是指，假設你今天想從網頁存取別人的個人資料，會碰到資安問題  
  
如果未來有第二個可以控制瀏覽器的語言，這個語言也會碰到CORS的問題  
:::


### 11-1、CORS proxy

(1) 另外架一個伺服器 (用ruby node python)，去抓對方網站資料  
(2) 然後因為伺服器是我們自己的，可以把該伺服器的 CORS 打開      
(3) 就可以用我們自己的JavaScript把那台伺服器的資料抓過來    

