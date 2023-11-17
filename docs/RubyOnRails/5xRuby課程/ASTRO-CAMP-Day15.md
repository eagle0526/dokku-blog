---
title: ROR/HTML&CSS(08)
sidebar_label: "DAY15. [ASTROCamp] HTML&CSS-8"
description: HTML&CSS
last_update:
  date: 2022-10-25
keywords:
  - 5xRuby
  - HTML
  - CSS
sidebar_position: 16
---


1、CSS reset
------

(1) 各家瀏覽器會有預設的外觀(margin or padding不一樣)，為了解決各家瀏覽器的差異性，css reset就此誕生。      
(2) css reset不一定要用誰的，只要能消除差異性就好。   
(3) 推薦老師很常用的[meyerweb](https://meyerweb.com/eric/tools/css/reset/)，並多開一個css檔案link: reset.css，放在原本網頁css檔案的前面。



2、CSS normalize
------

(1) 破壞性沒有這麼強大的reset寫法，只更改了瀏覽器差異性的地方     
(2) 先把normalize加進去後，在根據自己想要h1、h2、h3......等等的行高使用  


3、*->通用選取器
------

無差別全部選取



4、子選單製作 - Submenu
------
(1) 在第一層ul的li層，再建立一層ul>li>a，就可以建立子選單   
(2) 摸到li秀出子選單    
(3) 子選單用絕對定位    
(4) 選單大忌，滑鼠移到li處，但是不能點    


＊＊＊＊＊  
先設定好主選單的ul、li，再設定子選單的ul、li，接著對主選單的a，把a撐開後(block + padding)     
設定子選單摸到前要display: none(隱藏)，當主選單的li被摸到的時候，子選單要被撐開來   
＊＊＊＊＊  


```md
> 摸到主選單li，子選單要出現

> .mainMenu > li:hover > .subMenu {
>     display: block;
>     position: absolute;
>     width: 8em;
> } 
```

如果要設定第三層的話，第三層ul的class一樣用subMenu，因為這樣等等可以直接沿用特性
```markdown
> .mainMenu li:hover > .subMenu {   -> 原本只有座兩層的時候
                                       用 > ，但是有第三層就拿掉
>     display: block;                  原因是因為 > 只有對下一層作用
>     position: absolute;
>     width: 8em;
> } 
> 
> .subMenu {
>     display: none;                -> subMenu 一開始都隱藏，第二層第三層都隱藏
> }
> 
> .subMenu .subMenu{                -> 對第三層更改樣式
>     left: 100%;
>     top: 10px;
>     background-color: antiquewhite;
> }
```


最後最重要的，滑鼠移到 li 上的時候要變色，這邊一行就可以搞定，因為前面HTML結構的關係      
對主選單寫li:hover，可以直接影響到後面所有的li  

```markdown
> .mainMenu li:hover {
>     background-color: aquamarine;
> } 
```

Ps. 實際code再day15的 -> submenu.html     



5、RWD
------

(1) 父層寬度1200  
(2) 裡面有三個item (寬度都給 % 數)  
(3) 高度千萬不要寫死    

```markdown
> w: 370 = 30.833333% (370 / 1200 * 100)
> margin: 15px = 1.25% (15 / 1200 * 100)
> padding: 10px (不用轉%數)
> box-sizing: border-box
```

```markdown
> 父層
> .wrap {
>     /* width: 1200px; */
>     max-width: 1000px;              -> 最大寬度，寬度以下就會縮小
>     outline: 1px solid black;
>     margin: auto;
>     display: flex;
> }

> 子層
> .item {
>     width: 30.8333333%;             -> 30.833333% (370 / 1200 * 100)
>     /* height: 200px; */            -> 千萬不要對子物件寫高度
>     outline: 1px solid tomato;
>     margin: 1.25%;                  -> 1.25% (15 / 1200 * 100)
>     box-sizing: border-box;         -> 改變model-box
> 
> }
```




6、媒體查詢 - media
------

(1) 通常設定media都會寫到偏下方，更常見的是直接寫在另一個資料夾

:::tip
手機 -> < 768   
ipad畫面解析度 -> 768 * 1024  
桌機 -> >=1200  
更大桌機 -> >= 1440  
:::



```markdown
> @media裝置類型 and (條件一) and (條件二) {
>     符合條件的 css
> }
> 
> @media screen and (min-width: 768px) {   -> 設定最小寬度，也就是目標大於這個寬度
>     .box {                                  就會顯示下面的規則(768px稱為斷點)
>         background-color: blueviolet;
>     }
> }
```

由小寫到大： 手機 > 平板 > 桌機 -- 老師習慣     
由大寫到小： 手機 < 平板 < 桌機     




### 6-1、media詳細設定

```markdown
> .box {
>     width: 300px;
>     height: 300px;
>     background-color: antiquewhite;     -> < 768px 這個顏色
> }                                       
> 
> @media screen and (min-width: 768px) {
>     .box {
>         background-color: blueviolet;   -> 768px < x <  1024px 這個顏色
>     }
> }
> 
> @media screen and (min-width: 1024px) {
>     .box {
>         background-color: burlywood;   -> 1024px < x <  1200px 這個顏色
>     }
> }
> 
> @media screen and (min-width: 1200px) {
>     .box {
>         background-color: tomato;      -> 1200px < x <  1440px 這個顏色
>     }
> }
> 
> @media screen and (min-width: 1440px) {
>     .box {
>         background-color: gainsboro;   -> > 1440px 這個顏色
>     }
> }
```



### 6-2、實戰測試 - 建立6個物件在不同裝置上的模式

```HTML
<div class="box">
    <div class="item">1</div>
    <div class="item">2</div>
    <div class="item">3</div>
    <div class="item">4</div>
    <div class="item">5</div>
    <div class="item">6</div>
</div>
```

```markdown
> .box {
>     /* width: 100%; */
> }
> 
> .item {                                 -> 手機版面的樣式
>     /* width: 300px; */
>     height: 300px;
>     background-color: antiquewhite;
>     
> }

> @media screen and (min-width: 768px) {  -> 平板版面的樣式
>     .box {
>         display: flex;                  -> 平板的要折行
>         flex-wrap: wrap;
>         margin: auto;
>     }
>     .item {
>         width: 33.3333%;                -> 三個在一排
>         background-color: goldenrod;
>     }
> }
> 
> @media screen and (min-width: 1200px) { -> 電腦版面的樣式
>     .box {
>         display: flex;                  -> 電腦的要一整排在一行
>         flex-wrap: nowrap;
>         
>     }
> 
>     .item {
>         width: 16.666666%;
>         background-color: aqua;
>     }
> }
```



### 6-3、用自訂變數來修改剛剛計算item寬度的方式


```markdown
> body {
>     --column: 1                                -> 在一開始先定義column自訂變數
> }                                                 column: 1 欄
> 
> .box {
>     /* width: 100%; */
> }
> 
> .item {
>     /* width: 300px; */
>     height: 300px;
>     background-color: antiquewhite;
>     padding-right: 50px;
>     margin: 15px;
>     border: 10px solid black;
>     box-sizing: border-box; 
>     
> }
> 
> @media screen and (min-width: 768px) {
>     .box {
>         display: flex;
>         flex-wrap: wrap;
>         margin: auto;
>     }
>     .item {
>         /* width: 33.3333%; */
>         --column: 3;                              -> 更改 column 的數字
>         background-color: goldenrod;   
>         width: calc(100% / var(--column) - 30px);  
>                                                   -> 用 100% / 變數 - margin
>     }
> }
> 
> @media screen and (min-width: 1200px) {
>     .box {
>         display: flex;
>         flex-wrap: nowrap;
>         
>     }
> 
>     .item {
>         /* width: 16.666666%; */
>         --column: 6;
>         background-color: aqua;
>         width: calc(100% / var(--column) - 30px);
>     }
> }
```


### 6-4、gird + RWD

```markdown
> body {
>     --column: 1
> }
> 
> 
> .item {
>     height: 300px;
>     background-color: antiquewhite;
>     padding-right: 50px;
>     margin: 15px;
>     border: 10px solid black;
>     box-sizing: border-box; 
>     
> } 

> @media screen and (min-width: 768px) {
>     .box {
>         display: grid;                         -> 直接在平板最小值設定grid
>         grid-template-columns: repeat(3, 1fr);    就可以套用到桌機設定
>         margin: auto;
>     }
>     .item {
>         --column: 3;                      
>         background-color: goldenrod;
>     }
> }
> 
> @media screen and (min-width: 1200px) {
>     .box {
>         grid-template-columns: repeat(6, 1fr);   
>     }
> 
>     .item {
>         --column: 6;
>         background-color: aqua;
>     }
> }
```


7、格線系統 grid system
------

最常見的分成欄位 -> 1\2\3\4\6 欄 -> 最小公倍數為12
```md
> 100% / 12 = 8.333333%
> col 1 => 分成 1 欄的版面 -> 8.333333%
>       =>     2         -> 16.66666%
>       =>     3         -> 以此類推
```




### 7-1、屬性選取器

如果某些class或者id有相同的屬性，只要使用屬性選取器，並把共同屬性寫在一起，就可以做到很乾淨的code(記得名字要取抓共同的)

```markdown
> [class*="col"] {
>     padding-left: 15px;
>     padding-right: 15px;
>     box-sizing: border-box;
> }
> 
> .col-1 {
>     width: 8.333333%
> }
> .col-2{
>     width: 16.666666%
> }
> .col-3 {
>     width: 25%
> }
```


### 7-2、格線系統實戰

完整檔案在這邊 - gird_system.html   

(1) css 取名      
(2) 平板 -> 直寬 md     
(3) 平板 -> 橫寬 lg     
(4) 桌機 -> xl    


#### 7-2-1、第一種，直接針對子層物件給寬度
```markdown
> [class*="col"] {                    -> 這個是屬性選取器
> 
>     padding: 15px;
>     box-sizing: border-box;
> }
> 
> .col-1  {  width: 8.333333%   }
> .col-2  {  width: 16.666666%  }
> .col-3  {  width: 25%         }
> .col-4  {  width: 33.333333%  }
> .col-5  {  width: 41.666666%  }
> .col-6  {  width: 50%         }
> .col-7  {  width: 58.333333%  }
> .col-8  {  width: 66.666666%  }
> .col-1  {  width: 8.333333%   }
> .col-9  {  width: 75%         }
> .col-10 {  width: 83.333333%  }
> .col-11 {  width: 91.666666%  }
> .col-12 {  width: 100%        }
```



#### 7-2-2、第二種，直接用父層控制子層物件的寬度

```markdown
> [class*="cols-"] > .col {
>     padding: 20px;
>     box-sizing: border-box;
> }
> 
> .rows-cols-1 .col { width: 100%       }
> .rows-cols-2 .col { width: 50%        }
> .rows-cols-3 .col { width: 33.333333% }
> .rows-cols-4 .col { width: 25%        }
> .rows-cols-5 .col { width: 20%        }
> .rows-cols-6 .col { width: 16.666666% }
```


:::tip
兩種各有不同好處  
第二種比較直觀  
但是第一種比較彈性(因為是直接針對子層物件給寬度)  
:::


### 7-3、實際練習 - 方法二

```markdown
> HTML

> <div class="container">
>     <div class="row rows-cols-1 rows-cols-md-3 rows-cols-lg-4 > rows-cols-xl-6">
>         <div class="col">
>             <div class="item">
>                 也是公里深深新竹包含網易主人說什麼，討論區講話合作跟着來源前往同學天空可> 見一本台北，製品交通周邊收入理解正是透明傳說成人，如下金錢他也媽媽供求不> 住前往討論需要身體整合重視文件今年，什麼樣當初放下壓縮想起也要告訴你，以> 為那麼經過一年，至少醫生選單互動，出台。
>             </div>
>         </div>
>         <div class="col">
>             <div class="item">
>                 也是公里深深新竹包含網易主人說什麼，討論區講話合作跟着來源前往同學天空可> 見一本台北，製品交通周邊收入理解正是透明傳說成人，如下金錢他也媽媽供求不> 住前往討論需要身體整合重視文件今年，什麼樣當初放下壓縮想起也要告訴你，以> 為那麼經過一年，至少醫生選單互動，出台。
>             </div>
>         </div>
>         <div class="col">
>             <div class="item">
>                 也是公里深深新竹包含網易主人說什麼，討論區講話合作跟着來源前往同學天空可> 見一本台北，製品交通周邊收入理解正是透明傳說成人，如下金錢他也媽媽供求不> 住前往討論需要身體整合重視文件今年，什麼樣當初放下壓縮想起也要告訴你，以> 為那麼經過一年，至少醫生選單互動，出台。
>             </div>
>         </div>
>         <div class="col">
>             <div class="item">
>                 也是公里深深新竹包含網易主人說什麼，討論區講話合作跟着來源前往同學天空可> 見一本台北，製品交通周邊收入理解正是透明傳說成人，如下金錢他也媽媽供求不> 住前往討論需要身體整合重視文件今年，什麼樣當初放下壓縮想起也要告訴你，以> 為那麼經過一年，至少醫生選單互動，出台。
>             </div>
>         </div>
>         <div class="col">
>             <div class="item">
>                 也是公里深深新竹包含網易主人說什麼，討論區講話合作跟着來源前往同學天空可> 見一本台北，製品交通周邊收入理解正是透明傳說成人，如下金錢他也媽媽供求不> 住前往討論需要身體整合重視文件今年，什麼樣當初放下壓縮想起也要告訴你，以> 為那麼經過一年，至少醫生選單互動，出台。
>             </div>
>         </div>
>         <div class="col">
>             <div class="item">
>                 也是公里深深新竹包含網易主人說什麼，討論區講話合作跟着來源前往同學天空可> 見一本台北，製品交通周邊收入理解正是透明傳說成人，如下金錢他也媽媽供求不> 住前往討論需要身體整合重視文件今年，什麼樣當初放下壓縮想起也要告訴你，以> 為那麼經過一年，至少醫生選單互動，出台。
>             </div>
>         </div>
>     </div>
> </div>
```

```markdown

> [class*="cols-"] > .col {
>     padding: 20px;
>     box-sizing: border-box;
> }
> 
> .rows-cols-1 .col { width: 100%       }
> .rows-cols-2 .col { width: 50%        }
> .rows-cols-3 .col { width: 33.333333% }
> .rows-cols-4 .col { width: 25%        }
> .rows-cols-5 .col { width: 20%        }
> .rows-cols-6 .col { width: 16.666666% }
> 
> /* 768~1024  md */
> @media screen and (min-width: 768px) {
>     .rows-cols-md-1 .col { width: 100%       }
>     .rows-cols-md-2 .col { width: 50%        }
>     .rows-cols-md-3 .col { width: 33.333333% }
>     .rows-cols-md-4 .col { width: 25%        }
>     .rows-cols-md-5 .col { width: 20%        }
>     .rows-cols-md-6 .col { width: 16.666666% }
> }    
> 
> /* 1024~1200  lg */
> @media screen and (min-width: 1024px) {
>     .rows-cols-md-1 .col { width: 100%       }
>     .rows-cols-md-2 .col { width: 50%        }
>     .rows-cols-md-3 .col { width: 33.333333% }
>     .rows-cols-md-4 .col { width: 25%        }
>     .rows-cols-md-5 .col { width: 20%        }
>     .rows-cols-md-6 .col { width: 16.666666% }
> }    
> 
> 
> 
> /* 1200  xl */
> @media screen and (min-width: 1200px) {
>     .rows-cols-md-1 .col { width: 100%       }
>     .rows-cols-md-2 .col { width: 50%        }
>     .rows-cols-md-3 .col { width: 33.333333% }
>     .rows-cols-md-4 .col { width: 25%        }
>     .rows-cols-md-5 .col { width: 20%        }
>     .rows-cols-md-6 .col { width: 16.666666% }
> } 
```



