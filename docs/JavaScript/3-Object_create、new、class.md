---
title: JavaScript/Object_create、new、class
sidebar_label: "3. [JavaScript] Object_create、new、class"
description: Object_create、new、class
last_update:
  date: 2022-09-25
keywords:
  - JavaScript
  - Object
sidebar_position: 3
---




Object.create(null)
------

### 設定一個創建英雄模組

今天想創造一個英雄模組(之後可以快速生成一個英雄)，因此我們用Object.create來處理(不過目前是還沒加上參數的)
```md
> // 創造英雄的fc
> function heroCreator(profess, skill) {
>     // 創造新的英雄，是否要參考某個物件(ex. 我要讓英雄都有共通技能)
>     // 如果沒有要參考，就給一個空物件就好
>     const hero = Object.create(null)
>     
>     // 新增英雄的屬性(可以隨便加)
>     // 這邊選擇的加法是，直接把傳進來的參數變成英雄的屬性
>     hero.profess = profess
>     hero.skill = skill
> 
>     // 為英雄物件加一個技能
>     hero.bye = function() {
>         console.log("byeeeeeee");
>     }
> 
>     // 把使用fc的結果傳出來
>     return hero
> }
```

如果直接使用fc、帶引數進去的話，會得到下面這一串
```md
> console.log(heroCreator(123, 46456));

> [Object: null prototype] {
>     profess: 123,
>     skill: 46456,
>     bye: [Function (anonymous)]
> }
```

```
> // 來用上面那個打造一個新英雄
> const warrior = heroCreator("warrior", "bigSword")
> 
> // 這樣我們就有一個戰士英雄了，並且之後要打造新英雄，直接用heroCreator就好
> console.log(warrior);
> 
> [Object: null prototype] {
>   profess: 'warrior',
>   skill: 'bigSword',
>   bye: [Function (anonymous)]
> }
```


***

### 讓英雄模組可以繼承共通技能

不過剛剛那樣我們新創的英雄沒有繼承共有技能  
所以我們稍微改一下Object.create、創造一個共通技能(新物件)  

```md
> 先創造一個所有英雄都有的技能(一個物件，裡面有兩個fc)

> const skills = {
>     attack: function() {
>         console.log("attack!!!!");
>     },
>     shield: function() {
>         console.log("defense!!!!!!");
>     }
> }
```



```md
> function heroCreator(profess, skill) {

>     const hero = Object.create(skills)   -> 創造新物件時，會繼承skills能力

>     hero.profess = profess
>     hero.skill = skill
> 
>     hero.bye = function() {
>         console.log("byeeeeeee");
>     }
> 
>     return hero
> }
```

我們來看看新英雄有哪些技能
```md
warrior.bye()            # 自己本身有的技能

warrior.attack()         # attack、shield -> 繼承自skills的技能
warrior.shield()
```

那我們來看看warrior怎麼繼承技能的
```md
> warrior.__proto__      # {attack: f, shield: f}

查看warrior的原型鏈，可以發現共用技能放在上一層，所以warrior會自動可以使用共用技能
```


***


new Object
------

上面用Object.create(null)流程創造新物件，有沒有覺得超級麻煩？  
因此我們用另一個更快、更彈性的方法 - new 新物件  


```md
> 前面一樣要先建立function，不過中間流程不太一樣

> function heroCreator(profess, skill) {
>     this.profess = profess,
>     this.skill = skill
> }

> Ps. 原本超級長的英雄模組，現在縮短成這樣有沒有覺得超讚的
```

建好模組後，我們要用這個模組來創造新的英雄了
```md
> 創建一個鷹眼，在模組fc前，多加一個new，就可以做到跟Object.create(null)一樣的功能

> const hawkEye = new heroCreator("hawkEye", "arrow")
```

```md
console.log(hawkEye)          # heroCreator { profess: 'hawkEye', skill: 'arrow' }
```




### new 物件的時候發生了啥事？

這個new會幫我們做幾件事情  
1. 產生一個空的物件  
2. {}.__proto__ = function.prototype  
3. 會有一個 this 指向空物件  
4. 自動 return 物件  


#### 以剛剛英雄模組為例
```md
> function heroCreator(profess, skill) {
>      // this -> {}                             -> new的時候會有一個空物件，並且this會指向空物件
>      // {}.__proto__ = heroCreator.prototype   -> 再來物件的proto會 = fc的prototype
>     this.profess = profess,
>     this.skill = skill
> }

> const hawkEye = new heroCreator("hawkEye", "arrow")   -> new 物件
```

:::tip
new寫法除了比較簡短外，還多了 this 功能，this 功能之後會再開一篇解釋  
:::


***


### prototype、proto是什麼

不過剛剛那個new物件，有沒有發現好像少了什麼?  
沒錯，就是少了繼承的技能，也就是共通技能attack、shield沒有繼承到  
那我們要怎麼用new物件來做呢?  
  
這邊我們要先提到另外一個東西，就是prototype，還記得剛剛有提到，new物件的時候  
有做幾件事，其中一件事是這個嗎?
```md
> {}.__proto__ = heroCreator.prototype  
```
  
這一句話的意思就是，把一個空物件指向一個空物件  
```md
> {}.__proto__ =  {} 空物件
> heroCreator.prototype = {} 空物件
> 
> ex. 當有**物件**去找__proto__的時候，請去找由heroCreator這個fc，做此物件的prototype

> 如果今天用typeof把兩個狀態印出來，會印出Object喔!!
> console.log(typeof hawkEye.__proto__)        # Object
> console.log(typeof heroCreator.prototype)    # Object
```

:::tip
**problem**
為啥 heroCreator.prototype 是一個物件  
heroCreator本來不是一個fc嗎？  
   
   
**Ans**  
function 自己本身就是一個物件 & prototype也是一個物件，所以物件.物件還是一個物件   
:::


上面這樣說可能會有點模糊，實際把兩個印出來看看
```md
> hawkEye.__proto__      # {constructor: ƒ} -> hawkEye 的原型鏈下個節點是fc

> heroCreator.prototype  # {constructor: ƒ} -> fc的prototype印出來跟hawkEye是一模一樣的東西

> hawkEye.__proto__ === heroCreator.prototype  # true 
```

:::tip
如果今天是用瀏覽器看，把 `{constructor: ƒ}` 點開，可以發現裡面還有一堆東西  
那些東西還可以繼續用原型鏈往下看還有哪些共同的東西寫在裡面  
:::
  
  
如果上面還是看不懂，我們實際來寫個範例試試  
```md
> 剛剛上面有提到，heroCreator.prototype是一個物件，那我們來為這個物件加功能

> heroCreator.prototype.good = "g123123ood"
> console.log(hawkEye.good)

> Ps. 物件狀態可以隨意加屬性，我們現在加一個.good的屬性，再加上
>     前面用new有讓兩個空物件相等的動作，所以 **鷹眼可以直接取用good屬性**
```


#### 額外提醒1 - 當**Object1 = Object2**時，會發生的事情
```md
> 我現在直接做兩個物件，並讓物件1 = 物件2

> let test1 = {
>     bye1: "1111",
>     hello1: "2222"
> }
> let test2 = {
>     bye2: "33333",
>     hello2: "44444"
> }
> 
> test1 = test2
> console.log(test1.bye2);   # "33333"  -> 物件1可以直接取用物件2的屬性
```


#### 額外提醒2 - prototype、proto注意事項
:::tip
所有物件都有 __proto__ 屬性  
所有 fn 都有 prototype 屬性、__proto__屬性  => 所有 fc 的 prototype 都是空物件  
(只有fn同時都有兩個屬性)  
:::



***


### function.prototype = Object

了解完prototype後，我們就可以來為剛剛的鷹眼(新英雄)增加繼承技能了  
  
先寫共同的技能(物件)  
```md
> const skills = {
>     attack: function() {
>         console.log("attack!!!!");
>     },
>     shield: function() {
>         console.log("defense!!!!!!");
>     }
> }
```


#### 一樣以剛剛英雄模組為例
```md
> function heroCreator(profess, skill) {
>      // this -> {}                            
>      // {}.__proto__ = heroCreator.prototype  
>     this.profess = profess,
>     this.skill = skill
> }

> 在new物件之前，我們先把heroCreator = skills，新英雄就可以使用共有技能了!!!!
> heroCreator.prototype = skills
> const hawkEye = new heroCreator("hawkEye", "arrow")   
```









Class
------

剛剛提到的Object.create、new，都是JS創造物件的方式，不過在其他語言，class的寫法反而是大宗，因此JS也有class寫法的語法糖(白話文就是，JS也可以用class格式寫法來創造物件，只是它底層還是用Object.create、new寫出來的，只是我們肉眼看不出來)  


首先創造一個最上層的英雄模組(創建新英雄傳的引數，都會在這被接住)
```md
> class Hero {
>     constructor(profess, skill) {
>         this.profess = profess,
>         this.skill = skill;
>     }
> }
```

再來創建英雄技能，此模組有繼承Hero的功能，所以所以新創建的英雄都可以有以下技能
```md
> class heroCreator extends Hero {                   -> extends Hero 可以繼承
>     attack() {
>         console.log("attack!!!!");
>     };
>     shield() {
>         console.log("defense!!!!")
>     }
> }
```

最後創建新英雄 -> 蜘蛛人!!!!!!!!!
```md
> const spider = new heroCreator("spider", "silk")
```


這時候我來把spider的原型鏈下個節點有哪些東西，印出來看看
```md
> spider.__proto__  

> 印出以下四個項目，可以發現constructor就是Hero
> attack: f attack()
> constructor: class heroCreator
> shield: f shield()
> [[Prototype]]: Object
```


