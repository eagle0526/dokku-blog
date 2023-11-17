---
title: ROR/HTML&CSS(06)
sidebar_label: "DAY11. [ASTROCamp] HTML&CSS-6"
description: HTML&CSS
last_update:
  date: 2022-10-18
keywords:
  - 5xRuby
  - HTML
  - CSS
sidebar_position: 12
---


1、relative實戰練習 - div區塊向上偏移做法
------

```HTML
<div class="box"></div>
```

```markdown
> .box {
>     width: 200px;
>     height: 200px;
>     background-color: aquamarine;
>     position: relative;            -> 相對定位
>     top: 0px;
>     margin: 30px; 
> }
> 
> .box:hover {                       -> hover效果
>     top: -10px;                    -> 往上移動10px
>     box-shadow: 0px 10px 0px red;  -> 移動後產生陰影
> }
```


2、自訂屬性
------

(1) 子層可以拿父層的自訂屬性    
(2) 父層不能拿子層的自訂屬性    
(3) 如果在兩個區塊都訂了同個變數名稱，只會在自己區塊內顯示該名稱    
(4) 這裡的變數名稱設定也有 scope的概念  

```markdown
> .box {
>     --box-size: 300px;          -> 自定義屬性 -> 有套用到此屬性的區塊都是300px
>     width: var(--box-size);     -> width  為 300px
>     height: var(--box-size);    -> height 為 300px
>     background-color: beige;
> }
```

### 2-1、在body設定完所有自定義變數
:::tip
所以通常在定義變數的時候  
通常會在body or HTML 設定完所有的變數，並在子層想要用這些變數的時候直接取用，如果今天子層想要更改變數，直接在     
子層用同變數並更改屬性設定，在取用就OK    
:::


3、calc 動態計算
------

假設今天我想要讓 **整個區塊的右側靠在在畫面中間線** ，並且左側要給 30px，可以像畫面這樣設定
```css
.box {
    width: calc(50% - 30px);        -> 減法中記得要給空格，要不然可能會發生錯誤
    margin-left: 30px;
    height: 300px;
    background-color: red;
}
```



4、grid
------
(1) gird跟表格很像，先寫row再寫column，先寫橫線，再寫直線s   
(2) 都是先畫出表格，在把東西放進去  
(3) grid的格子可以做得跟父層空間不一樣大    
(4) 建好gird後，裡面的item都會放在框格裡面  


### 4-1、可以用repeat來寫
```markdown
> repeat(4, 100px)
```

### 4-2、可以控制格子的對齊與分佈

把格子畫出來後，可以控制格子的對齊與分佈   


#### 4-2-1、justify-content
可以用justify-content => 把格子放在整個 **空間** 左右的開頭、中間、尾端或分佈   


#### 4-2-2、justify-items
justify-items => 控制格 **子裡面物件** 的對齊與分佈(左右對齊)


#### 4-2-3、如果不給子物件寬高的話，內容物就會跟格子一樣大

#### 4-2-4、align-content
align-content => 控制格子上下的 **空間** 對齊與分佈

#### 4-2-5、align-items
align-items   => 控制格 **子裡面物件** 的對齊與分佈(上下對齊)


#### 4-2-6、gap -> 表格間的距離
```markdown
> gap: 格子與格子的距離
> gap: 10px 20px;        -> 第一個是column-gap, 第二個是row-gap
> row-gap -> 欄之間gap
> column-gap -> 列之間gap
```


### 4-3、grid子層設定

如果今天沒有設定子項目的寬跟高，寬高會被內容撐開
```markdown
> grid-row: 1 / 3           => row第一條直線~第三條直線
> grid-column: 1 / 2        => column第一條直線~第二條直線
> 
> grid-area: 1 / 1 / 3 / 2  => 給出兩點，讓他形成一個矩形
> 
>           row column row column
>           第一個格子的點~第二的格子的點形成的圖
> 
> 下圖是顯示在網頁f12的使用者視窗，點擊grid區塊後會顯示的格線
>  1   2  3  4 
> 1          -4
> 2          -3
> 3          -2
> 4          -1
>  -4 -3 -2 -1
```


### 4-4、grid注意事項

(1) 如果今天沒用到定位，也可以使用grid來排版，因為grid自帶z-index   
(2) 如果今天gird 有兩個圖層會重疊，只要把想要顯示在外面的那個區塊 z-index 調高就可以成功    


下面這樣寫，可以讓item1 蓋住 item2，如果今天兩個都沒設定z-index，下面會蓋掉上面的
```markdown
> .item:nth-child(1) {
>     grid-area:1/1 / 3/4;
>     z-index: 2  ;                  - z-index 設定2
> }
> 
> .item:nth-child(2) {
> 
>     grid-area: 2/1 / 4/4;
>     background-color: bisque;
> }
```



### 4-5、grid實戰練習 - 做一個三欄兩列產品顯示

完整code在day11的shop-grid裡面


```markdown
> .section {
>     margin: 0px auto;
>     width: 960px;
>     background-color: antiquewhite;          
>     display: grid;                             -> 使用grid                    
>     grid-template-columns: repeat(3, 1fr);     -> 給三個欄位，給完後會把之後的item放進裡面
>     gap: 20px;                                 -> item的間格
> 
>     padding: 20px;
> }
> 
> .card {
>     /* width: 270px; */                        -> 不用給寬度，因為grid的特性，沒width的情況下，會值填滿該格
>     border: 1px solid tomato;
>     /* margin: 10px; */                        -> 有gap了，不用margin
>     padding: 10px;
>     background-color: beige;
> }
> 
> img {
>     width: 100%;
> }
```




5、卡片實戰練習 - 製作文字在圖片中間
------

```HTML
<div class="item">
    <div class="pic">
        <img src="https://fakeimg.pl/300x200/200">
    </div>
    <div class="txt">
        <h2>title</h2>
        <p>Lorem, ipsum dolor sit amet consectetur adipisicing elit. Repellat sed quisquam, q
        uasi doloribus pariatur ratione adipisci voluptates. Temporibus nam quibusdam possim
        us libero aut culpa, cumque hic, ut blanditiis eum distinctio.</p>
    </div>
</div>
```



```markdown
> * {
>     margin: 0px;
>     padding: 0px;
> }
> 
> .item {
>     width: 400px;
>     border: 1px solid tomato;
>     margin: 20px;
>     position: relative;              -> 文字的父層，把文字限制在該層區域
> 
> }
> 
> .pic img {
>     width: 100%;
>     vertical-align: top;             -> 這個一定要記得設定，要不然圖片下方會有一塊空白
> }
> 
> .txt {
>     position: absolute;              -> 文字絕對定位
>     top: 0px;
>     bottom: 0px;
>     left: 0px;
>     right: 0px;
>     color: aliceblue;
>     padding: 30px;
>     opacity: 0;                      -> 透明度0(隱藏文字)
>     transition: 1s;
>     background-color: rgba(109, 37, 37, 0.5);
> 
> }
> 
> .txt:hover {
>     opacity: 1;                      -> hover，滑鼠移過來後透明度變1
> }
```



