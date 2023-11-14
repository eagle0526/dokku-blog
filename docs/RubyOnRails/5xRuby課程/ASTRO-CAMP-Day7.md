---
title: ROR/HTML&CSS(04)
sidebar_label: "DAY7. [ROR] HTML&CSS-4"
description: HTML&CSS
last_update:
  date: 2022-10-12
keywords:
  - 5xRuby
  - HTML
  - CSS
sidebar_position: 8
---




1、css心智圖
------

### 1-1、文字
 - 字型 font-family
 - 大小 font-size
 - 樣式 font-style
 - 色彩 color
 - 裝飾 text-decoration

### 1-2、背景
 - 背景色 background-color
 - 背景圖 background-image
 - 背景圖尺寸 background-size
 - 背景圖位置 background-position
 - 背景圖重複 background-repeat
 - 背景圖固定方式 background-attachment

### 1-3、位置
 - position
 - top
 - right
 - bottom
 - left
 - z-index

### 1-4、尺寸
 - 背景尺寸 background-size
 - 物件尺寸 width、height
 - 邊框線尺寸 border-width
 - 文字尺寸
 - 內距
 - 外距

### 1-5、邊框 
 - border
 - outline

### 1-6、陰影
 - box-shadow
 - text-shadow

### 1-7、圓角
 - border-radius

### 1-8、段落
 - 行高 line-height
 - 首行縮排 text-indent

### 1-9、排版
 - flex
  - 父層 display、align-items  
      - (flex-direction、flex-wrap) = flex-flow(前面兩個屬性可以用後面此屬性代替)
      - (justify-content、align-content) = place-content(前面兩個屬性可以用後面此屬性代替)
  - 子層 align-self、order
      - (flex-grow、flex-shrink、flex-basis) = flex(前三個屬性可以用後面此術性代替)
 - float
 - grid

### 1-10、變形
 - transform
 - transform-origin
 - perspective
 - transform-style

### 1-11、動畫


2、切版小提醒
------

(1) 如果今天對一個物件設定transform rotate，但是他沒有改變，大機率是因為該物件為inline特性  
(2) 設定圖片排版的時候，絕對要記得 vertical-align: center -> 圖片對齊的設定，如果對齊base的話畫面會差幾像素     
(3) 調整p段落的margin，建議使用em這個單位，因為如果使用px，更改p內容的字體大小，間距不會更改，用em的話會一起更改    


3、align-content、align-items、justify-content間的差異
------

使用這兩個物件是，先想這四件事
 - 對象是誰
 - 參考空間是誰
 - 哪個軸向
 - 作用是

### 3-1、align-content 

align-content是設定你的 **彈性列**，在 **父層空間** **交叉軸** 上的 **對齊或分佈**  
在空間中有一個虛擬的框框，經由flex wrap換行後(多換一行就會多一個框框)，align-item就是在控制這些框框的位子。     

```md
> align-content屬性 (基本上他的效果跟justify-content很像，不過作用對象不一樣)
> - align-content: flex-start; 
> - align-content: flex-end; 
> - align-content: center; 
> - align-content: space-around; 
> - align-content: space-between;
> - align-content: stretch;        - 虛擬框框填滿整個空間後平分給每個列 - 預設值
```

### 3-2、align-items

align-items是設定你的 **子物件**，在 **彈性列** 中 **交叉軸** 上的 **位置**     

```md
> align-items屬性 (操作子物件的中，在彈性列的位子)
> - align-items: flex-start; 
> - align-items: flex-end; 
> - align-items: center; 
> - align-items: baseline; -> 沒啥在用，不用理他
```

### 3-3、justify-content

justify-content是設定你的 **子物件** 在 **父層空間** 中 **主軸** 的 **對齊與分佈**  



4、特殊選取器 nth-child(an+1)
------

(1) nth-child(an+b)     
(2) (an+b) - a個裡面的第b個     

```HTML
<ul>
    <li>1</li>
    <li>2</li>
    <li>3</li>
    <li>4</li>
    <li>5</li>
</ul>
```
```CSS
<style>
    .child:nth-child(3) {          -> 這樣就會抓到第三個物件
        background-color: red;
    }
</style>
```



### 4-1、情境題一

假設今天有一個設計師，給你的版面是一個商品清單，像下面，設計師想要你幫他把中間那個商品對兩旁設定間距

```markdown
> 口<口>口
> 口<口>口
> 口<口>口
> 口<口

> 做到的方法就是，使用nth.child(3n+2)，抓取到三個中的第二個，選到後再使用margin往旁邊推

> <style>
>     .child:nth-child(3n+2) {
>         margin: 0px 10px;
>     }
> </style>
```

### 4-2、情境題二

設計師給你的圖像下方，想要把商品列表做成下方的排版

```markdown
> 口口口
>  口口
> 口口口
>  口口
> 口口口
> 
> nth-child(5n+4) {    => 針對第四個物件，往左邊設定margin，
>                         因為wrap的關係，會把地6個物件往下一行放
>     margin-left: 50px;
> }
> 
> nth-child(5n) {
>     margin-right: 50px;    => 再把第五個物件設定margin-right
> }                             設定這個關係是因為如果不設定，置中慧偏移
>                               原因是因為，第四個物件左邊設定了left
>                               如果右邊不設定，這個空間的剩餘空間會不平衡
> ul {
>     justify-content: center;  => 讓第四個第五個框框置中
> }

> ---實際的code


> <ul>
>     <li>1</li>
>     <li>2</li>
>     <li>3</li>
>     <li>4</li>
>     <li>5</li>
>     <li>6</li>
>     <li>7</li>
>     <li>8</li>
>     <li>9</li>
>     <li>10</li>
>     <li>11</li>
>     <li>12</li>
>     <li>13</li>
>     <li>14</li>
>     <li>15</li>
>     <li>16</li>
>     <li>17</li>
>     <li>18</li>
>     <li>19</li>
>     <li>20</li>
> </ul>


> ul {
>     width: 400px;
>     display: flex;
>     flex-wrap: wrap;
>     justify-content: center;
> }
> 
> li {
>     margin: 10px;
>     width: 100px;
>     height: 100px;
>     background-color: #666;
>     list-style-type: none;
>     color:black;
> }        
> 
> ul li:nth-child(5n+4) {
>     background-color:blueviolet;
>     margin-left: 50px;
> }
> 
> ul li:nth-child(5n) {
>     background-color:blueviolet;
>     margin-right: 50px;
> }
> 
> </style>
```


### 4-3、情境題三

現在有20個物件，設計師要你針對15~20個物件設定
```markdown
> <style>
>     li:nth-child(n+15):nth-child(-n+20)  -> 選取15個之後，20物件之前
> </style>
```

:::tip
快速生出5區塊，中間會有1~5數值    
快捷鍵使用 -> `div*5>{$}`   
:::  



