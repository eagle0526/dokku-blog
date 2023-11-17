---
title: ROR/JavaScript(06)
sidebar_label: "DAY16. [ASTROCamp] JavaScript-6"
description: JavaScript
last_update:
  date: 2022-10-27
keywords:
  - 5xRuby
  - JavaScript
sidebar_position: 17
---



1、事件流
------

### 1-1、JS的事件留有三階段
  
如果今天有兩層結構，會先往下穿過外層到內層，再從內層到外層    
(1) 第一階段 -> 捕獲階段 (capturing)    
(2) 第二階段 -> targeting       
(3) 第三階段 -> 冒泡階段 (bubbling)     
  
  
#### 1-1-2、監聽器的第三個參數
事件監聽器的第三個參數，預設值是false  
(1) 第三個參數，如果是True，就會是capturing階段    
(2) 第三個參數，如果是false，就會是bubbling階段  

Ps. 這個東西做遊戲會用到，像是卡牌遊戲   


### 1-2、實際範例

我現在做了三層的div  
(1) card1最外層  
(2) card2中間層  
(3) card3最內層   
```html
<div class="card1">我在最外層
    <div class="card2">我是中層
        <div class="card3">我是內層</div>
    </div>
</div>
```

並多做三個監聽器，點擊到div的時候，會印出被點擊到的區塊
```md
> const card1 = document.querySelector(".card1")
> const card2 = document.querySelector(".card2")
> const card3 = document.querySelector(".card3")
> 
> 
> card1.addEventListener("click", (e)=>{
>     console.log("card1");
> })
> card2.addEventListener("click", (e)=>{
>     console.log("card2");
> })
> card3.addEventListener("click", (e)=>{
>     console.log("card3");
> })
```

這樣假如我們點擊三個div的重疊區塊，印出card的順序是什麼呢？
```md
> card3    -> 第一個被印出
> card2    -> 第二個被印出
> card1    -> 第三個被印出

> 會這樣印出來的原因就是因為，事件監聽器的第三個參數，預設值是false()，也就是在冒泡階段才會被觸發
> Ps. card3被包在最內層，在空間中就是最下層
```

那假設我今天這樣改(把card1的監聽階段改成true)，印出來會變怎麼樣？
```md
> card1.addEventListener("click", (e)=>{
>     console.log("card1");
> }, true)

> card1    -> 第一個被印出
> card3    -> 第二個被印出
> card2    -> 第三個被印出

> 原因就是因為， card1改成在捕捉階段就觸發，所以card1在事件流肛下去就碰到card1
```

### 1-3、Event.stopPropagation()

#### 1-3-1、應用範例一
假設我今天想要點到3者共同的區塊，但是只想印出最上層的，那要怎麼辦？     
使用stopPropagation()，停止事件流，可以辦到     

```md
> card1.addEventListener("click", (e)=>{
>     e.stopPropagation()                     -> 1. 把事件流停止在card1
>     console.log("card1");
> }, true)                                    -> 2. card1改成補捉階段碰到
> card2.addEventListener("click", (e)=>{
>     console.log("card2");
> })
> card3.addEventListener("click", (e)=>{
>     console.log("card3");
> })
```

這樣就只會印出card1了   
原因是因為，首先我們設定card1的事件流是補多階段碰到，接著設定碰到card1就停止    
```md
> card1
```

#### 1-3-2、應用範例二
再假設，碰到三者共同區塊，但是我想印出的順序是card3 -> card2，這樣怎麼設定
```md
> card1.addEventListener("click", (e)=>{
>     // e.stopPropagation()
>     console.log("card1");
> })
> card2.addEventListener("click", (e)=>{
>     e.stopPropagation()                     -> 直接把事件流停在二就好
>     console.log("card2");
> })
> card3.addEventListener("click", (e)=>{
>     console.log("card3");
> })
```

這樣就可以照上面想印出的順序印出來了    
原因是原本事件流的順序就是冒泡，所以只要停在card2，就可以完成   
```md
> card3
> card2
```

### 1-4、target階段

如果今天對監聽器最外層下e.target，並點擊三者共通處，會發生什麼事？
```md
> card1.addEventListener("click", (e)=>{
>     console.log("card1");
>     console.log(e.target);              -> 在第一層設定e.target
> })
> card2.addEventListener("click", (e)=>{
>     
>     console.log("card2");
> })
> card3.addEventListener("click", (e)=>{
>     console.log("card3");
> })
```

```md
> <div class="card3">我是內層</div>

> 會發現明明是設定在card1上，卻印出card3的元素，原因是因為
> 他會印出 **target層** 前一層的target
> 以這個例子來說，離target層最近的元素是card3
```

### 1-5、currentTarget
那要怎麼解決摸到哪一個元素，就印出該元素呢？
```md
> card1.addEventListener("click", (e)=>{
>     console.log("card1");
>     console.log(e.currentTarget);              -> 在第一層設定e.currentTarget
> })

> 這樣設定，就可以只印出這一層的target
```

:::tip
currentTarget // 註冊事件的那傢伙，永遠不會因為事件流的而改變  
:::



2、Lexical Scope(語彙範疇)
------

代表著區塊間的包裹關係，被包裹在內層的區塊可以保護自己的變數不被外層取用，相反的外層區塊的變數還是可以被內層區塊使用

(1) Scope Chain -> 內部找不到變數，就往外找     
(2) Lexical Scope -> Scope跟他寫在哪裡有關      

下面就是一個Lexical Scope的狀況，外面宣告的變數，不管在FC還是外層都可以抓到     
反而在FC裡面宣導的變數，只有在FC裡面抓得到，外面反而抓不到   
```md
var outer = 'From Outer';
function myFunction() {
  var inner = 'From Inner';
  console.log(outer);				// "From Outer"
  console.log(inner);				// "From Inner"
}
console.log(outer);					// "From Outer"
console.log(inner);					// Uncaught ReferenceError: inner is not defined
```




3、'use strict' 嚴格模式
------

JavaScript 不斷的演進下，許多不嚴謹的寫法都應該逐漸被修正，但哪些是需要修正的字詞呢!?   
'use strict' 則是新加入的標準，目的是為了讓編寫「具穩定性的 JavaScript 更容易」  
在不穩定的語法或妨礙最佳化的語意都會跳出警告，讓開發者避開這些寫法。    

而在傳統的瀏覽器下 'use strict' 僅會被視為沒有用處的字串，所以不會對舊有的瀏覽器產生影響。  
```md
> function hi() {
>     'use strict'         -> 加上這個，就會對此函數有嚴格的標準
>     a = 2                   如果沒有宣告，就不會自動在window生成變數
>     console.log(2)
> }
> 
> hi()
```



### 3-1、加上嚴格模式後，有哪些事情不能做？

(1) 直接定義未宣告變數
```md
> myAge = 20
```
(2) 使用 delete 刪除變數或函式  
```md
> let x = 'good';
> delete x;
> function name() { console.log('hi'); }
> delete name;
```

(3) 重複變數    
```md
> function a(age, age) { console.log('hi');}
```

(4) 使用8進位值   
```md
> let num = 010
```

(5) 使用 with   
```md
> with (Math){x = cos(2)};
```

(6) eval、arguments 不能當作變數名稱    
```md
> const eval = 10;
> const arguments = 20;
```

(7) 以下這幾種保留字也不能當作變數名稱
```md
> implements、interface、let、package、private、protected、public、static、yield
```

(8) this禁止指向全域
若沒有使用 strict mode，以下這個 code block 會印出 window 這個全域環境，若啟用了嚴格模式，則會返回 undefined
```md
> function hi() { 
>   console.log(this); }
> hi();                         # undefined
```


4、物件導向
------

:::tip
物件導向的目的，用比較容易理解的方式，來整理你的程式碼，並把那些fc放到對的地方    
:::  


### 4-1、生成一個物件

下面是一個hero物件  
```markdown
> const hero = {
>     name: "kk",
>     power: 100,
>     attack: function() {
>         console.log("attack!!!)
>     }
>  }

> 呼叫hero的attack fc
> hero.attack();                    # attack!
```
  
:::danger
但是假如今天要是在做遊戲，要做一個小兵模組，總不會每一個小兵都像上面那樣一個一個物件生成，這樣太慢        
:::
  
  
### 4-2、建立一個物件模組

因此我們來建立一個英雄模組，並且這個模組要傳兩個參數進去(客製化英雄的名字、力量)
```markdown
> function heroCreator(name, power) {
>     const hero = {
>         name: name,                  -> 把函式的name傳給物件
>         power: power,                          power
>         attack: function() {
>             console.log("attack!!!)
>         }
>     }
>     return hero;
> }
```

這樣就可以直接創造出一個英雄(這樣寫，以後想要生成一個英雄，只要heroCreator(引數, 引數)就好)
```md
> const h1 = heroCreator("kk", 100)
> h1.attack();
```


:::tip
但是像上面這樣寫會遇到一個狀況，假設現在有三個新英雄(h1, h2, h3)，都用heroCreator生成，這樣三個新物件，  
都會有同樣的attack函式，因為記憶體存量的關係，如果我們把這個fc抽出來，可以大大的減少記憶體消耗。   
ex.以遊戲子彈為例，每一個子彈都是一個物件，如果每一個子彈都獨立出一個fc，這樣消耗太多記憶體
:::



5、Object_create(proto) - 參數可加可不加
------


下面的創造物件的寫法跟上面的寫法的結果一樣

```md
> function heroCreator(name, power) {
> 
>     const hero = Object.create(null)     -> 先做一個空的物件
>     hero.name = name;                    -> 屬性一個一個加上去
>     hero.power = power;                  -> 屬性一個一個加上去
>     hero.attack = function() {           -> 屬性一個一個加上去  
>         console.log("attack!")
>     }
>     return hero;
> }

> const h1 = heroCreator("kk", 100)
> console.log(h1)                          -> 這個物件印出來，也會有name, power, fc
```


:::tip
Object_create神奇的地方就在於，如果今天這樣寫       
Object.create(null) -> 這個null位置的參數，他是一個原型(prototype)，他會根據原型去打造一個東西出來  
:::




6、原型打造 prototype 
------

(1) 原型打造的意思是，根據某個東西，打造出相似的東西出來    
(2) 如果今天Object.create(null)，是null，代表我沒有要參考任何原型   


今天定義一個物件，裡面有兩個fc  
```md
> const actions =  {
>     attack: function() {
>         console.log("attack!!!!!!")
>     },
>     sleep: function() {
>         console.log("zzzzzzzzzZZZ")
>     }
> }
```

然後我在剛剛那個heroCreator的Object.create把actions傳進去  
這代表的意思是，我要用actions物件當作我的原型，去打造一個新的原型出來  
```md
> function heroCreator(name, power) {
> 
>     const hero = Object.create(actions)      -> 繼承actions物件的功能      
>     hero.name = name;
>     hero.power = power;
>     hero.attack = function() {
>         console.log("attack!")
>     }
> 
>  }
>     return hero;
> 
> 這樣我們會印出什麼，會發現h1印出來多了一個 Prototype 的東西，而fc就塞在那裡面
> const h1 = heroCreator("kk", 100)
> console.log(h1)                          -> {name: "kk", power: 100, [[Prototype]]-> 函式在這裡面}

> 因為函式在Prototype的關係，所以  
> h1.attack()  
> h1.sleep()  
> 這樣寫兩個一樣都可以動，這個新物件有繼承actions的函式 
```

7、__proto__
------

:::tip
當我要找某個物件的屬性時，JS會試著先找物件本身，如果物件本身沒有跟屬性，會循著另一個管道去找該屬性，這個管道就是__proto__     
所有的物件都有很特別的屬性，叫做 **__proto__**  
**__proto__**這個特性是，他要找東西的時候，會一直沿著物件去尋找，直到找不到東西為止  
:::


### 7-1、原型鏈 - hero.__proto__.__proto__

原型鏈的優點在哪，就是可以把共通的fc放到上層，要用的時候，再去上層拿取就好，可以大大減輕物件的大小

```md
> 假設我今天創造一個新的物件hero
> 然後再我要找hero物件的某個屬性
> ex. ability屬性
> 
> 這樣的話我們可以在console打上
> hero.ability
> 這樣的話就代表我想要找h1物件的屬性
> 
> 那實際上程式是怎麼去找這個ability的
> 是這樣 
> hero.__proto__
> 這個會去找hero前一層還有沒有屬性
> 如果往前一層還沒找到，但是沒有印出錯誤訊息
> 就再往前一層找
> hero.__proto__.__proto__
> 直到印出null為止
> 
> 這一直_proto_找尋屬性的行為，就是叫做原型鏈
```

:::danger
這邊我一直提到上層，其實不太精確，因為對於原型鏈來說，其實是下一個鏈結 - 下一個鏈結   
:::



```md
> 所以如果我今天這樣打，會印出true，因為h1往上一層找，可以找到actions物件
> code在原型打造那邊

> h1.__proto__ === actions    # true
```




### 7-2、原型鏈實際範例

你現在寫一個 
```md
> const list = [1,2,3,4]
```

我們想對這個list寫一個map方法，那這個map放在哪裡呢?就放在陣列的上一層
```md
> list.__proto__ -> 可以找到map的寫法
```



8、new新物件
------

另外一種創造物件的方法，如果今天有在前方寫給new，會直接在函式裡面給一個{}空物件     
  
這個new會幫我們做幾件事情   
(1) 產生一個空的物件    
(2) 會有一個this指向空物件    
(3) {}.__proto__ = function.prototype   
(4) 自動return物件    


```md
> function heroCreator (name, power) {
>     // this -> {}       -> this會指向new出來的空物件
>     this.name = name;
>     this.power = power
>     // return this      -> 這個this會順便幫你回傳值，不用自己寫
> }
> const h1 = **new** heroCreator("kk", 100)   -> 這邊加一個new

> 這樣我們直接印就有結果了
> console.log(h1)                # heroCreator {name: "kk", power: 100}
```

:::tip
如果今天上面那一串，在創造新物件的時候沒有寫 new 的話，會印出 undefined  
:::



### 8-1、new繼承actions的方法

不過目前new這個方式，會有一個問題，沒有繼承actions的功能  
如果使用new寫法，想要把actions傳進去，要怎麼寫呢  

```md
> function heroCreator (name, power) {       
>     // {}.__proto__ = heroCreator.prototype  -> new 物件的時候還會做這一件事
>     // this -> {}
>
>     this.name = name;
>     this.power = power
>     // return this      -> 這個this會順便幫你回傳值，不用自己寫
> }
> const h1 = **new** heroCreator("kk", 100)   -> 這邊加一個new
```

#### 8-1-1、下面這一句的意思就是，把一個空物件指向一個空物件
```md
> {}.__proto__ = heroCreator.prototype 
> 
> {}.__proto__ =  {} 空物件
> heroCreator.prototype = {} 空物件
> 
> ex. 當有人有去找__proto__的時候，請去找heroCreator這個人的prototype
```



:::tip
所有物件都有 __proto__ 屬性  
所有 fn 都有 prototype 屬性、__proto__屬性  => 所有 fc 的 prototype 都是空物件
(只有fn同時都有兩個屬性)  
:::



### 8-2、幫物件增加新功能
因此如果我今天這樣寫，可以幫新創的heroCreator物件新增功能
```md
> heroCreator.prototype.aaa = "123"
> h1.aaa                      # 123
```


### 8-3、實際案例 - new繼承某個物件的函式

```md
> function heroCreator (name, power) {       
>
>     this.name = name;
>     this.power = power
>     // return this      
> }

> heroCreator.prototype = actions               -> 直接在這邊加上想要繼承的物件，就可以成功
> const h1 = new heroCreator("kk", 100)   

> h1.attack()                   # attack!!!     -> 新物件可以印出actions的函式
```



### 8-4、mdn上Array.prototype.map()的由來
```md
> const a = new Array()             -> 用 new 創造一個變數a陣列
> a.__proto__ === Array.prototype   # true
> a.map === Array.prototype.map     # true
```



### 8-5、實際更改Array功能
像下面這樣寫，就可以幫助所有陣列增加.hello功能
```md
> Array.prototype.hello = function() {
>     console.log("hi")
> }
```


9、Class 新物件
------

因為其他的程式語言是用class的寫法新增新物件，因此JavaScript也有這種寫法(語法糖)

```md
> class heroCreator {
>     constructor(name, power) {    -> constructor 固定寫法
>         this.name = name;
>         this.power = power
>     }
> 
>     attack() {
>         console.log("attack!")
>     }
> }
> 
> const h1 = new heroCreator("kk", 100)  
> console.log(h1)                           -> 結果跟剛剛上面一樣
```

:::tip
如果今天用Class寫法，創新物件的時候沒有給new，系統會直接報錯 -> TypeError  
如果今天是用new寫法，創新物件的時候沒有給new，系統會說 -> undefined   
:::  



10、物件繼承
------

物件導向真正的意義，把共通的行為放到最裡面層，想要用的時候在叫出來用，比較好管理

```md
> class Animal {                                -> 設定一個上層動物類型
>     constructor(name, power) {                -> 這個constructor是固定寫法，類似ruby的initialize
>         this.name = name;
>         this.power = power;
>     }
> }
> 
> 
> class heroCreator **extends** Animals {       -> 類別英雄繼承animals的類別
>     attack() {
>         console.log("attack!!!!!!")
>     }
> }
> 
> 
> const h1 = new heroCreator("kk", 100)
> 
> console.log(h1)         # heroCreator {name: "kk", power: 100} -> 一樣有相同的屬性
> 
> h1.__proto__            # Animal {constructor} -> proto會指向上一層
```




11、面試會考的題目
------

(1) Q. All objects have prototype?      
Ans. false  
詳解：是所有function都有prototype，所有物件都有的是__proto__    

(2) 箭頭函式沒有prototype

(3) 使用new實體時，在類別裡面呼叫return，會發生啥事

```md
> function Cat(name) {
>     this.name = name
>     return 123             -> 如果今天刻意給實體 return  (正常人不會在這邊return)
> }
> 
> const c = new Cat("cc")    -> 實際上會忽略上面傳下來的123，一樣是傳Cat下來
> console.log(c)             # 印出 Cat { name: 'cc' }
```




12、分號到底要不要加
------

### 12-1、ASI(Auto Semicolon Insertion)

ASI是JavaScript自動插入分號的機制，當JavaScript語句沒有加上分號時，則會受到自動插入分號 (ASI) 規則影響

### 12-2、ASI觸發，自動增加分號
以下幾種規則介紹
(1) 當執行 continue、break、return... 語句，語句後會自動加上分號
(2) 當 JavaScript 語句'後一行'接到'前一行'會發生語法錯誤時，會自動加上分號

第二點有點難懂，舉個例子，假設今天寫了一個if/else的判斷
```md
> if (1>2) a=1
> else a=2
> console.log(a) // 2
```

實際運行下去會是這樣
```md
> if (1>2) a=1;
> else a=2;
> console.log(a); // 2
```

### 12-3、ASI沒有觸發
不過有時候不會觸發ASI，所以會導致程式發生錯誤
(1) 當新的一行是 (、[、/ 開始     
(2) 新的一行以 +、-、*、% 開始  
(3) 新的一行以 ,、. 開始    


下面這個例子，就是因為下一行是一個中括號[]的話，前面一行code的尾端他就不為自動幫你加分號，所以會發生錯誤
```md
> console.log(123)
> [1,2,3].map((x) => {})   -> 這樣兩行都不加分號的話，會無法執行
```

### 12-4、return想要換行怎麼寫
如果要換行，可以加一個小括號
```md
> return (
>     1+2
> )
```


13、this介紹
------

:::tip
this是代名詞  
:::  

### 13-1、誰呼叫，誰就是this
```md
> const hero = {
>     hi: function() {
>         console.log(this)   -> 這個this就是hero
>     }
> }
> hero.hi()
```



另一種狀況
```md
> function hi() {
>     function hey() {
>         const hero = {
>             name: "kk"
>             action: function () {
>                 console.log(this.name)   -> 這個this 是 hero
>             }
>         }
>         
>     }
>     hero.action()
> }
> hi()
```

### 13-2、沒人呼叫
```md
> function hi() {
>     console.log(this)       -> 這個this會變成全域物件
> }
> 
> hi()
```


另一種狀況，就算this被包在第二層裡面，他還是算沒人呼叫他  
因為this注重的是，在哪發動，並且發動的當下，有沒有人呼叫他  
```md
> function hi() {
>     function hey() {
>         console.log(this)       -> 這個this也是全域物件
>     }
>     hey()                        
> }
> 
> hi()
```

### 13-3、箭頭函式沒有自己的this 

範例一
```md
> const hey = {} => {}
```

範例二
```md
> const btn = document.querySelector("#btn")
> 
> btn.addEventListener("click", ()=>{           
>     console.log(this)                   -> 這個this因為箭頭函式，所以全域物件
> })
```



### 14-4、是否有使用 new

如果今天想要創造新物件，但是沒有使用new
```md
> function hi(age) {    
>     this.age = age      -> this會直接在全域產生一個 age 變數
> }
> 
> const h = hi(18)        -> 今天沒有使用 new
```
```md
> console.log(h)         # undefined   ---> 如果今天有加new "會印出hi { age: 18 }"
> console.log(age)       # 18 

> **不過這很可怕，一定要記得加上new
```




### 15-5、是否有使用 apply, call, bind

#### 15-5-1、用function來看情況
```md
> function hi() {
>     console.log(this)       -> 直接呼叫hi的情況下，this是全域
> }
> 
> hi()     # window(全域)      
```

```md
> function hi() {
>     console.log(this)       -> 用call呼叫，這個會變成call裡面的東西
> }
> 
> hi.call(88888)     # [Number: 888]     
```





#### 15-5-2、用物件來看情況，先給一個物件
```md
> const hero = {
>     name: "kk"
>     action: function() {
>         console.log(this.name);    
>     }
> }
> #### 呼叫物件中的函式
> hero.action()                     -> 這會印出 kk
```


#### 15-5-3、用.call來呼叫
```md
> const hero = {
>     name: "kk"
>     action: function() {
>         console.log(this.name);    -> 這個this會變成 cc 
>     }
> }
> 
> const cc = { name: "cc" }
> hero.action.call(cc)   -> 會印出 "cc"
```


#### 15-5-4、.call呼叫方法可以帶參數，回傳進去呼叫的物件裡面
```md
> const hero = {
>     name: "kk"
>     action: function(n, m) {
>         console.log(this.name, n, m);    ->  印出 cc, 1, 2
>     }
> }
> 
> const cc = {name: "cc"}
> hero.action.call(cc, 1, 2)   -> 這1, 2 會傳上去function裡面
```

:::tip
**call 跟 apply 的差異**  
call後面參數是接多個數字  
apply後面是接陣列  
:::  



#### 15-5-5、call物件實際案例

```md
> 先生成一個戰士物件
>
> const hero = {
>     hp: 100,
>     mp: 20,
>     attack: function() {
>         console.log("attack!!!!!")
>     }
> }
>
> 再生成一個法師物件，多一個補血技能，用call方法可以幫戰士補血
> const mage = {
>     hp: 70,
>     mp: 80,
>     attack: function() {
>         console.log("attack!!!!!")
>     }
>     heal: function() {
>         this.hp += 30  -> this 會依照下面的方法更改對象(call方法)
>     }
> }
> 
> mage.heal.call(hero)   -> 這樣寫可以幫英雄補血，因為call可以改變傳進來技能的指向
```




:::tip      
**續 this**   
這part在10/28的影片       
:::


#### 15-5-6、bind跟call、apply的差異

bind跟apply、call的差異是，bind會比較晚觸發(你去主動執行的時候才會觸發)

:::tip
#### bind方法  
該bind()方法創建一個新函式，該函式被呼叫時，會將 this 關鍵字設為給定的參數，並在呼叫時，帶有提供之前給定順序的參數。
  
#### 回傳值  
他會回傳一個新的 fc，並且會把傳進去的東西東做this的初始值
:::



#### 15-5-6、bind舉個例子更好懂 - fc舉例

先給一個fc，印出this    
```md
> function hi() {
>    console.log(this)   -> 這個this是全域物件
> }
> hi()
```

創建一個新變數，並用hi.bind()接著會發生什麼事   
```md
> function hi() {
>     console.log(this)   # [Number: 123]    -> 這個this是bind傳進來的數字 
> }
> 
> const newHi = hi.bind(123)    -> 這個 newHi 因為 bind 的關係，是一個fc
```

也因為newHi是一個新的fc，所以在主動呼叫之前，他不會發動
```md
> newHi()   -> 主動發動
```

#### 15-5-7、bind舉個例子更好懂 - 物件舉例

剛剛只有fc和123可能不太好懂，我們今天加入物件試試  
  
創建一個ironMan物件
```md
> const ironMan = {name: "ironMan"}
```

把這個ironMan換成剛剛bind裡面的123
```md
> const newHi = hi.bind(ironMan)
```

這時候呼叫newHi，就會跑出ironMan物件了
```md
> newHi()         # {name: "ironMan"}
> 
> ps. 會印出來是因為前面的hi()，我有寫console.log(this)喔！不要忘記
```
  
#### 15-5-8、用bind的好處在哪？

**不過目前還看不出好處在哪對不對，我把整段code更改一下**
```md
> 1. 一樣給一個fc，然後帶兩個參數
> function hello(n, m) {
>     console.log(this)    # {name: "ironMan"}  -> 在fc執行前，物件被修改了
>     console.log(n, m)    # (1, 2)             -> fc 可以帶參數進來
> }
>   
> 2. 一樣有一個ironMan物件
> const ironMan = {name: "ironMan"}
>   
> 3. 一樣用bind做出一個新物件，並帶入ironMan
> const newHello = hello.bind(ironMan)
>   
> 4. **不一樣的來了，我們在執行前，先改變 ironMan 的值**
> ironMan.name = "ironHeart"
>   
> 5. 執行，並帶一些參數給newHello
> newHello(1, 2)
```
  
:::tip
**bind的好處在哪**  
bind 會比較晚觸發(你去主動執行的時候才會觸發)  
在觸發前，可以更改一些值，改完後在觸發  
:::

  
  
### 15-6、是否為嚴格模式 use strict
不是嚴格模式的情況下，在 fc 裡面使用this，會變成全域變數，嚴格模式下會變成undefined
```md
> function hi() {
>     `use strict`
>     console.log(this) -> undefined
> }
```
  
### 15-7、this 觸發先後順序  
:::tip
1. 是否為嚴格模式 use strict  
2. 是否有使用 new  
3. 是否有使用 apply, call, bind  
4. () => {} 沒有自己的 this  (箭頭函式)
5. 誰呼叫，誰就是 this  
6. 沒人呼叫，this -> 全域物件  
  
嚴格模式 > new > apply、call、bind > 箭頭函式 > 誰呼叫誰就是this > 沒人呼叫為全域物件    
:::



### 15-8、對箭頭函式new物件

箭頭函式沒有this，那我們對一個箭頭函式 new 物件會發生什麼事
```markdown
> 會報錯 -> not a constructor
> 
> const heroCreator = () => {}
> 
> const h1 = new heroCreator()
> console.log(h1)
```


16、箭頭函式缺少的東西
------
:::info
1. 缺少this  
2. 缺少一般fc有的arguments  
  
一般函式有一個隱藏 的arguments(引數)  
ps. 可以試試做一個正常的fc -> console.log(arguments)  
:::


