---
title: JavaScript/What Is Closure 什麼是閉包
sidebar_label: "5. [JavaScript] What Is Closure"
description: What Is Closure 什麼是閉包
last_update:
  date: 2022-09-27
keywords:
  - JavaScript
  - Closure
sidebar_position: 5
---


## 什麼是Closure閉包呢？


:::tip
閉包（Closure）是函式以及該函式被宣告時所在的作用域環境（lexical environment）的組合。     
from MDN   
:::


:::tip **lexical的意思**    
代表著區塊間的包裹關係，被包裹在內層的區塊可以保護自己的變數不被外層取用，相反的外層區塊的變數還是可以被內層區塊使用   
:::


:::tip **作用域的意思**   
指的正是作用域環境在程式碼指定變數時，使用 location 來決定該變數用在哪裡的事情。巢狀函式的內部函式，能訪問在該函式作用域之外的變數。   
:::



這樣有可能還是不太懂，我們直接舉個例子好了，下面這個函式有幾個特色   
(1) init() 建立了局部變數 name 與 displayName() 函式     
(2) displayName() 是個在 init() 內定義的內部函式，且只在該函式內做動。displayName() 自己並沒有局部變數  
(3) 由於displayName自己沒有局部變數，他就會向外找，找到var name這個變數     

```js
function init() {
  var name = "Mozilla";         // name 是個由 init 建立的局部變數
  function displayName() {      // displayName() 是內部函式，一個閉包
    alert(name);                // 使用了父函式宣告的變數
  }
  displayName();
}
init();
```


## 一般閉包

#### 題目 - 做出一個加法函式
```js
var addSix = createBase(6);
addSix(10); // returns 16
addSix(21); // returns 27
```

#### 解題
```js
createBase(baseNumber) {
    return function(x) {
        return x + baseNumber
    }
}

var addSix = createBase(6)
addSix(10); // returns 16
addSix(21); // returns 27
```

#### 題目詳解

(1) 這題我們定義一個帶有單一參數baseNumber，並且回傳一個新函式的createBase()函式  
(2) 本質上，createBase()是一個韓式工廠，他建立一個基本值
(3) addSix兩個都是閉包，他們共享了createBase的定義，但是保有不同的環境(兩個分別傳了不同的參數進去)





## 迴圈中建立閉包
   
### 每隔一秒印出加一的函示

假設今天我想做一個印出 1 -> 2 -> 3 (每隔1s，印出)，因此用 for迴圈 + setTimeout 做出了下面的code  

```js
for(var i = 0; i < 3; i++) {
    setTimeout(()=>{
        console.log(i)        // 3 -> 3 -> 3 (每隔一秒印出一個3)
    }, 1000 * i)
}
```

:::info
不過結果發現，每隔一秒會印出一個數字是沒錯，但是全部都印出 3，是為什麼？？    
:::



這邊要另外提到 var、let，在for迴圈設置變數的差異  

### var - function scope
用var設定一個變數，他離開for迴圈後還是存在，就像以下
```js
for(var good = 1; good < 5; good++) {
}

console.log(good);    # 5
```


### let - block scope
用let設定一個變數，他離開for迴圈後，會錯誤
```js
for(let good = 1; good < 5; good++) {
    
}
console.log(good);  // ReferenceError: good is not defined
```

:::danger
簡單說就是，var離開迴圈後還會繼續作用，而let不行，所以var的for迴圈才印得出來  
[變數提升是什麼？](https://eagle0526.github.io/javascript/2022-09-23-Hoisting.html)   
:::


看完上面 var vs. let 跑迴圈，這樣就可以說明，為什麼一開始使用var設定變數setTimeout的for迴圈，
會全部印出3了，因為var離開迴圈，它還是存在全域變數裡面，所以setTimeout從webAPI轉回來時，他接到的i就是3  



### 解決方法
這就會牽扯到 **Closure閉包** 的概念  
  
直接舉個例子比較好懂，一樣用剛剛的for回圈來講解  
```md
> 把var i = 0 改成let i = 0
> 這樣寫就可以做到我們想要的效果了 0 -> 1 -> 2

> for(let i = 0; i < 3; i++) {
>     setTimeout(()=>{
>         console.log(i)       # 0 -> 1 -> 2
>     }, 1000 * i)
> }
```

:::tip
原因就是因為**閉包**的關係，當setTimeout執行時，如果for迴圈外面沒有變數i
會順便把變數i帶進webAPI裡面(webAPI是什麼請參見[EventLoop這篇文章](https://eagle0526.github.io/javascript/2022-09-24-What-Is-Event-Loop.html))  
for迴圈跑一圈，就會包一個變數i進去webAPI，並跑一次EventLoop，接著就會在畫面上照順序印出 0 -> 1 -> 2
:::













