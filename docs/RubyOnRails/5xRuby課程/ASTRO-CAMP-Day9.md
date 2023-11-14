---
title: ROR/JavaScript(03)
sidebar_label: "DAY9. [ROR] JavaScript-3"
description: JavaScript
last_update:
  date: 2022-10-14
keywords:
  - 5xRuby
  - JavaScript
sidebar_position: 10
---




1、陣列 - 續
------

### 1-1、slice語法
定義：slice() 方法會回傳一個新陣列物件，為原陣列選擇之 begin 至 end（不含 end）部分的淺拷貝（shallow copy）。而原本的陣列將不會被修改。

假設今天宣告一個陣列，並再宣告一個變數等於前面那個陣列，如果今天修改了變數中陣列的元數，前面那個陣列也會被修改
```markdown
> const list = ["a", "b", "c", "d"]
> cost hi = list;
> hi[0] ="x"                       -> 這樣兩個印出來，都會兩個都會改到
> 
> console.log(list)               # ["x", "b", "c", "d"]
> console.log(hi)                 # ["x", "b", "c", "d"]
```

### 1-2、slice 複製一個陣列
如果今天想要一個新的陣列，且不想要影響到原陣列
```markdown
> const list = ["a", "b", "c", "d"]
> cost hi = list.slice()           -> 複製一個陣列出來
> hi[0] ="x"                       -> 這樣兩個印出來，不會影響到list
```




### 1-3、尋找陣列中是否有某元素存在

#### 1-3-1、方法一:indexOf()
indexOf() = indexOf() 方法會回傳給定元素於陣列中第一個被找到之索引，若不存在於陣列中則回傳 -1。

```markdown
> const list = ["a", "b", "c", "d"]
> console.log(list.indexOf("c"))      # 2
> console.log(list.indexOf("u"))      # -1
```



#### 1-3-2、方法二:includes()
includes() = includes() 方法會判斷陣列是否包含特定的元素，並以此來回傳 true 或 false。
```markdown
> const list = ["a", "b", "c", "d"]
> console.log(list.indexOf("c"))      # true
> console.log(list.indexOf("u"))      # false
```



### 1-4、印出陣列中所有元素

```markdown
> const list = ["a", "b", "c", "d"]
```

#### 1-4-1、for迴圈
```markdown
> for(i = 0; i<= lst.length-1; i++) {
>     console.log(lst[i]);
> }
```

#### 1-4-2、forEach - 遍歷元素
forEach() 方法會將陣列內的每個元素，皆傳入並執行給定的函式一次。

```markdown
> list.forEach((item) => {
>     console.log(item)
> })
```

#### 1-4-3、forEach 實戰 - 印出陣列中所有元素類別(typeof)
```markdown
> const lst = [1, "2" ,3 ,"4" ,5]
> lst.forEach((item) => {
>     return console.log(typeof item);             # Number String Number String Number
> })
```


### 1-5、find方法 - 找指定元素


find = find() 方法會 **回傳第一個** 滿足所提供之測試函式的元素值。否則回傳 undefined。

```markdown
> const list = [1, 2, 3, 4, 5]
> 
> const result = list.find((x)=>{
>     return x > 3;                  # 4
> })
> console.log(result);
```


#### 1-5-1、find實戰 - 得到大於6的偶數
```markdown
> const lstNum = [1,2,3,4,5,6,7,8,9,10]
> const newLst = lstNum.find((x)=>{
>     return (x > 6 && x % 2 === 0)  ;
> })
> console.log(newLst);
```



### 1-6、實戰 - 得到所有偶數

#### 1-6-1、forEach方法
```markdown
> const list = [1, 2, 3, 4, 5]
> const result =[];               -> 用 forEach，要先在外部宣告一個變數，這樣裡面取用的時候才能用
> list.forEach((item) => {
>     result.push(item*2)            # [2,4,6,8,10]
> })
```


#### 1-6-2、map方法 - 這個好用在會自動產生新陣列
map = map() 方法會建立一個新的陣列，其內容為原陣列的每一個元素經由回呼函式運算後所回傳的結果之集合。

```markdown
> const list = [1, 2, 3, 4, 5]
> const newResult = list.map((item) => {
>     return item * 2;
> })
> 
> console.log(newResult);
```



### 1-7、filter - 單純過濾陣列

filter() 方法會建立一個經指定之函式運算後，由原陣列中通過該函式檢驗之元素所構成的新陣列。

```markdown
> const list = [1, 2, 3, 4, 5]
> const newResult = list.filter((item) => {
>     return item > 3;                      # [4, 5]
> })
> 
> console.log(newResult);
```


#### 1-7-1、filter實戰 - 篩選出陣列中的字串，長度大於3的元素
```markdown
> const list = ["a", "aaaa", "cc", "ccccc", "dddd"];
> 
> const newResult = list.filter((item) => {
>     return item.length > 3;
> })
> 
> console.log(newResult);
```





### 1-8、加總陣列中的元素


```markdown
> const list = [1, 2, 3, 4, 5]
```

#### 1-8-1、forEach 方法
```markdown
> let result = 0;
> list.forEach((item) => {
>     result = result + item
> })
> 
> console.log(result)         # 15
```

### 1-8-2、reduce 方法
reduce() 方法將一個累加器及陣列中每項元素（由左至右）傳入回呼函式，將陣列化為單一值。

```markdown
> const result = list.reduce((累加值, 目前值) => {
>     return 累加值 + 目前值;
> }, 起始值)
 
> const result = list.reduce((x, y) => {
>     return x + y;                 
> }, 0)
> console.log(result)                         # 15
```



#### 1-8-3、用 reduce 取最大值

用 reduce 的特性，reduce 照著陣列順序 2 個 2 個抓出來，然後用 if 來比較兩個數字
```markdown
> const list = [1, 5, 2, 19, 7, 18]
> const result = list.reduce((acc, cv) => {
>     if (acc > cv) {
>         return acc                           -> 兩兩比較，比較大的值放在累計值
>     } else {
>         return cv                            ->         比較小的值放在目前值
>     }
> })
> 
> console.log(result)                          # 19 -> 最大的值是最後的累計值
```



:::tip
**何謂高階函數 higher-order function**  
1、可以把 function 當作參數傳入另一個 function  
2、可以回傳 function  
只要符合上面其中一點，就可以說是高階函數      
:::




2、物件 Object
------

:::tip **物件是什麼？**   
物件 = 一個東西 = 屬性 + 行為  
:::  



### 2-1、創造一個物件
假設今天有一個物件 aa       
(1) key = a, b, c, attack   
(2) value = 123, 333, "ddd", function   
  
```markdown
> const aa = {
>     a: 123,
>     b: 333,    
>     c: "ddd",
>     attack: function() {
>         console.log("hi")
>     }
> }
```


### 2-2、拿取key中的值

下面兩種方法都可以  

#### 2-2-1、方法一 : aa["c"]
```markdown
> console.log(aa["c"])    # ddd
```
#### 2-2-2、方法二 : aa.c
```md
> console.log(aa.c)       # ddd
```

#### 2-2-3、什麼時候只能用[]拿取值

變數字數是組合的方式，就一定要用此種方式   
```md 
> const myKey = "na" + "me"
> console.log(aa["myKey"])
```

### 2-3、呼叫物件的function
```markdown
> aa.attack()              # hi
```


### 2-4、物件的 CRUD

```markdown
#### C - 新增物件中的值，把key塞進物件中

> aa.test = "fff"         
> console.log(aa)          # { a: 123, b: 333, c: 'ddd', attack: [Function: attack], test: 'fff' }
```

```markdown
#### R - 讀取指定key的value

> console.log(aa.b)        # 333
```

```markdown
#### U - 更新物件的key

> aa.a = "no"
> console.log(aa)          # { a: "no", b: 333, c: 'ddd', attack: [Function: attack], test: 'fff' }
```

```markdown
#### D - 刪除值物件中的值

> delete aa.b;
```

### 2-5、如果今天key值是數字，要怎麼取value

```markdown
> const hero = {
>     0: "a",
>     1: "b",
>     2: "c"
> }
> 
> console.log(hero[1])         # b
```



3、DOM 文件物件模型
------

(1) HTML不是物件，為了方便操作，瀏覽器會根據HTML結構，轉成一個一個物件      
(2) DOM是瀏覽器把HTML物件化後的結果  
(3) HTML不是DOM元素     

:::tip
JS 無法直接取用 HTML，中間有一層瀏覽器  
JS 會操作瀏覽器來更改瀏覽器的介面(但是HTML 的 code 不會被動到)  
:::

### 3-1、querySelector

如果今天使用querySelector，並且有多個id都是一樣，那JS只會抓到第一個


```markdown
> 我先給五個同id的div

> <div id="test">1</div>
> <div id="test">2</div>
> <div id="test">3</div>
> <div id="test">4</div>
> <div id="test">5</div>

> 抓 id 為 test 的元素
> const newId = document.querySelector("#test")

> 把剛剛抓到的元素，點到他會印出"123"
> newId.addEventListener("click", () => {
>     console.log("123");
> })

> 這樣最後在瀏覽器那邊會發現，只有點到第一個元素，可以觸發這個console.log
```



### 3-2、script的觸發、defer
:::info
如果印出來的結果是 null  
有可能是寫在HTML的 script src 寫在太前面  
導致物件化過程比較慢  
:::



#### 3-2-1、方法一 : 把script放在抓取物件的後面
```markdown
> 解決方法
> <div id="abc">hi</div>
> <script src="dom.js"></script>
```
#### 3-2-2、方法二 : 直接加一個defer
```md
> <script src="dom.js" defer></script>
> 
> defer 是延遲執行的意思，這個加上可以讓JS等HTML執行完再執行
```


### 3-3、getElementById、querySelector

先給一個DIV，並用兩種語法來抓取div的id
```HTML
<div id="abc">123</div>
```

#### 3-3-1、舊方法 - getElementById
```markdown
> const mydiv = document.getElementById("abc")
```

#### 3-3-2、新方法 - querySelector 
這個方法比較好用，因為後面是使用選取器  
```md
> const mydiv2 = document.querySelector("#abc");
```



### 3-3-3、實際應用 - 選取器
```markdown
> HTML
> <ul>
>     <li></li>
>     <li></li>   -> 我要抓這個
>     <li></li>
> </ul>
```

用nth-child來抓指定標籤
```markdown
> const mydiv2 = document.querySelector("ul li:nth-child(2)");
```

:::tip
用 querySelector 抓元素，可以用任何 css 選取器的方法      
:::


### 3-4、getElementsByClassName、querySelectorAll - 一次抓取多個元素

先在HTML設定li的class，並用兩種方法抓取所有元素
```markdown
> HTML
> <ul>
>     <li class="list"></li>
>     <li class="list"></li>
>     <li class="list"></li>
> </ul>
```


#### 3-4-1、方法一 : getElementsByClassName
```markdown
> const lists = document.getElementsByClassName("list")

> console.log(lists)                  # HTMLCollection(3) [li.list, li.list, li.list]
```

#### 3-4-2、方法二 : querySelectorAll
```markdown
> const lists2 = document.querySelectorAll(".list")

> console.log(list2)                  # NodeList(3) [li.list, li.list, li.list]
```



### 3-5、HTMLCollection vs NodeList

可以發現剛剛兩種抓取ID的方法，抓到的東西不同

(1) getElementsByClassName 抓到的是HTMLCollection，要把裡面東西印出來只能用for迴圈  
(2) querySelectorAll 抓到的是NodeList，可以使用forEach把裡面東西印出來  


#### 3-5-1、HTMLCollection  
getElementsByClassName 抓到的是一個物件，裡面包著所有東西  
如果想要把物件裡面的東西印出來，只能用for迴圈印出來  
```markdown   
> for(let i = 0; i < oldBlock.length; i++) {
>     console.log(oldBlock[i]);
> }

> 上面的 for迴圈，會把剛剛抓到的區塊，全部印出來
> <li class="list"></li>
> <li class="list"></li>
> <li class="list"></li>
```


#### 3-5-2、NodeList - 多了forEach功能  
querySelectorAll 抓到的是一個物件，也包著所有東西  
但是想要把裡面東西抓出來的話，可以用forEach拿法  

```markdown
> newBlock.forEach((item) => {
>     console.log(item);
> })

> 用forEach把內容物全部印出來
> <li class="list"></li>
> <li class="list"></li>
> <li class="list"></li>
```



### 3-6、textContent - 更改文字

剛剛抓到指定元素後，可以使用textContent來更改元素的文字

先設定一個div id
```markdown
> HTML
> <div id="abc">hi</div>
```

先抓到想改變文字的區塊，再利用textContent更改該區塊的文字    
```markdown
> const mydiv = document.getElementById("abc")

> 把這個物件裡面的值，改成 "aaa"
> mydiv.textContent = "aaa"
```


### 3-6-1、實戰練習 querySelectorAll + textContent

利用 querySelectorAll 一次抓相同class的物件，並用 textContent 更改文字

```markdown
> HTML
> <div class="aa">1</div>
> <div class="aa">2</div>
> <div class="aa">3</div>
> <div class="aa">4</div>
> <div class="aa">5</div>
```

#### 使用 forEach + textContent更改每一個div的文字
```markdown
> sinBlock.forEach((e) => {
>     e.textContent = "帥哥"              # 這可以把HTML的五個div文字都改成帥哥
> })  
```


#### 使用 for 迴圈 + textContent
```markdown
> for(let i = 0; i < sinBlock.length; i++ ) {
>     sinBlock[i].textContent = "帥哥"    # 這可以把HTML的五個div文字都改成帥哥
> }

```


### 3-7、innerHTML - 渲染HTML

先創造一個div
```HTML
<div id="test">hi</div>
```

用querySelector抓到該區塊後，用innerHTML改變此區塊的標籤
```markdown
> 把該元素變成 h1 元素
> mydiv.innerHTML = "<h1>dsf</h1>"
```


### 3-8、classList 類別元素

用 classList 可以修改抓取元素的 class   

首先先設定CSS
```HTML
<style>
    .red {
        font-size: 20px;
        color: red;
    }
    .green {
        font-size: 30px;
        color: green;
    }
</style>
```

再利用classList.add，更改該區塊的CSS
```markdown
> 把元素的class，修改class為.red的屬性
> mydiv.classList.add("red")

> 把元素的class，修改class為.green的屬性
> mydiv.classList.add("green")
```




4、事件監聽器 addEventListener
------

什麼時候會有 **事件** 發生？網頁只要有任何變動，都會有事件發生  
加一個事件監聽器，這個可以解決 script scr 被晚寫的狀況  
整個網頁會先執行完後，再來執行這些事件  


### 4-1、解決script問題
像下面這樣寫，就可以解決script先後讀取的問題  


#### 4-1-1、舊寫法 - 使用DOMContentLoaded 
```markdown
> document.addEventListener("DOMContentLoaded", () => {
>     const mydiv = document.querySelector("#test");
>     mydiv.innerHTML = "<h1>dsf</h1>";
> });
```
#### 4-1-2、新寫法 - 加一個defer，讓js晚一點執行
```md
> <script src="dom.js" defer></script>
```




### 4-2、callback function 回呼函數
```markdown
> myDiv.addEventListener("click", (e) => {           => 這個 e 稱為 callback function
>     console.log(123);
>     console.log(e);                    
> })

> 上面兩個log分別會印出下面這兩個
> 123
> PointerEvent {isTrusted: true, pointerId: 1, width: 1, height: 1, pressure: 0, …}
```

:::tip
回傳函數的功用 -> 把函數當作引數在傳  
:::  


### 4-3、重複使用監聽元素

先設定兩個 div
```HTML
<div id="test">hi</div>
<div id="test2">goodby</div>
```


抓取後，在用addEventListener監聽器，重複觸發中間段設計的監聽函數
```markdown
> const myDiv = document.querySelector("#abc");
> const youDiv = document.querySelector("#xyz);
> 
> const clickHandler = (e) => {
>     console.log(234)
>     console.log(e.target);        -> 可寫可不寫, 這一行只是提醒你 那個 e 會傳下來
> }
> 
> myDiv.addEventListener("click", clickHandler);
> youDiv.addEventListener("click", clickHandler);
```

各自點擊元素後，會印出下面訊息
```markdown
> 點擊 id = "test" 的div
> 234
> <div id="test2">goodby</div>
 
> 點擊 id = "test2" 的div
> 234
> <div id="test">hi</div>
```




4-4、預設行為 - preventDefault

很多標籤都有預設行為，像是超連結 or 表單，我們可以利用 preventDefault 來先暫停，這些預設行為
  
  
設定一個a標籤
```HTML
<a href="google.com">Google</a>
```

抓到此標籤後，把a標籤的預設行為關掉
```markdown
> const link = document.querySelector("a")
> 
> link.addEventListener("click", (e) => {
>     e.preventDefault();
>     console.log("clicked");
> })
```




### 4-4、實戰 - 練習寫一個計數器 (減法的值不可以 < 1)


```markdown
> HTML
> button id="minus">-</button>
> input type="number" id="counter" value="1">
> button id="plus">+</button>
```

```markdown
> JavaScript
> / 抓中間的顯示器
> const counter = document.querySelector("#counter")
> 
> / 抓 + 號
> const plusBtn = document.querySelector("#plus")
> lusBtn.addEventListener("click", () => {
>    counter.value = Number(counter.value) + 1
> )
> 
> 
> / 抓 - 號
> const minusBtn = document.querySelector("#minus")
> minusBtn.addEventListener("click", () => {
>    const value = Number(counter.value)
>    if (value > 1) {
>        counter.value = value - 1    
>    } 
> )
> 
```


