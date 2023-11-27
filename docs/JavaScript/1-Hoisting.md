---
title: JavaScript/Hoisting
sidebar_label: "1. [JavaScript] Hoisting"
description: Hoisting
last_update:
  date: 2022-09-23
keywords:
  - JavaScript
  - Hoisting
sidebar_position: 1
---


1、什麼是Hoisting變數提升？
------

提升的意思就是，讓變數可以在宣告之前就使用  


:::tip
它是一種釐清 JavaScript 在執行階段內文如何運行的思路（尤其是在創建和執行階段）  
然而，提升一詞可能會引起誤解：例如，提升看起來是單純地將變數和函式宣告，移動到程式的區塊頂端，然而並非如此。  
變數和函數的宣告會在編譯階段就被放入記憶體，但實際位置和程式碼中完全一樣。    
[from MDN](https://developer.mozilla.org/zh-TW/docs/Glossary/Hoisting)  
:::


### 1-1、JS分兩階段運作
 
(1)第一輪是掃瞄 - 建立期    
1A - 註冊名稱(identifier)      
1B - 進行初始化(檢查語法、建立變數)    
    
(2)第二輪式執行 - 執行期    
執行函數、賦值     


```markdown
> 情境算式
> var a = 1
> console.log(a)
>
> ----------
> 建立期         執行階段  
> var a         1A、1b
> 
> 執行期
> a = 1           2
> console.log(a)  2       
```

:::info
JS宣告變數兩階段運作  
建立期間把所有程式碼掃過，並把名稱都設定好  
執行期間才把值給出去  
:::


### 1-2、var宣告執行範例

#### 1-2-1、情境一
```markdown
> console.log(a)
> var a = 1
> console.log(a)
```

兩階段執行狀況
```md
> 第一階段
> console.log(a)     # 不執行
> var a = 1          # 執行 1A、1B -> 已初始化，但是還沒賦值
> console.log(a)     # 不執行

> 第二階段
> console.log(a)     # 執行，印出undefined
> var a = 1          # 執行，賦值
> console.log        # 執行，印出1
```


#### 1-2-2、情境二

```markdown
> var a = 1
> var a
> console.log(a) 
```

兩階段執行狀況
```md
> 第一階段
> var a = 1              # 執行 1a、1b
> var a                  # 執行 1a、1b
> console.log(a)         # 不執行

> 第二階段
> var a = 1              # 執行 2，給值變1
> var a                  # 執行 2，但是啥都沒給，所以忽略
> 
> console.log(a)         # 執行，印出a是1
``` 




### 1-3、let如何解決變數提升

let 怎麼解決變數提升 => 1b不執行    

兩階段執行狀況
```markdown
> 第一階段
> console.log(a)     # 不執行
> let a = 1          # 執行1a，不執行1b, 被一個罩子蓋住了(TDZ)
> 
> 第二階段
> console.log(a)     # 執行, 但是因為，1b沒執行，噴出referenceError(尚未初始化)
```

### 1-4、TDZ暫時死區(Temporal Dead Zone)


正常情況 let 的執行狀況
```markdown
> 第一階段
> let a = 1          # 執行，不執行1B，並給一個蓋子
> console.log(a)     # 不執行

> 第二階段
> let a = 1          # 執行，把蓋子拿掉
> console.log(a)     # 執行，印出 a = 1
```


:::tip
const/let也有變數提升，只是他們多了TDZ來防止初始化這件事  
ES6中let/const宣告的變數被賦值之前不允許使用。 
:::


### 1-5、函式宣告順序

#### 1-5-1、範例一
兩階段執行狀況
```markdown
> 第一階段
> sayHi()                                    # 第一輪不執行
> const sayNyName = function sayHi() {       # 第一輪做宣告，但是1b沒做 
>     console.log()
> }

> 第二階段
> sayHi()                                    # 第二輪呼叫變數
> const sayNyName = function sayHi() {       # 第二輪做事，印出裡面的東西
>     console.log()                  
> }
>
> BUT!!!!!!!，上面這樣會出錯，會發生initialization的問題，原因就是TDZ的問題
```

#### 1-5-2、範例二

JS執行function的順序，在function裡面，1a、1b、2都會一起做好  

兩階段執行狀況
```markdown
> 第一階段
> sayHi()                   # 不執行
> function sayHi() {        # 執行 - sayHi執行1a、1b   {}執行2 
>     console.log("hello") 
> }
 
> 第二階段
> sayHi()                   # 執行
>
> 印出 hello
```  
=> 因爲第一階段{}會執行2(執行函數賦值)，因此第二階段執行的時候，才可以成功執行  
    

#### 1-5-3、範例三
```md
> foo = "outer bar"
> 
> function bar() {
>     foo = "inside bar" 
>     console.log(`inner : ${foo}`)
> }
> bar()
> console.log(`outer : ${foo}`)
```
Ans
```md
> inner : inside bar
> outer : inside bar
```
原因詳解，今天不用變數宣告的話，會直接在window(全域)放一個foo變數，因此後宣告的foo就會直接變成inside bar


2、scope的概念
------

### 2-1、var = function scope  
var 會被 function 包住，但是無法被 {}block 包住   

### 2-2、let/const = block scope  
let/const 只要有一個大括號就可以把他包住  

Ps. 包住的意思是，變數只能在該範圍運作



### 2-3、情境題一
在 if 回圈裡面設定 const 會怎麼樣

```markdown
> var age = 20
> if (age >= 18) {
>     const message = "未成年"          # is not defined
> } else {
>     const message = "已成年"          # is not defined
> }
> 
> console.log(message) -> 因為const被{}包住，所以噴會出is not defined
```

這一題解法很簡單，只要在if前面先宣告message     
```markdown 
> var age = 20
> let message;               -> 先宣告，不要給值
> if (age >= 18) {
>     message = "未成年"
> } else {
>     message = "已成年"
> }
> 
> console.log(message) -> # 未成年，這樣就可以跑了，因為前面先宣告過
```


### 2-4、情境題二
在 function 裡面設定 var 、 let 會怎麼樣

```markdown
> function hello() {
>     var a = 1
>     let b = 2
> }
> hello()
> 
> console.log(a)    會出錯 is not defined，因為var和let都被限定在function中
> console.log(b)    會出錯 is not defined，因為var和let都被限定在function中
```



### 2-5、全域變數

瀏覽器裡面有一個全域物件叫做 window     
node 世界裡面也有一個全域物件叫做 global    
```md
> window.console.log("aaa")  
> window.alert("123")  
```

如果今天在網站上宣告一個 var ccc = 123      
這個變數ccc會存進window裡面     
    
但是，如果今天在網站上宣告一個 let ddd = 123     
這個變數ddd **不會** 存進window裡面     

:::tip
簡單的說var宣告會造成全域物件的污染   
:::


:::danger
**那如果我們直接使用賦值 -> age = 20 呢**  
假設window裡面原本沒有age這個變數     
age = 20 這個動作會直接生出一個 變數age 丟在window環境裡面    
這樣會造成很多問題，所以千萬不要輕易的宣告全域變數    
:::



3、let/const vs var
------

### 4-1、三個比較表


|特性|var|let/const|
|:-:|:-:|:-:|
|scope|function scope|block scope|
|變數宣告|可|不可|
|全域屬性|會|不會|
|變數提升|會|會(但是有TDZ壓住)|


***

別人寫的文章
```md
> 這篇文章的重點有兩個：
> 
> let const var 最主要的差異是：ES6 let 和 const 以「區塊」作為其作用域 (scope)，而 var 以「函數」作為其作用域。
> 
> 宣告 var 的時候，宣告會被提前至函式作用域的開頭，這個特性又稱為 hoisting。
> 
> 如果你還不是很熟悉這兩個概念的話，下面會詳細解釋，跟我一起看下去吧！
```
