---
title: ROR/HTML&CSS(02)
sidebar_label: "DAY4. [ROR] HTML&CSS-2"
description: HTML&CSS
last_update:
  date: 2022-10-06
keywords:
  - 5xRuby
  - HTML
  - CSS
sidebar_position: 5
---




1、class的命名
------

### 1-1class 命名禁止事項

(1) 開頭不能為數字 -> ex. 1good     
(2) 不要用編號 -> ex. btn1、btn2、btn3  
(3) 不要用左跟右 -> ex. btnleft、btnright   
(4) 不要用奇怪符號 -> ex. $、%  


### 1-2、class 命名風格
不管是蛇式，還是駝峰式都有機會用到，沒有特定要使用的命名方式，看不同場合有不同適合的寫法    
```md
> .news-list
> .news_list
> 
> .newsList
> .NewsList
> 
> .news_list_disable
> .news-list_disable
```


### 1-3、BEM命名方式

(1) Block -> ex. header、container、menu、checkbox、input   
(2) Element -> ex. menu item、list item、header title   
(3) Modifier(狀態) -> ex. disabled、checked、fixed、size big    

在做class命名的時候，也有很多人會依照此順序來做命名，假設今天有一個header，裡面有title，title的狀態式disabled，就會像這樣依序的命名下去，不過這樣命名有個缺點，就是class會變得很長  



2、inline vs. block
------

(1) inline 類型不能設定寬跟高  
(2) block  類型可以設定寬跟高  
  
inline可以把物件全部縮到同一行，但有時候縮到同一行，又因為不能控制他的寬，導致使用上常常覺得卡卡的，這時候就要用**flex**來搞定，後面會介紹到  


3、inline vs block vs inline-block
------
||inline|block|inline-block|
|:-:|:-:|:-:|:-:|
|width|X|O|O|
|height|X|O|O|
|padding上下|X|O|O|
|padding左右|O|O|O|
|margin上下|X|O|O|
|margin左右|O|O|O|
|預設排列方向|橫向|直向|橫向|
|vertical-align|O|X|O|
|transform|X|O|O|
|物件間的空白|O|X|O|
|預設寬度|內容寬|父層content寬|內容寬|

Ps. 物件間的空白意思是，換行產生的空白(換行產生的空白字元)，新手常常用inline-block排版，但是忘記這些空白字元，導致壞掉



### 3-1、修改物件間的空白

如果今天像下面這樣設定，換行的時後就會有一點點的空白字元

```markdown
> ---HTML----
> 
> <div class="group">
>     <span>span</span>
>     <span>span</span>
>     <span>span</span>
>     <span>span</span>
> </div>
> 
> 
> ----css----
> 
> .group {
>     width: 500px;
>     height: 100px;
>     background-color: black;
> }
> 
> span {
>     background-color: white;
> }
```

### 3-2、空白間隔解法

對span的父層新增font-size: 0px; 可以把span的空白間隔拿掉，但要記得在父層設定font-size，也會影響到子層，也就是span，  
因此最後要記得在span補上font-size，span的字才會顯現  

```markdown
.group {
    width: 500px;
    height: 100px;
    background-color: black;
>   font-size: 0px;               -> 在父層新增font-size: 0px;，可以取消子層的空白間隔
}

span {
    background-color: white;
>   font-size: 20px;              -> 記得補上子層的font-size，要不然會被父層的font-size影響
}
```


### 3-3、inline-block 介紹



:::tip **inline block = inline + block**  
inline-flex => 橫向排列的flex     
inline-grid => 橫向排列的gird  
如果有一天發現有區塊的寬高設定不了，記得查看是不是inline物件  
:::


4、初試切版
------

切版注意事項：  
實際佔據空間 - 寬 + padding + **margin** + border   
可見佔據空間 - 寬 + padding + border    

```markdown

<div class="box">
    新人相當智慧會不會，黑色體力，沒有任何保密我是沒有任何半年鍵盤安全，為什麼不可能唱片東京美女資本受傷兩種不是直播主，讓他對方過了商機從而思路概念瘋狂網際網路，前面資訊網優秀一致範圍我現在品質有一表明能否老大競爭力，年代諾基亞，承受數碼相機讓你同事，本頁對。
</div>


* {
    margin: 0px;
    padding: 0px;
}
.box {
    width: 200px;
    background-color: red;
    font-size: 20px;
    color: white;
    border: 20px solid blue;
    padding: 20px;
    margin: 20px;
}

```

### 4-1、情境題-切版練習

```markdown
> 題目： 父層 寬1000
>       子層 有四個區塊 margin: 15 padding: 10 border: 1px 

> <div class="first">
>     <div class="second">123</div>
>     <div class="second">123</div>
>     <div class="second">123</div>
>     <div class="second">123</div>
> </div>

> .first {
>     width: 1000px;
>     background-color: beige;
>     display: flex;
>     flex-wrap: wrap;
> }
> .second {
>     width: 198px;
>     border: 1px solid black;
>     padding: 10px;
>     margin: 15px;
> }
```



5、box-sizing 一個物件由四個盒子組成
------

(1) content-box -> 文字區域(寬跟高作用的地方)   
(2) padding-box -> 文字區域 + padding   
(3) border-box  -> 文字區域 + padding + border  
(4) margin-box  -> 文字區域 + padding + border + margin  

**box-sizing -> 這個屬性是把寬高作用在另外一個上面 (預設是作用在content-box上)**    
```md
> .box {
>     width: 200px;
>     padding: 20px;
>     border: 1px solid red;
>     box-sizing: border-box        -> 寬高設定作用在border-box上
>                                      *整個顯示就是 width + padding + border = 200
> }
```



6、flex 介紹
------

### 6-1、flex的特性 - 子物件可以設定寬度

(1) 當把一個父層設定成flex，會對子層產生作用(只有第一層)，簡單說，對一個父層設定flex，他的子層會有block特性，並且會縮到同一行  
(2) 父層自己設定為flex，自己也可以設定寬跟高

下面的是flex對子物件的影響

```HTML
> <div class="flex">
>     <div class="item">1</div>
>     <div class="item">2</div>
>     <div class="item">3</div>
> </div>
```

```css
.flex {
    display: flex;
    width: 500px;
    height: 300px;
    background-color: red;
}

.item {
    width: 50px;
    height: 50px;
    margin: 2px;
    background-color: grey;
}
```


### 6-2、子層不設定高度的情況下，會跟父層等高

```markdown
> <div class="flex">
>     <div class="item">1 教學對付選項認定世紀刺激影視意見</div>
>     <div class="item">2</div>
>     <div class="item">3</div>
> </div>

> <style>
>     .flex {
>         width: 600px;
>         height: 600px;
>         background-color: grey;
>         display: flex;
>     }
> 
>     .item {
>         width: 120px;
>         background-color: red;
>         margin: 2px;
>     }
> </style>
```

### 6-3、加上flex-wrap: wrap;  -> 如果沒有算好寬度，會折行
```markdown
> <div class="flex">
>     <div class="item">1 教學對付選項認定世紀刺激影視意見</div>
>     <div class="item">2</div>
>     <div class="item">3</div>
> </div>

<style>
    .flex {
        width: 600px;
        height: 600px;
        background-color: grey;
        display: flex;
>       flex-wrap: wrap;           -> 如果設定，寬度沒算好會折行
    }

    .item {
        width: 200px;
        background-color: red;
        margin: 2px;
    }
</style>
```

### 6-4、flex的軸向

flex的軸向 - 調整flex前都要先看軸向，再調整
```markdown
    | 
----|-----> 主軸 (main axis)
    |
    V
  交叉軸 (cross axis)
```

### 6-5、justify-content
控制子物件中在主軸上的對齊

### 6-6、align-items
控制子物件中在交叉軸上的對齊


### 6-7、flex-direction - 可以改變item排列方向
在使用 flex-direction: row 的時候，不一定是由右至左，如果今天改變了書寫方向     
ex. direction or writing-mode，都會改變row的方向(要看當下row的定義是什麼)   

```md
> flex-direction: row (此為預設值)
> flex-direction: column
> flex-direction: row-reverse
> flex-direction: column-reverse
```


### 6-8、flex-wrap
當你不希望 flex items 溢出 container 時，你可以使用 flex-wrap 來「包裝」內容，也就是讓他們遇到邊界就「自動換行」    
這在設計 RWD 時會非常地有用 

```md
> flex-wrap: nowrap (此為預設值)
> flex-wrap: wrap
> flex-wrap: wrap-reverse -> 非常少用到
```


:::tip
**wrap其實是在控制交叉軸的方向**  
要特別注意的是，如果 flex items 設定了 min-width，items 不會縮小反而會溢出 flex container 
:::


### 6-9、flex-flow

flex-direction 和 flex-wrap 這兩個屬性可以合併成 flex-flow 屬性。flex-flow 接受兩個值，第一個是給 flex-direction 第二個是給 flex-wrap

```markdown
.flex-box {
    flex-flow: row wrap;
}
```

### 6-10、對齊flex內容物

由於 flex 是單一維度的，你可以用 flex-direction 來切換主軸 (main axis)與交叉軸 (cross axis)，然後再用以下兩個屬性控制對齊
```md
> justify-content - 用於 main axis
> align-items - 用於 cross axis
```

### 6-11、justify-content 可接受6個值
```md
> justify-content: flex-start (default)  
> justify-content: flex-end  
> justify-content: center  
> justify-content: space-between; -> 左右貼齊外框線，中間空格平均  
> justify-content: space-around;  -> 最旁邊兩個物件的空格寬，是中間所有空格的一半  
> justify-content: space-evenly;  -> 所有物件左右平均  
```


7、實戰練習 - 用flex來做nav選單
------

```HTML
<div class="nav-bar">
    <div class="logo">logo</div>
    <div class="ul-list">
        <ul>
            <li><a href="">首頁</a></li>
            <li><a href="">關於我們</a></li>
            <li>聯絡我們</li>
            <li>首頁</li>
            <li>關於我們</li>
            
        </ul>
    </div>
</div>
```
```CSS
* {
    margin: 0px;
    padding: 0px;
}

.nav-bar{
    width: 1000px;
    background-color: beige;
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    height: 200px;
    align-items: center;           -> 在選單父層設定，交叉軸置中
}

.logo {
    width: 200px;
    background-color: aqua;
    /* vertical-align: top; */     -> 讓圖片對齊TOP線，可以解決圖片下面會留一點小空間
    display: block;                -> 或者直接把圖片改成不是inline屬性
}

.ul-list {
    width: 600px;
    background-color: brown;

}


.ul-list ul {                      -> 在 ul 設定flex 可以把li變橫排
    display: flex;
    flex-direction: row;
    justify-content: space-evenly; -> 平分 li 空間 
}
.ul-list li {
    list-style: none;              -> 把 li 的點點刪掉
}

.ul-list a {
    padding: 20px;                 -> 把 超連結 的寬度撐開，更容易點擊
    display: block;                -> 記得要把此區塊設定成 block ，才可以設定padding
}

```

### 7-1、注意事項

#### 7-1-1、圖片本身是inline物件

```css
.logo {
    vertical-align: top;        -> 讓圖片對齊TOP線
    display: block;             -> 或者直接把圖片改成不是inline屬性
}
```

#### 7-1-2、如果今天要讓選單的li撐開高度，可以用padding
```css
.ul-list li {
    list-style: none;
    padding: 20px;

}
```

#### 7-1-2、不過如果今天有超連結，想要讓超連結連動寬度的話，padding就要設定在 a 上面
```css
.ul-list a {
    padding: 20px;
    display: block;      -> 記得要讓 a 變block，超連結才能更動寬高，要不然因為前面設定flex
                            他還是inline狀態
}
``` 


8、製作五倍官網的前往課程區塊
------

```HTML
<section>
    <div class="container">
        <div class="img">img</div>
        <div class="text">
            <h1>線上課程</h1>
            <p>你的不在數碼，盯着推進重視，氣氛神經無比需求接口你沒有允許印刷傳說，並且上一頁黑暗減肥都在美麗水晶</p>
            <a href="">前往課程</a>
        </div>
    </div>

    <div class="container">
        <div class="img">img</div>
        <div class="text">
            <h1>線上課程</h1>
            <p>你的不在數碼，盯着推進重視，氣氛神經無比需求接口你沒有允許印刷傳說，並且上一頁黑暗減肥都在美麗水晶</p>
            <a href="">前往課程</a>
        </div>
    </div>

</section>
```
```css
<style>
    section {
        width: 960px;
        background-color: azure;
        display: flex;
        flex-direction: row;
        justify-content: space-around;

    }

    .img {
        width: 450px;
        height: 200px;
        background-color: grey;
    }

    .container {
        width: 450px;
        background-color: bisque;
        padding-bottom: 10px;
    }        

    h1, p {
        padding: 10px;
    }

    a {
        padding: 10px;
        display: inline-block;
    }

</style>

```


9、block區塊置中 - margin: auto
------

(1) margin: auto - 只適用block物件  
(2) 如果使用auto，左 or 右可以 **取得剩餘空間** (剩餘空間分配)  

```markdown
.div {
    margin: 0px auto;       -> 左右都取剩餘空間，區塊會置中 (常見div置中用法)
}
```


