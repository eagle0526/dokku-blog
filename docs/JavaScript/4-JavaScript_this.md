---
title: JavaScript/JavaScript_this
sidebar_label: "4. [JavaScript] JavaScript_this"
description: JavaScript_this
last_update:
  date: 2022-09-26
keywords:
  - JavaScript
  - Object
sidebar_position: 4
---

this是什麼？
------

JavaScript中的this是不是很常讓人誤解現在到底指向誰呢？  
我們現在就來看看，到底要怎麼辨別在不同地方出現的this都代表什麼意思。  

:::tip
this就是代名詞的意思  
:::
  
  
  
### this在以下幾種情況下，會有不同的意義
  
1. 是否為嚴格模式 use strict  
2. 是否有使用 new  
3. 是否有使用 apply, call, bind  
4. () => {} 沒有自己的 this  
5. 誰呼叫，誰就是 this  
6. 沒人呼叫，this -> 全域物件  
  
**重要順序為**
嚴格模式 > new > apply、call、bind > 箭頭函式 > 誰呼叫誰就是this > 沒人呼叫為全域物件  
  

接下來就為各位介紹每個情況下的this意思
  
  
一、strict 
------

### 1. 是否為嚴格模式 use strict
  
  
#### 非嚴格模式
非嚴格模式的情況下，在 fc 裡面使用this，會變成全域物件 Window

```md
> function hi() {
>     console.log(`this: ${this}`)             # this: Window
> }
> hi()
```

Ps. 這個全域物件有點抽象，我們舉個例子
```md
> 假設剛剛不下新在外面設定一個全域物件this
> 並最使用var宣告變數 

> var a = 1                 > 這個宣告方式，會在全域物件產生一個變數 a
> this.a === window.a       > # true

> 上面為true的原因，就是因為var宣告會在全域物件window塞一個變數，
> 今天this也是全域物件，所以也會塞一個var宣告的變數進去
```



#### 嚴格模式
嚴格模式下會變成undefined
  
```md
> function hi() {
>     `use strict`                             # 嚴格模式
>     console.log(`this: ${this}`)             # this: undefined
> }
> hi()
```
:::danger
非嚴格模式下，讓this變成全域物件，不好的影響就是會導致此this會變另外一個window  
:::


二、new
------

### 2. 創建新物件時，是否有使用 new  
  


#### 有使用new
  
正常new新物件，都會在產生新物件時放上一個new  
  
```md
> function HeroCreator(fullName, profess) {
>     // this -> {}                             -> 有new會有一個this指向空物件
>     this.fullName = fullName                  -> 這個this就是下面那個spider
>     this.profess = profess
> }
> 
> const spider = new HeroCreator("tom", "spider")  -> this因為new的關係，指向spider囉

> 我們把spider和內容物印出來看看
> console.log(spider);                    # HeroCreator {fullName: 'tom', profess: 'spider'}
> 
> console.log(spider.fullName);           # tom
> console.log(spider.profess);            # spider
```




#### 沒有有使用new

同樣用上面的例子，如果今天想要創造蜘蛛人的時候，但是沒有使用new會發生什麼事  
```md
> function HeroCreator(fullName, profess) {
>     XX this > {} XX                         -> 2. 沒有new，所以this沒有指向空物件
> 
>     this.fullName = fullName                -> 3. 導致這個this直接讓fullName變成全域變數
>     this.profess = profess                  -> 4. 同上
> }
> 
> let spider = HeroCreator("tom", "spider")   -> 1. 增加蜘蛛人沒給 new 方法

> 我們把spider和fullName印出來看看
> console.log(spider)           # undefined

> console.log(fullName)         # tom
> console.log(profess)          # profess
```

:::tip
所有這非常可怕，new物件時一定要記得加上new  
:::





三、apply, call, bind 
------

### 3. 是否有使用 apply, call, bind  
  
首先來展示一下call方法，大家還記得前面說的，假設今天在非嚴格狀態下，直接在fc裡面使用this  
這個this是代表什麼嗎？給大家幾秒鐘想一下~
  
1s  
2s  
3s...  
  
想起來了嗎？沒錯，this會直接變成全域物件，我在印出來一次給大家看  
```md
> function HeroCreator() {
>     console.log(this);                # Window
> }
> 
> HeroCreator()
```


### call
  
#### 範例一
那假設我們現在對上面這個英雄創造模組，使用call方法，會發生什麼事呢

```md
> function HeroCreator() {
>     console.log(this);
> }
> 
> HeroCreator.call("111")     # String {"111"}
> HeroCreator.call(222)       # Number {222}
```

有沒有發現，原本是全域物件的的this，主要作用對象換了，變成call()裡面的的東西  
  
  

#### 範例二
今天我們換一下，用物件來測試call方法會發生什麼事  
  
先給一個物件，物件中帶有英雄姓名、攻擊函式，並在攻擊fc裡面放一個this.name
```md
> const hero = {
>     name: "spider",
>     attack: function() {
>         console.log(`attack: ${this.name}`)    -> 這個會印出什麼呢 ？？？？
>     }
> }
> 
> hero.attack()                                  > 被呼叫後，this會被指向hero物件

> Ans. 印出 attack: spider
```
  
  
  
＊＊＊＊important＊＊＊＊  
那我們今天用創造一個新物件，並用call方法呢(沿用上方hero物件)  
  
在創造一個新物件  
```md
> const antMan = {
>     name: "AntMan"
> }
```
  
並用搭配剛剛的hero物件  
```md
> hero.attack.call(antMan)              -> 這個會印出什麼呢？？
```
  
因為用call的方式，this的作用對象會改為call()裡面的物件  
```md
> Ans: 印出 attack: AntMan
```
  
＊＊＊＊important＊＊＊＊



#### 範例三
call方法可以帶參數進去，讓this接到，一樣用剛剛的hero物件做延伸  

在剛剛的攻擊fc裡面，放兩個參數，並在新英雄call的時候帶引數進去
```md
> const hero = {
>     name: "spider",
>     attack: function(one, two) {
>         console.log(`attack: ${this.name} ${one} {two}`)    -> 這個等等會印出什麼呢 ？？？？
>     }
> }
```

```md
> // 有一個蟻人物件
> const antMan = {
>     name: "antMan",
> }
> 
> // 有一個one物件
> const one = {
>     name: "enemy2"
> }
> 
> // 把兩個物件和一個引數帶進去
> hero.attack.call(antMan, one.name, 3)
> 
> Ans. 印出 attack: antMan enemy2 3
```



### apply

apply跟call非常像，差異的地方，只有在最後帶參數的地方，call是帶多個參數  
而apply是帶一個陣列，上面的fc接到的是陣列中的值  


#### 範例一
一樣用剛剛的英雄模組來看看



在剛剛的攻擊fc裡面，放兩個參數，並在新英雄apply的時候帶陣列引數進去
```md
> const hero = {
>     name: "spider",
>     attack: function(one, two) {
>         console.log(`attack: ${this.name} ${one} {two}`)    -> 這個等等會印出什麼呢 ？？？？
>     }
> }
```

```md
> 一樣有蟻人物件
> const antMan = {
>     name: "antMan",
> }

> 使用apply，後面帶陣列引數
> hero.attack.apply(antMan, ["333",2,3,4,5]))

> Ans. 印出 attack: antMan 333 2
> Ps. 有沒有覺得奇怪，陣列明明就給一堆，但是只有印出前兩個，原因就是因為，普通在帶進引數時
>     超過的參數數量的時候，fc會直接無視多給的值
```




### bind
  
:::tip
**bind方法**  
該bind()方法創建一個新函式，該函式被呼叫時，會將 this 關鍵字設為給定的參數，並在呼叫時，帶有提供之前給定順序的參數。  
  
**回傳值**  
他會回傳一個新的 fc，並且會把傳進去的東西東做this的初始值  
:::


bind 跟 apply、call 的差異是， bind 會比較晚觸發(你去主動執行的時候才會觸發)  
這聽起來有點抽象，下面實作幾個範例  

#### 範例一   
一樣先做一個英雄製造function  

```md
> function HeroCreator() {
>     console.log(this)         
> }
```
```md
> HeroCreator()     # 印出window
```

接著用bind創造出新的fc
```md
> const newHero = HeroCreator.bind(123)       # 對英雄製造fc用bind
> console.log(newHero)                        # 印出新英雄
> 
> 印出 function HeroCreator()      -> 可以發現，被bind設定過後，新的變數會變成fc
```
```md
> 也因為是newHero是fc，他要主動呼叫才會執行
> newHero()                                 # Number {123}
```
  
  
  
#### 範例二
我們用物件來輔助說明，了解bind用在物件上會發生什麼事  


創立一個新英雄物件 - 鳥人  
```md
> const birdMan = {
>     name: "bird",
>     profess: "wind",
> }
```
  
一樣用剛剛的newHero的bind，不過要把bind裡面的東西改成birdMan  
```md
> const newHero = HeroCreator.bind(birdMan)    # 改bind裡面的東西
> newHero()                                    # 主動呼叫

> 印出 {name: "bird", profess: "wind"}         # newHero()呼叫後，會印出鳥人英雄
```




#### 範例三

不過這樣還是不清楚，為啥要用bind對不對？不知道bind的好處在哪，我們多加一行，你就知道優點了
```md
> function HeroCreator() {
>     console.log(this);
> }

> const newHero = {name: "bird", profess: "wind"} 

> const newHero = HeroCreator.bind(bird)
 
> //新增這一行
> newHero.profess = "windsssssss"       # 我在呼叫前，先改變bird的profess

> newHero()                             # 正式呼叫

> 印出 {name: "bird", profess: "windsssssss"}  
> 因為在呼叫前，鳥人的prefess就被更改了，所以正式呼叫後物件內容換新
```


:::tip **call 跟 apply 的差異**  
call後面參數是接多個數字  
apply後面是接陣列  
:::

:::tip **call、apply和bind的差異**  
bind在呼叫前都可以更改，不像call一下指令下去，東西就被更改了  
而且bind出來的變數，會變成function  
:::



### 4. 是否使用箭頭函式，箭頭函式沒有this

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
  
:::info
> 箭頭函示不只沒有this，也沒prototype  
:::





五、誰呼叫this
----

### 5. 誰呼叫，誰就是this
  
#### 範例一  
哪一個主體去呼叫這個this，這個this就是他，以下面的案例，hero物件主動去呼叫fc裡面的this  
```md
> const hero = {
>     hi: function() {
>         console.log(this)   -> 這個this就是hero
>     }
> }
> hero.hi()
```
  
#### 範例二
另一種狀況，如果今天this被包在第二or三層的fc裡面，還是要呼叫this的人是誰    
```md
> function hi() {
>     function hello() {
>         const solider = {
>             name: "nothing",
>             attack: function() {
>                 console.log(this)   # 2. 因此這個this指向solider
>             }
>         }
>         solider.attack()            # 1. 呼叫的人是solider
>     } 
>     hello()
> }
> hi()
```



六、沒人呼叫this
------

### 6. 沒人呼叫this的話，this會變全域物件

#### 範例一
```md
> fuction hi() {
>     console.log(this)       -> 這個this會變成全域物件
> }
> hi()
```
  
#### 範例二
另一種狀況，就算this被包在第二層裡面，他還是算沒人呼叫他  
因為this注重的是，在哪發動，並且發動的當下，有沒有人呼叫他  
  
```md
> fuction hi() {
>     function hey() {
>         console.log(this)       -> 這個this也是全域物件
>     }
>     hey()                        
> }
> 
> hi()
```
