---
title: ROR/HTML&CSS(09)
sidebar_label: "DAY18. [ASTROCamp] HTML&CSS-9"
description: HTML&CSS
last_update:
  date: 2022-10-31
keywords:
  - 5xRuby
  - HTML
  - CSS
sidebar_position: 19
---



1、格線系統補充
------

在做格線系統的巢狀容器的時候  
row 層會被設定 margin-left: -10px;、margin-right: -10px;      
要設定負值的原因，column 被設定 padding  
如果沒用 margin 負值去吃掉這些 padding，整個版面會歪掉(之後可以自己切試試)  
  
  
  
格線會分成三個容器  
1. container - 這個也要記得設定 padding，要不然會被第二層的 row 可以推出來  
2. row - 讓裡面的欄可以切分多個，並對 column 設定 flex  
3. column - 設定欄寬  
  
Ps. 如果想要在 column 裡面在切分，就要在 column 塞一層 row(為了要讓 paddding 被抵消)  


### 1-1、格線系統共用設定  
```md
> .container {
>     margin: auto;
>     padding-left: 15px;
>     padding-right: 15px;
> }
> .row {
>     display: flex;
>     flex-wrap: wrap;
>     margin-left: -15px;
>     margin-right: -15px;
> }
> 
> [class*="col"] {
>     padding: 15px;
>     box-sizing: border-box;
> }
```


### 1-2、實際隔線切版練習

完整code在day18的test_grid
```md
> <div class="container">
>     <div class="row">
>         <div class="col-12 col-md-6">
>             <div class="item">
>                 1111111111
>             </div>
>         </div>
>         <div class="col-12 col-md-6">
>             <div class="row">
>                 <div class="col-12 col-md-3">
>                     <div class="item">12345</div>
>                 </div>
>                 <div class="col-12 col-md-3">
>                     <div class="item">12345</div>
>                 </div>
>                 <div class="col-12 col-md-3">
>                     <div class="item">12345</div>
>                 </div>
>                 <div class="col-12 col-md-3">
>                     <div class="item">12345</div>
>                 </div>
>                 <div class="col-12 col-md-3">
>                     <div class="item">12345</div>
>                 </div>
>                 <div class="col-12 col-md-3">
>                     <div class="item">12345</div>
>                 </div>
>                 <div class="col-12 col-md-3">
>                     <div class="item">12345</div>
>                 </div>
>                 <div class="col-12 col-md-3">
>                     <div class="item">12345</div>
>                 </div>
> 
>             </div>
>         </div>
>     </div>
> </div>
```



2、自訂[snippet](https://snippet-generator.app/)
------

進到網站後，寫上description、trigger，就可以把想要使用快捷鍵的code貼上去  
接著就可以到vs的設定使用者程式碼片段的html.json檔  
把剛剛snippet的code貼上去，之後就可以使用快捷鍵  







3、手機選單 - hamburger
------
  
最外層用div，內層可以用span就好  
  
### 3-1、方法一：三塊div
```md
> HTML

> <div class="hb">
>     <div class="bar"></div>
>     <div class="bar"></div>
>     <div class="bar"></div>
> </div>
```
```md
> css

> .hb {
>     width: 40px;
>     height: 40px;
>     background-color: gold;
>     display: flex;
>     flex-direction: column;
>     justify-content: center;
>     align-items: center;
> }
> 
> .bar {
>     width: 34px;
>     height: 4px;
>     background-color: beige;
> }
> .bar:nth-child(2) {
>     margin: 4px 0px;
> }
```

### 3-2」方法二:一塊div + before、after
```md
> HTML

> <div class="hb">
>     <div class="bar"></div>
> </div>
```
```md
> CSS

> .hb {
>     width: 40px;
>     height: 40px;
>     background-color: goldenrod;
>     display: flex;
>     flex-direction: column;
>     justify-content: center;
>     align-items: center;
> }
> 
> .hb::before,
> .hb::after {
>     content: '';
>     display: block;
>     background-color: red;
>     width: 34px;
>     height: 4px;
> }
> 
> .bar {
>     width: 34px;
>     height: 4px;
>     background-color: beige;
>     margin: 5px 0px;
> }
```

### 3-3、方法三：一塊div + box-shadow
```md
> HTML

> .hb {
>     width: 40px;
>     height: 40px;
>     background-color: blanchedalmond;
>     display: flex;
>     flex-direction: column;
>     justify-content: center;
>     align-items: center;
> }
> 
> .bar {
>     width: 34px;
>     height: 2px;
>     background-color: beige;
> 
>     box-shadow: 0px -8px 0px #fa0, 0px 8px 0px;
> }
```
```md
> CSS

> .hb {
>     width: 40px;
>     height: 40px;
>     background-color: blanchedalmond;
>     display: flex;
>     flex-direction: column;
>     justify-content: center;
>     align-items: center;
> }
> 
> .bar {
>     width: 34px;
>     height: 2px;
>     background-color: beige;
> 
>     box-shadow: 0px -8px 0px #fa0, 0px 8px 0px;
> }
```





### 3-4、menu-switch概念

純css控制漢堡的打開關閉
```md
> <input type="checkbox" id="menu-switch">
> <p>Lorem ipsum dolor sit amet consectetur adipisicing elit. Expedita, eos > </p>
> <p>Lorem ipsum dolor sit amet consectetur adipisicing elit. Expedita, eos > </p>
> <p>Lorem ipsum dolor sit amet consectetur adipisicing elit. Expedita, eos > </p>
> <p>Lorem ipsum dolor sit amet consectetur adipisicing elit. Expedita, eos > </p>
> <label for="menu-switch">III</label>
```

```md
p {
    display: none;
}

#menu-switch:checked ~ p {
    display: block;
}
```

### 3-5、手機漢堡選單、RWD製作
```md
> HTML

> <input type="checkbox" id="menu-switch">
> <header class="main-header">
>     <div class="logo">
>         <a href="">
>             <img src="https://fakeimg.pl/80x40/200">
>         </a>
>     </div>
> 
>     <label for="menu-switch" class="hb">
>         <span class="bar"></span>
>         <span class="bar"></span>
>         <span class="bar"></span>
>     </label>
> 
>     <div class="main-nav">
>         <a href="">Link</a>
>         <a href="">Link</a>
>         <a href="">Link</a>
>         <a href="">Link</a>
>         <a href="">Link</a>
>     </div>
> </header>
```
```md
> CSS

> * {
>     margin: 0px;
>     padding: 0px;
> }
> 
> .main-header {
>     display: flex;
>     justify-content: space-between;
>     padding: 10px;
>     background-color: antiquewhite;
>     align-items: center;
> 
>     position: relative;
> }
> 
> .logo img {
>     vertical-align: top;
> }
> 
> .hb {
>     width: 40px;
>     height: 40px;
>     background-color: blanchedalmond;
>     display: flex;
>     flex-direction: column;
>     justify-content: center;
>     align-items: center;
> }
> 
> .hb span.bar {
>     width: 34px;
>     height: 4px;
>     background-color: green;
> }
> 
> .hb span.bar:nth-child(2) {
>     margin: 5px 0px;
> }
> 
> .main-nav {
>     position: absolute;
>     top: 100%;
>     left: 0px;
>     display: none;
>     background-color: gold;
>     width: 100%;
> 
> }
> 
> .main-nav a {
>     display: block;
>     padding: 10px 20px;
>     text-decoration: none;
> }
> 
> .main-nav a + a {
>     border-top: 1px solid black;
> }
> 
> #menu-switch:checked ~ .main-header .main-nav {
>     display: block;
> }
> 
> #menu-switch {
>     display: none;
> }
> 
> @media screen and (min-width: 768px) {
> 
>     .hb {
>         display: none;
>     }
> 
>     .main-nav {
>         position: relative;
>         top: auto;
>         left: auto;
>         display: flex;
>         background-color: transparent;
>         width: fit-content;
> 
>     }
> 
>     .main-nav a {
>         
>         padding: 10px 20px;
>         
>     }
> 
>     .main-nav a + a {
>         border-top: transparent;
>     }
> 
>     #menu-switch:checked ~ .main-header .main-nav {
>         display: flex;
>     }
> 
>     #menu-switch {
>         display: none;
>     }
> }
```








