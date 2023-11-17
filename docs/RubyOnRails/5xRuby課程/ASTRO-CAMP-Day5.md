---
title: ROR/JavaScript(01)
sidebar_label: "DAY5. [ASTROCamp] JavaScript-1"
description: JavaScript
last_update:
  date: 2022-10-06
keywords:
  - 5xRuby
  - JavaScript
sidebar_position: 6
---



1、JS前言
------

### 1-1、前端工程師

html結構、css樣式、JavaScript行為


### 1-2、JavaScript 跟 Java 的關係

JS當初取名的來由，是去蹭Java名字的熱度(當初Java很紅)，除此之外都沒啥關係


### 1-3、JavaScript 跟 ES6 的關係
- ECMA       ： 歐洲電腦製造商協會  
- TC39       ： 第39號委員會，負責JS的標準化    
- ECMA-262   ： JS的規格    
- ECMAScript ： 根據ECMA262規範所設計出來的程式語言規則  
- JS         ： 某一款符合ECMAScript 的程式語言  
- ES6        ： ECMAScript 第六代   


### 1-4、為什麼不學最新版本就好
瀏覽器跟不上


### 1-5、JS的runtime

**runtime是什麼？** 
runtime就是執行環境 (這個程式語言可以在哪些環境執行)        

以下三個是JS的執行環境  

:::tip
(1)瀏覽器     
(2)Node   
(3)Deno   
:::


### 1-6、JS 推薦外掛
- Babel JS ： 如果語法寫對，幫助 hightlighting 語法
- Prettier ： 幫忙排列程式碼
- Material Icon Theme ： 根據你的檔案類型，在檔案左邊給相對 Icon
- Live Server


***


2、JS 的 hello world
------

### 2-1、console.log()
```markdown
> console.log("hi, world");
```

### 2-2、HTML連結JS檔案

```markdown
 > <script src="file1.js"></script>
 ```


### 2-3、alert、confirm

```markdown
> alert(" don't touch me ")   -> 彈出警告視窗
> confirm(" are u sure? ")   -> 彈出建議視窗
```



3、變數、常數
------

### 3-1、宣告變數 - variable(var)

變數就像一個標籤，標籤裡面有很多內容物，想要拿出內容物的時候，就去呼叫他

```markdown
> var a = 2;

> "a" = 變數名字
> "=" assign 指定的意思
> "2" 是值
```

### 3-2、變數設定要注意
 
1、不能用數字開頭的字  
2、不能用保留字 - 看起來像語法的字 ex. var if = 2  
3、注意大小寫有差別      



### 3-3、未定義undefined vs 沒有定義not define

```md
> 這兩個是有差異的      
> var a;   console.log(a)   -> undefined    
>          console.log(b)   -> not define   
```
Ps. undefined 是一個值，not define 就只是純粹啥都沒有   



### 3-4、變數如果還沒宣告就拿來用

**變數提升 variable hoisting**

這個東西是JS的機制，不過很不妙，會造成很多意外
```markdown
> console.log(a);    -> 會印出undefined  
> var a = 1;
> 
> console.log(b)     -> 會印出not define
> b = 1
```



### 3-5、let 設定變數

這兩個看起來都一樣，但差異很大
```markdown
> var a = 1;  
> let b = 2;  
```

**如果今天用let，可以直接解決變數提升的問題**
```md
> console.log(a)     -> 會印出not define  
> let a = 1  
```



### 3-6、const 設定常數

假如今天跟別人一起協作，有某個數值你不想被更改，你就可以設定常數
```markdown
> const a = 1
> a = 2;
>
> console.log(a);      -> 印出 Assignment to constant variable.
```

**const 的定義不是不能修改，是不能re assign**
```md
> 舉個例子，如果今天我用const宣告一個陣列
> const a = ["a","b","c"]

> (1) 情境一 : 修改陣列裡面的值
> a[0] = 123
> console.log(a)              -> 會印出 [123, "b", "c"] (可以修改裡面的值)

> (2) 情境二 : 重複賦值
> a = ["c", "d", "e"]         -> 會印出 Assignment to constant variable.（不能重新指派）
> console.log(a)
> 
```

### 3-7、const vs. let vs. var

(1) 寫JS 選擇上使用順序 const > let > var  
(2) var也可以用，不過範圍大了一點  
(3) 千萬不要沒宣告就使用，會造成很多全域污染  

```md 
> const 和 let、var的差異是，const是用來設定常數的(不可re-assign)  
> 而 let 和 var 的差異是，var因為設計漏洞的關係，會造成 **變數提升**  
> 發生變數提升會讓後指派的 var 變數，console出來會變成 undefined，而不是not defined  
>
> ex. console.log(a);   # undefined  
>     var a = 1;  
```

**undefine 和 not define 的差異在於，undefined是一個值，not define是啥都沒有**  
```md
> 而let就是解決變數提升的問題  
>  
> ex. console.log(b);    # not define  
>     let b = 1;  
```


4、a++ vs ++a
------

a++和++a的差異之處

```markdown
> var a = 1
> console.log(a++)        # 1   -> 先印出a，再執行++
> console.log(a)          # 2   
> 
> var b = 1
> console.log(++b)        # 2   -> 先執行++，再印出b
> console.log(b)          # 2
```



5、原始型別 vs 物件型別
------


### 5-1、原始型別有哪些  
數字、字串、真假值、之類的  

### 5-2、物件型別有哪些  
大括號、中括號、函數、物件之類的


### 5-3、為什麼要有不同型別

為了不同型別之間的搭配使用  
```markdown
> ex. console.log(1+2)  
>     console.log("a"+"b")  
```



### 5-4、字串顯示

使用``，並搭配${}，就可以把宣告的變數，塞在一個字串裡面

```markdown
> let num = 123  
>   
> console.log("aaa")           # aaa  
> console.log('aaa')           # aaa  
> console.log(`aa${num}a`)     # aa123a -> 這種顯示方式可以塞變數在裡面  
>
> console.log(`she said: "hi"`)# she said: "hi" -> 要在字串裡顯示""，要記得用``  
> console.log(" she said: \"hi\" ")                或是用跳脫字元\  
```


### 5-5、改變型別、確認型別typeof

typeof 運算子會傳回一個字串值, 指出未經運算 (unevaluated) 的運算元所代表的型別。

```markdown
> let a = 2;  
> console.log(typeof a)          #Number  

> let b = String(b)  
> console.log(typeof b)          #String  

> console.log(typeof(typeof(a))) #String -> 真實樣式是這樣 typeof(Number)
```


### 5-6、NaN介紹 

**NaN = Not a Number**

5-6-1、數字除以字串，會印出NaN
```markdown
> console.log(2/"c")               #NaN
```

5-6-2、NaN是一個數字，不過他代表不是數字
```md 
> console.log(typeof NaN);         #number  -> NaN是一個數字，不過他代表不是數字  
>                                              就像Null他存在，不過他代表不存在  
```

5-6-3、JS唯一一個自己不等於自己的值
```md
> console.log(NaN === NaN);        #false
```



6、流程控制 Flow Control
------

### 6-1、if else判斷式的寫法

```markdown
> let age = 20  

> if (age >= 18) {  
>  console.log("已成年")      
> } else if ( age > 10 && age < 18 ) {        # &&而且、||或  
>  console.log("青少年")  
> } else {  
>   console.log("未成年")     
> }  
```

### 6-2、三元運算子

原本很長的if else判斷式，改用一行完成

#### 6-2-1、判斷age是否大於18歲，如果成立 "已成年" 否則 "未成年"
```markdown
> age >= 18 ? console.log("已成年") : console.log("未成年")
```

#### 6-2-2、寫三元運算子順便宣告變數
```md
> let result = age >= 18 ? "已成年" : "未成年"
> console.log(result)                          #另一種三元運算子
```

#### 6-2-3、三元運算子巢狀寫法

```md
(1) 如果a<1，就印出C    
(2) 如果a>1，並且b>2，就印出A   
(2) 如果a>1，並且b<=2，就印出B  
```


```md
> 用普通if判斷式寫
> if (a>1) {
> 
>   if (b>2) {
>       A
>    } else {
>       B
>    }
>   else {
>    c
>   } 
>
> }
```

三元運算子呈現
```md
> (a>1) ? (b>2) ? A : B : c
```


### 6-3、falsy 比較
在JS，有以下的情況會回傳false值

|值|描述|
|:-:|:-:|
|false|關鍵字false|
|0|數字0|
|-0|負數得數字0|
|0n|大整數的0(The BigInt zero )|
|"", '', ``|空字串|
|null|沒有任何的值|
|undefined|尚未宣告的值|
|NaN|不是一個數字|
|document.all|Objects are falsy if and only if they have the "IsHTMLDDA" internal slot.|


[falsy表](https://developer.mozilla.org/en-US/docs/Glossary/Falsy)



### 6-4、switch - if else另外一種寫法


```markdown
> let gender = 0
> switch(gender) {
>     case 1:
>         console.log(1);
>         break;
> 
>     case 2:
>         console.log(2);
>         break
> 
>     default:
>         console.log(3);         # 3  -> 因為gender值為0，因此印出預設值
>         break
> }
```


### 6-5、for迴圈 


**for(起始值; 滿足條件; 每回合做的事情)**
```markdown
> for (var a = 0; a < 10; a = a + 1) {  
>    console.log(a);                    # 0~9  -> 印出0~9的數字  
> }  
```

情境題 -> 今天想印出 0~100 的所有奇數  
```md  
> for (let n = 0; n<=100; n++) {  
>     if (n % 2 !== 0) {  
>         console.log(n);               # 0~100 的奇數 -> 先用for迴圈設定0~100  
>     }                                   再用if判斷這些數字中哪些是奇數  
> }  
>  
```


### 6-5、while 迴圈

如果今天while裡面的值是true，就會一直執行，如果是false，就會停止

```markdown
> while (true) {                       # 如果()中為true，就一直做下去  
>     console.log("......")  
> }  
```

情境題 -> 今天想印出 0~100 的所有奇數  
```md
> let n = 0  
>  while (n<=100) {             
>    if (n % 2 !== 0) {  
>        console.log(n);  
>    }  
>    n++  
>  }
```

情境題 -> 印出**你好5、你好3、你好1**
```md
> let n = 5
> 
> while (n >= 0) {
>     if (n%2 !== 0) {
>        console.log(`你好${n}`);
>    }
>    n--
>}
>
```



7、prompt使用
------

使用prompt("請輸入......")，可以讓瀏覽器跳出輸入框，使用者可以輸入值在這個框中並進行流程判斷    


情境題 -> 判斷是平年還是閏年
```markdown
> 被4整除 閏年   被100整除 平年   被400整除 閏年
>        平年            閏年            平年

> let year = Number(prompt("請輸入年份"));
> 
> if (year % 4 === 0 && year % 100 !== 0 || year % 400 === 0 ) {
>     console.log("閏年");
> } else {
>     console.log("平年");
> }
```



