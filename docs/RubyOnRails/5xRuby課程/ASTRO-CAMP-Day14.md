---
title: ROR/HTML&CSS(07)
sidebar_label: "DAY14. [ASTROCamp] HTML&CSS-7"
description: HTML&CSS
last_update:
  date: 2022-10-24
keywords:
  - 5xRuby
  - HTML
  - CSS
sidebar_position: 15
---


1、五倍官網code review
------
  
1. 導覽列的語意標籤是 - nav，記得要用  
  
2. 假設文字不會很長，可以用一個h3包起來，然後裡面是兩塊span，然後裡面的兩個span設定成兩個block，這樣兩個span就可以佔滿兩行  
  
3. 製作超連結按鈕的時候，可以盡量用 block 把整個寬度撐開，這樣做比較好，就像五倍官網的更多報導的按鈕，我只設定了padding，只有把 a 的外圍撐開，但文字實際的寬高還是沒變。ps. fit-content => 可以針對 a 的 width 設定此屬性，非常好用!!  
  
4. ::before、::after 預設為inline，所以會在父層內容的前面，如果用block，可以變成佔滿整塊的空間  
  
5. 字體下面的紅底線(::before、::after)，直接用margin auto置中就好，不用用絕對定位一直調(並且用block就好，不需用到絕對定位)  
  
6. 如果今天hover後才跑出陰影，建議hover前的物件、hover後的物件，定位都要寫，比較好控制 (.box, .box:hover => 建議兩邊都用box-shadow)  
  
7. 千萬不要寫死高度，如果今天把高度寫死，在RWD會出大事  
  
8. 記得要在body層，寫一個h1、h2，這樣SEO才會吃到你的title  
  
9. **選單** 千萬不要寫死寬度，這樣以後在新增選單或刪減選單的時候不用一直更改  
  
10. 選單的架構 -> nav > navbar > ul > li > a  
ps. li把a包住，撐開選單連結的寬度，a寫在最裡面(就可以把li撐開了)  
  
11. gap 可以用在flex、grid  
  
12. 父層和子層間，盡量用padding，少用margin-top，會產生margin collapse的問題  
  
13. 圖片高度寬高等比例  
法一: img 寬度 100%，高度 auto，就可以讓  
法二: aspect-ratio: 2/1  
  
14. [網站的favicon增加的方法](https://ithelp.ithome.com.tw/articles/10285383)   


2、transform - 選單特效
------

```HTML
<div class="box"></div>
```

```markdown
> .box{
>     width: 300px;
>     height: 300px;
>     background-color: red;
>     transform: scaleX(0.3);  - 原本規模為原本寬度的0.3
>     transition: 1s; - 動畫時間 1s
> }
> 
> .box:hover {
>     transform: scaleX(1); - 碰到後變成 1
> }
```




3、下拉選單標準製作方式 - navbar
------

在day14的transition_menu.html可以看到完整畫面

```HTML
<nav class="main-nav">
    <ul>
        <li><a href="">link</a></li>
        <li><a href="">link</a></li>
        <li><a href="">link</a></li>
        <li><a href="">link</a></li>
        <li><a href="">link</a></li>
    </ul>
</nav>
```


```markdown
> .main-nav {
>     width: 100%;
>     background-color: antiquewhite;
> }
> 
> ul {
>     display: flex;            -> 讓 li 水平排列
>     list-style: none;         -> 不要ul的點點
>     
> }
> 
> a {
>     outline: 1px solid white;
>     display: block;           -> 預設是inline，因此要撐開超連結空間
>     padding: 20px 30px 0px;   -> 撐開的寬度、高度
>     text-decoration: none;    -> 不要超連結底線
> }
> 
> a::after {
>     content: "";
>     display: block;           -> 撐開寬度
>     width: 100%;              -> 跟父層(文字區塊)等寬
>     height: 5px;              -> 給高度
>     background-color: blue;
>     margin-top: 10px;         -> 讓下底線離文字遠一點
>     
>     transform: scale(0);      -> 還沒觸碰的狀態不要出現
>     transition: 0.5s;
> }
> 
> a:hover::after {
>     transform: scale(1);      -> 觸碰之後跑出來
> }
```


4、object-fit 圖片設定
------
```md
> object-fit ->  當你圖片寬高都給100%的時候，會適當的裁切整張圖片(就是wordpress圖片 cover效果)
> object-position -> 定位要切的位置
```





