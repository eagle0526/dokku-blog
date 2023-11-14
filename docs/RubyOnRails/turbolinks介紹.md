---
sidebar_position: 1
---


0、前言
------

:::tip **寫此篇文章的動機**
身為一個"前"數位行銷人員，幫網站串接GTM，連接數據是一個非常常見的事！   
因此在學習ROR的時候，想當然也想要嘗試把GTM裝在ROR的網站上，不過在安裝前，發現Rails的Turbolinks     
用以往的GTM安裝方法，會讓網站產生BUG，因此決心來徹底了解Turbolinks到底是什麼東西？   
並且希望最後能成功把GTM成功裝上ROR的網站！   
  
**參考文章**   
[Rails Turbolinks™ 5 深度研究](https://www.writershelf.com/article/rails-turbolinks%E2%84%A2-5-%E6%B7%B1%E5%BA%A6%E7%A0%94%E7%A9%B6?locale=zh-TW)  
[軒軒的家 - Turbolinks 5 教學翻譯](http://doremi.logdown.com/posts/1680321)     
:::




1、什麼是Turbolinks？
------

:::tip
Turbolinks 是Javascript程式，是源自 Ruby on Rails 的 Web 加載優化方案  
通常會搭配Ruby On Rails一起做使用，目的是用來讓網頁切換更快速，主要是透過fetch HTML 直接抽換的方式來避免直接換頁要重新發送請求 
:::


### 1-1、Turbolinks怎麼做到換頁？

#### 1-1-1、xhr替換

點擊A頁面的連結後(連到pageB)，會用ajax的方式去抓pageB的內容物        
首先把頁面A的body抽掉，並把頁面B的body(XXX)，塞到pageA的body    
再來把頁面A的header留著，並看頁面B的header中，有哪些是A頁面header不存在的，會把不存在的塞到pageA的header    
最後把頁面A的URL，換成頁面B的URL    

```md
> pageA                        pageB
> 
> --------------   merge       ----------
> |  header    |   <------     | header |
> |            |               |        |
> --------------   ajax        ----------
> | link pageB |   ------>     | XXX  |
> |            |               | XXX  |
> |            |   replace     | XXX  |
> |            |   <------     | XXX  |
> --------------               --------
> 
```


:::tip
當你前往新的頁面，Turbolinks 會搜尋任何在新頁面下 head 的 script 元件，如果當前頁面不存在某些元件    
然後會新頁面的元件，附加在當前頁面的 head 地方，你可以利用這種方式來動態載入需要的額外 JavaScript 檔案   
:::


#### 1-1-2、原理
Turbolinks會去攔截畫面上所有的超連結(a-link)，如果今天他發現這個超連結是站內的超連結，他就會用xhr的方式，把連結的頁面抓回來

:::tip
普通你點一個超連結，會觸發的網頁事件是DOMContentLoaded，不過在Turbolinks底下，沒有這個事件，因為根本沒有換頁的行為，所以也就根本沒有觸發DOMContentLoaded的事件     
:::


### 1-2、實作Turbolinks
 
1、先抓到所有超連結     
2、所有超連結點擊的時候，先暫停預設行為     
3、最後用fetch方法，去把點擊的站內網頁內容抓過來    

```md
> const links = document.querySelectorAll("a")
> 
> links.forEach((link) => {
>     link.addEventListener("click", () => {
>         e.preventDefault()
> 
>         fetch(.....)
>         .then()
>         .catch()
>     })
> })
```



2、turbolinks的生命週期
------

### 2-1、cache是什麼

:::tip
在講解turbolinks的獨特生命週期前，要先介紹一下cache是什麼  
快取（cache, web cache, HTTP cache）是暫時存放 HTTP 回應的組件。只要滿足特定要求，就能在接下來的 HTTP 請求派上用場。   
[from MDN](https://developer.mozilla.org/zh-TW/docs/Glossary/Cache)    
:::



### 2-2、visit


剛剛前面有說到，Turbolinks 攔截同個網域下所有a的連結，當你點擊該連結，Turbolinks會取消預設動作，     
取而代之，Turbolinks 利用 [History API](https://developer.mozilla.org/zh-TW/docs/Web/API/History) 來改變瀏覽器的 URL, 使用 XMLHttpRequest 請求新頁面, 然後將結果 render 成 HTML  
在 render 時, Turbolinks 替換當前的 `<body>` 的全部元件, 並且合併 `<head>` 的內容. JavaScript 的 window 和 document 物件     
以及 HTML `<html>` 元件, 在每一個 render 之間則會保留起來.     
     
     
那visit是啥呢？其實就是每個連結切換的動作，代表一個visit(Each Navigation is a Visit)     
也就是說，從點擊頁面A的連結(連結到B)後，到turbolinks把網頁B頁面的內容render到A頁面上，這整個流程就是一個Visit    
     
Ps. Turbolinks 的導覽模型代表採取一個 action 來 visit 一個 location (URL)    
Ps. 包含改變瀏覽器歷史、發起網路請求、從快取回存某個頁面、render最後的回傳、更新捲動位置。      


:::tip
visit有兩種型態    
1、application visit，代表 advance 或 replace 的 action    
2、restoration visit，代表 restore 這個 action     
:::


### 2-3、application visit

一個 application visit 一定會發起一個網路請求，當收到回應Turbolinks會render該HTML，以完成當次visit.  

:::tip
什麼時候會觸發application visit呢？   
只要點擊一個 Turbolinks 的連結，或由程式呼叫 Turbolinks.visit(location)，就可以初始化Application visits 
:::



使用者點擊連結後～render畫面出來前，到底發生了哪些事？   
     
1、發出一個網路的需求：request a network request     
2、如果有 cache，就先把它 render 出來：render a preview of the page from cache   
3、移到先前 cache 的錨點(如果先前有的話，如果沒有就移動到頁面最上方)：scroll to the anchored element     
4、等到 server 回應內容，就再把它 render 出來：response arrived, renders HTML    
5、改瀏覽器的歷史紀錄：change to the browser’s history   
 

### 2-4、restoration visit

Turbolinks會從cache render畫面，而不用發出請求，否則，他會從網路取得一份新的資料

:::tip
什麼時候會觸發restoration visit呢？   
當你使用瀏覽器的前進或後退按鈕來瀏覽時，Turbolinks會自動啟動restoration visit.
:::


使用者點擊前進/後退的按鈕後，發生了哪些事情？

1、把 cache 的內容 render 出來：render a copy of the page from cache without making a request   
2、回到先前 cache 時的停留點：returns to this saved position    


Ps. Turbolinks在每一頁要離開時，會自動儲存捲軸位置，並且當返回瀏覽時會回復卷軸位置




### 2-5、visit完整生命週期


:::tip
Turbolinks 發出允許您追蹤導航生命週期和響應頁面加載的事件，除非另有說明，Turbolinks會在document object上觸發事件。   
   
Turbolinks emits events that allow you to track the navigation lifecycle and respond to page loading     
Except where noted, Turbolinks fires events on the document object.      
:::



#### 2-5-1、完整事件列表(Full List of Events)
```md
> turbolinks:click          單擊啟用 Turbolinks 的連結時觸發(fires when you click a Turbolinks-enabled link)
> turbolinks:before-visit   在 visit 某個位置之前觸發(fires before visiting a location)
> turbolinks:visit          visit 開始後立即觸發(fires immediately after a visit starts)
> turbolinks:request-start  在 Turbolinks 發出獲取頁面的請求之前觸發( fires before Turbolinks issues a network request to fetch the page)
> turbolinks:request-end    請求完成後觸發(fires after the network request completes)
> turbolinks:before-cache   在 Turbolinks 將當前頁面保存到緩存之前觸發( fires before Turbolinks saves the current page to cache)
> turbolinks:before-render  在 render 頁面之前觸發(fires before rendering the page)
> turbolinks:render         在 Turbolinks render 頁面後觸發。該事件在應用程序訪問緩存位置期間觸發兩次：
>                           一次是在呈現緩存版本之後(once after rendering the cached version)
>                           另一次是在呈現新版本之後(and again after rendering the fresh version)
> turbolinks:load           在初始頁面加載後觸發一次，並在每次 Turbolinks 訪問後觸發一次(fires once after the initial page load, and again after every Turbolinks visit)
```




#### 2-5-2、application visit(點擊turbolinks下的連結)
```md
> 無快取                            有快取(之前有拜訪過此頁面)

> 1、turbolinks:click               1、turbolinks:click                 -> 連結點擊
> 2、turbolinks:before-visit        2、turbolinks:before-visit          -> 觸發visit前
> 3、turbolinks:request-start       3、turbolinks:request-start         -> 開始提交需求
> 4、turbolinks:visit               4、turbolinks:visit                 -> 觸發visit
> 5、turbolinks:request-end         5、turbolinks:before-cache          -> 無快取、結束需求。 有快取、快取前
> 6、turbolinks:before-cache        6、turbolinks:before-render         -> 無快取、快取前。   有快取、渲染前
> 7、turbolinks:before-render       7、turbolinks:render                -> 無快取、渲染前。   有快取、渲染
> 8、turbolinks:render              8、turbolinks:request-end           -> 無快取、渲染。    有快取、結束需求
> 9、turbolinks:load                9、turbolinks:before-render         -> 無快取、load。    有快取、渲染前
>                                  10、turbolinks:render                ->                 有快取、渲染
>                                  11、turbolinks:load                  ->                 有快取、load
>                                   
```


#### 2-5-3、restoration visit(點下前進/倒退鍵)


```md
> 原先網頁內容"沒"改變                 原先網頁內容"有"改變

> 1、turbolinks:visit               1、turbolinks:request-start         -> "沒"改變、觸發visit。 "有"改變、開始提交需求
> 2、turbolinks:before-cache        2、turbolinks:visit                 -> "沒"改變、快取前。    "有"改變、觸發visit
> 3、turbolinks:before-render       3、turbolinks:request-end           -> "沒"改變、渲染前。    "有"改變、結束需求
> 4、turbolinks:render              4、turbolinks:before-cache          -> "沒"改變、渲染。      "有"改變、快取前
> 5、turbolinks:load                5、turbolinks:before-render         -> "沒"改變、load。     "有"改變、渲染前
>                                   6、turbolinks:render                ->                    "有"改變、渲染
>                                   7、turbolinks:load                  ->                    "有"改變、load
>                                   
```

#### 2-5-4、按下f5或者Refresh按鈕

```md
> turbolinks:load 
```


#### 2-5-5、重點整理

從上面的生命週期可以發現，不管是application visit還是restoration visit，最大的差異就在於cache的有無    

舉個例子，現在有兩個頁面，一個是A頁面，裡面有一個B頁面的連結，假設先前沒有拜訪過B頁面，點該連結的時候，會先觸發request-end後，才觸發cache，相反的，如果今天有拜訪過B頁面，觸發request-start後，就會觸發cache。   

上面的例子是application visit，那restoration visit呢？一樣舉個例子，假設今天先進到A頁面、點擊A頁面的B頁面連結後，把B頁面的內容置換到A頁面(url也換了)，接著點擊瀏覽器的上一頁，而他觸發的是before-cache，不會再從server端提交需求，並拿到回應。   

但！假設今天按上一頁回到A頁面的時候，如果上一頁的內容改變了(跟cache裡面存取的A頁面不一樣)，turbolinks就會再重新跑一個提交需求，並把頁面存取到cache裡面。     


3、Idempotent是什麼？
------

既然知道turbolinks的生命週期，也知道turbolinks執行的時候，造成JS發生詭異的行為，是因為cache和server內容不一樣，那要怎麼避免不一樣呢？這個turbolinks文件中也有說到：掛在Turbolinks上的程式，必須要是：**Idempotent**     

### 3-1、Idempotent詳解

:::tip
**Idempotent是什麼？**     
在數學裡，Idempotent有兩種主要的定義。     
1、在某二元運算下，Idempotent元素是指被自己重複運算（或對於函數是為複合）的結果等於它自己的元素。例如，乘法下唯一兩個Idempotent實數為0和1。    
2、某一元運算為Idempotent的時，其作用在任一元素兩次後會和其作用一次的結果相同。例如，高斯符號便是Idempotent的。    
[from wiki](https://zh.m.wikipedia.org/zh-tw/%E5%86%AA%E7%AD%89)

不過看完wiki，還是不知道他在說啥，決定直接引用紅寶鐵軌客的翻譯
**他基本上是一個數學與電腦科學的專用字，是指這個程式或是數學運算式必須要每次執行結果都一樣！**
:::


### 3-2、沒有符合Idempotent例子
知道Idempotent的意思後，我們來舉個例子，如果今天程式碼沒有符合Idempotent，會發生什麼問題？
我有一個頁面，頁面上想要顯示使用者的當地時間，而這個頁面有很多元件，想要把這些元件，按照時間做分配(1/20的文章在一塊，1/22的文章在一塊)   

如果今天我用data-timestamp屬性註記一組元件，代表元件的UTC(世界時間標準)建立時間，接著我用一組JS的函式，來查詢這個頁面所有組件的時間，並把所有時間轉成使用者的當地時間，最後在每一個新的一天插入日期的標頭。  

如果今天你在turbolinks下實作這組函式，會發現一個大問題！！大家可以想一下會發生啥問題，給大家想3s～   
     
... 1S   
... 2S   
... 3S   
     
我們來實際走一次流程，假設我今天進到這個頁面，JS的函式如預期般的運作，並且會根據不同的日期，幫網頁內容插入不同日期的標頭，畫面正常的顯示出來，使用者接著點擊了另外一個頁面，Turbolinks保存一個頁面轉換後的副本到cache(有插入標頭的)，當你按下返回鍵(上一頁)後，再次啟動turbolinks:load，此時你的JS又會再次插入標頭一次！

### 3-3、這裡會發生兩個問題！
#### 3-3-1、第一個問題
假如今天你只是單純把時間印在A頁面上，像下面這樣，發生的問題就是，因為網頁是直接抓cache的內容，所以當下的時間點是第一次進入A頁面的時間點(第二次進入A頁面，顯示時間點是錯的)
```md
> <%= Time.now %>

> 第一次進到A畫面
> 時間點顯示 - 2023-01-28 18:13:55

> 第二次進到A畫面(1分鐘後)
> 時間點顯示 - 2023-01-28 18:13:55(完全沒變)
```

#### 3-3-2、第二個問題
如果你有用JS去更改Time.now，把時區更改成每個使用者登入的時區(美國人登入就改成美國時區、日本人登入就改成日本時區)，會造成標頭被再次插入(出現了兩次)，


### 3-4、用Idempotent解決問題
!!這一段我自己也不懂!!

一個讓轉換成為idempotent的技巧是，在每一次設定data屬性時，無論你之前是否已經設定都要檢查。
當 Turbolinks 從快取回存你的頁面時，這一些屬性仍然會存在，然後在你的轉換 function 中偵測這些屬性是否已經有處理過。

另外一個比較可靠的技巧是簡單的在轉換 function 本身檢查。
在上面的按日期分類的例子，意思就是在插入新的資料之前，檢查 div 元件的 date 是否已經存在，這可以優雅的處理尚未被原始轉換程式處理的新增元件.


:::tip
雖然這樣說，但是Idempotent其實蠻難解決的，原因下面會提到   
:::


4、如何關掉Turbolinks
------

前面有說到，只要讓Turbolinks符合Idempotent，就不會出問題，但！！要符合Idempotent其實蠻難的，因為程式執行後，只要更改原先的資料或者移動DOM位置之類的，就會不符合Idempotent，因此真的發生太多詭異的JS的時候，就...把這個頁面的Turbolinks給關掉吧！畢竟想辦法先讓程式正常運行比較重要   

:::tip
關掉Turbolinks有兩個方法   
1、關掉指定網頁的turbolinks cache   
2、reload網頁  
:::


### 4-1、關掉turbolinks cache


先來看一下cache 的行為：當 Cache 發生前，Turbolinks 會發出 turbolinks:before-cache 事件，當網頁還是顯示 cache 時的資料時，`<html>` 的 attribute 會加入 data-turbolinks-preview，但是我發現只有在 application visit 已經有 cache 的網頁才會有這 attribute，其他不會，因此要讓網頁不要顯示cache內容，只要把preview關掉就好，關掉 Turbolinks 的 cache 就是 no-cache，兩個都是加在 `<head>` 內，以下是關掉的方法：

#### 4-1-1、關掉preview
```md
> application.html.erb

> <head>
>   ...
>   <meta name="turbolinks-cache-control" content="no-cache">
> </head>
```

這樣可能會有一點模糊，我實際舉個例子好了，假設我今天在index.html.erb，最下方新增一個JS的函式，這個函式可以在你進入此頁面的時候，在這個畫面上增加一個按鈕區塊
```md
> index.html.erb

> <script>
>     var btn = document.createElement("button")
>     btn.innerHTML = "click me"
>     document.body.appendChild(btn)
> </script>
```

這樣當我們第一次進到這個頁面，就會有一個按鈕出現在畫面上，不過假如今天你進到站內另外一個畫面，接著再點上一頁/下一頁，就會發現按鈕會一次跑出兩個，會這樣就是因為原本頁面已經有cache的關係，要解決這件事只要把剛剛上面no-cache那一段加上去就可以解決。



#### 4-1-2、關掉a連結的turbolinks
剛剛說的是頁面的cache，今天如果直接在連結點下去時候，不要讓turbolinks運作，只要後面加上這一段，就可以把turbolinks關掉
```md
> <%= link_to ...., data:{ turbolinks: "false" } %>
```



#### 4-1-3、小總結

還蠻建議關掉 preview 的，畢竟很多瀏覽器都已經有內建的 cache 了，關掉 cache 有一個好處是 body 內的 javascript 就不會執行兩次了。


### 4-2、reload

幾的常用的 reload page 方式：
```md
> location.reload() 
> location=location 
> location.assign(location) 
> document.execCommand('Refresh') 
```

5、串接GTM
------

終於！！前面講了這麼多的turbolinks的作用、生命週期，終於可以開始把rails網站接上GTM了～   
Ps. [參考連結](http://labs.wrprojects.com/how-to-use-google-tag-manager-with-rails-and-turbolinks/)  


```md
> 原本放在head的程式碼

>  <!-- Google Tag Manager -->
> <script>(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
> new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
> j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
> 'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
> })(window,document,'script','dataLayer','GTM-WWSRBV2');</script>
> <!-- End Google Tag Manager -->

> 原本放在body的程式碼

> <!-- Google Tag Manager (noscript) -->
> <noscript><iframe src="https://www.googletagmanager.com/ns.html?id=GTM-WWSRBV2"
> height="0" width="0" style="display:none;visibility:hidden"></iframe></noscript>
> <!-- End Google Tag Manager (noscript) -->
```

更改過後的head程式碼，把下面這一段，塞進head裡面就可以
```md
<!-- Google Tag Manager trigger for Turbolinks -->
<script type="text/javascript">
$(document).on('page:change', function(){
var url = window.location.href;

dataLayer.push({
'event':'pageView',
'virtualUrl': url
});
});
</script>
<!-- End Google Tag Manager trigger for Turbolinks -->


<!-- Google Tag Manager -->
<noscript><iframe src="//www.googletagmanager.com/ns.html?id=[YOUR ID]"
height="0" width="0" style="display:none;visibility:hidden"></iframe></noscript>
<script>(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
'//www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
})(window,document,'script','dataLayer','[YOUR ID]');</script>
<!-- End Google Tag Manager -->
```



雖然這樣就裝好了，但是我看紅寶鐵軌客的說明，這樣會遇到點擊連結後，GTM的debug console會不見，不過目前我還沒遇到(可能是專案還不夠大)，之後遇到會在這邊補充！除了debug console不見以外，鐵軌客還有提出一些其他的問題，不過我目前遇到的專案也沒辦法體驗到(QQQ..希望可以趕快遇到大專案讓我試試)，所以也只能等到遇到問題的時候再來補充這篇文章了。



6、完整刪除turbolinks
------

前面有提到部分拿掉turbolinks，不過如果今天真的不想用turbolinks，要怎麼拿掉呢？
```md
> 1、移除 turbolinks gem(移除完後記得bundle install)
> 2、移除：<meta name="turbolinks-cache-control" xxx>，這有可能在任何一頁，做個 search 吧！
> 3、javascripts/application.js -> 移除：//= require turbolinks
> 4、assets/javascripts/ 中所有的 js 跟 coffee 都要改成：
>    js: $(document).on('turbolinks:load', function() {  ⇒  $(document).ready( function() {
>    coffee: jQuery(document).on 'turbolinks:load',  ⇒  $(document).ready
> 5、移除 'data-no-turbolink'，data-turbolinks 屬性
> 6、Javascript 中：改寫任何有 Turbolinks.visit、Turbolinks.clearCache()、Turbolinks.xxx 的程式，
```




7、總結
------

終於！最後成功把ROR網站成功串接GTM了，並且也有把GA成功串接上，不過問題就是目前做的專案太小，所以沒辦法遇到軌道客說的一堆問題，希望之後能把這一塊補完！！





