---
title: ROR/JavaScript(07)
sidebar_label: "DAY17. [ASTROCamp] JavaScript-7"
description: JavaScript
last_update:
  date: 2022-10-28
keywords:
  - 5xRuby
  - JavaScript
sidebar_position: 18
---




1、Closure 閉包
------

Closure是什麼？我們以setTimeout舉例  
Ps. [完整的 code 可以看closure那邊文章](https://eagle0526.github.io/javascript/2022-09-27-Closure.html)  


假設今天想要印出 1 -> 2 -> 3 (每過一秒印出 1++)         
但是發現用var宣告變數後，結果跟預想的不太一樣   
```md
> for(var i = 0; i < 3; i++) {
>     setTimeout(()=>{
>         console.log(i)        # 3 -> 3 -> 3 (每隔一秒印出一個3)
>     }, 1000 * i)
> }
```


:::tip 把上面 for 迴圈中的 var 改成 let 做到 1 -> 2 -> 3 (每隔一秒印出一個數字)
為什麼改成 let 就可以執行，是因為閉包的概念  
var的話就算離開for迴圈還可以存活  
所以最後 i 就會讀取 迴圈最後結束的值  

如果改成let的話，離開迴圈就無法存活，所以會有一個閉包把這個i一起包進webAPI  
之後 setTimeout 的 i 就會依照閉包中的i給值  
:::



### 1-1、function都會有Closure的特性
(1) JS再執行的過程中，會去考慮非同步相關的函式 or function，是否有用for迴圈包住，用let宣告的話，裡面的 i 會持續變動     
(2) let原本只會存活在for回圈裡面，但是被setTimeout一起帶去webAPI裡面，所以等 stack 裡面全部執行完後， let才會依然存活   
(3) 他會去捕捉周圍的環境變數，但是只會去抓自己有用到的東西  


### 1-2、Closure實際範例    

先做一個fc，裡面包一個變數、fc
```md
> function hi() {
>     let a = 1
>     function hey() {
>         console.log(a)      # 1   -> 會印出1
>     }
>     hey()
> }
> hi()

> Ps. let a 在 fc 存活，所以印出來很正常
```



現在我們換掉一些code，並在 fc 外面試著把 a 印出來
```md
> function hi() {
>     let a = 1
>     function hey() {
>         console.log(a)  4. 印出 1
>     }
>     return hey          1. 我們把fc hey傳出去(這時候封包會一起把let a傳出去)    
> }
> 
> const newHi = hi()      2. 並用一個新變數接住，這個newHi是一個fc喔！！
> newHi()                 3. 呼叫newHi

> Ps. 正常的情況下，let應該會被限制在function才對，不過因為封包的關係
>     return hey 的時候，會做一個閉包把要用的參數傳出去
```




2、遞迴函式
------

### 2-1、斐波那契數列

以斐波那契數列來示範遞迴函式    
斐波那契數列 = 前兩個位數的數字相加等於新的位數數字

```md
> function fib(n) {
>     if( n === 0 ) {               -> 到0停止
>         return 0
>     }
> 
>     if (n===1) {
>         return 1
>     }
> 
>     return fib(n-1) + fib(n-2)    -> 可以自己呼叫自己的原因，是因為有終止條件
> }
> 
> console.log(fib(10))
```

:::info
糖果題今天如果寫得太複雜，可以試試用遞迴方式來寫  
:::



3、IIFE = Immediately Invoked Function Expression (念 " 一 ㄈㄧ " )
------
寫套件的時候很常用到，因為不想讓自己的套件的fc被影響到
```md
> 用一個小括弧把fc包住，並在最後用()呼叫此fc
> (function () {
>     console.log("123")
> })**()**;              -> 最後記得用()呼叫，可以帶參數進去喔
> 
> 這種寫法的目的，是因為只能使用一次，不會漏到外面
```

:::tip
使用立即函式的好處，除了可以立即執行程式碼，省略多餘的呼叫，還可以用來避免汙染全域執行環境的東西，減少開發時因相同命名相衝的bug。      
:::


4、parcel 打包檔案
------

parcel是一個上線過程的“打包器”      
如果今天太多JS、CS檔案，要在index.html連結其他檔案的時候，該怎麼辦？    

```md
> 現在有3個js檔案，我想把這三個壓成一個JS檔案 -> webpack、parcel可以做到
> <script src="app.js"></script>
> <script src="test.js"></script>
> <script src="house.js"></script>
```

:::tip
市面上有很多打包工具，可以把一大堆的JS、css壓縮成一包小東西  
npm -> 前端世界專門安裝套件的東西 (npm有點慢)  
yarn -> facebook做的，誕生原因是因為 npm 有點慢  
( yarn 比較快的原因，是因為yarn可以平行下載，一次下載多個檔案 )  
:::


### 4-1、打包器實際執行

一、在專案終端機輸入    

```md
> yarn init
```

會誕生一個json檔案 => package.json
```md
> {
>   "name": "day17",
>   "version": "1.0.0",
>   "main": "index.js",
>   "license": "MIT",        -> 此專案的授權來源
> }
```
:::tip
(1) MIT是麻省理工  
(2) BSD是柏克萊  
這兩間授權的話，基本上都可以免費使用  
(3) GPL也是一種開源的受援，不過你今天用GPL的license寫一份專案，你那一份專案也要開源  
:::  



二、接著在json檔上的scripts加一些code   

```md
> scripts -> 這個很常是前端工程師，加入自己很常用的語法

> "scripts": {
>     "hi": "echo 123",
>     "hey": "echo 456"
> }
> 
> 呼叫hey預設方法
> yarn run hey           # 456 
```


三、安裝 parcel  

輸入後，會直接下載一大堆檔案，包在一個資料夾(node_modules)裡面   
```md
> yarn add --dev parcel 
```


並會在json檔生成一個多幾行程式   
```md
> "devDependencies": {
> "parcel": "^2.7.0"
> }

> dev是開發的意思 - devDependencies
```


:::tip
如果不小心刪掉node_modules  
再打上yarn install就會回覆   
:::


四、接著到.json檔的scripts裡面，新增一段    
下面這一行的指令，就跟 rails s 一樣，就是啟動server
```md
> "scripts": {
>     "dev": "yarn parcel index.html"  ->  加在這邊的關係，是因為我之後想重複只用 dev 這個指令
> },                                       要注意那個index.html，如果今天沒有把script包在index裡面
>                                          就要yarn parcel src="this_test.js"
```

五、接著直接呼叫

```md
> yarn run dev
```

如果今天沒有加上dev那一條，可以直接這樣
```md
> yarn run "yarn parcel index.html"
```

六、最後把type="module" 寫進script裡面

```md
> <script src="app.js" defer type="module"></script>
```

這樣寫後，開啟環境，並開後台會發現 scr: "一長串的姓名.js"  
以前的時代，瀏覽器會進行快取，所以index 有為每個檔案取名一長串得符號，如果內容物改變的話，名字究會改變   




5、JS import/export
------
es modules 如果今天想要引用別人的模組或是自己寫的模組

### 5-1、export模組輸出

#### 5-1-1、方法一、輸入給別人用
```md
> math.js

> export function add(a, b) {
>     return a + b
> }
```

#### 5-1-2、方法二、另外一種export寫法
```md
> math.js

> function add(a, b) {
>     return a + b
> }
> export { add, minus, multiple } -> 也可以在最後寫export，把所有想要傳出去的傳出去
```


### 5-2、import模組輸入

```md
> app.js

> import{ add, PI } from "./math.js"  -> 這個add套件是寫在math.js裡面的
>                                     -> 如果今天有多個要import，可以,隔開就好
> console.log(add(1, 5))     # 6
```

:::info
如果今天沒有打包模組，系統會看不懂import、export  
:::


### 5-3、export default意思 
export的時候，default是什麼意思？   
default其實就是一個預設匯出的寫法！   
```md
> export default add -> export default 只會有一個   
> import X from "./math.js"   -> 輸入預設的模組，可以隨便取名使用   
```

:::tip
輸入也可以全部加寫在一起  
```md
import X, { add, minus } from "./math.js"  
```
:::



### 5-4、引用別人的套件

```md
> yarn add dayjs -> 這樣會把dayjs裝在node_modules裡面
> 
> import dd from "./dayjs" -> 直接取用就可以，可以隨意取名的原因，是因為對方的fc是default
```


如果匯入同個資料夾有相同的fc，再匯入的時候，記得用as
```md
> import { add as myAdd } from "./math.js" 
> 
> function add() {
>     console.log(234)
> }
```




6、JavaScript 測試環境
------

到專案的終端機輸入以下
```md
> yarn add -dev jest
```