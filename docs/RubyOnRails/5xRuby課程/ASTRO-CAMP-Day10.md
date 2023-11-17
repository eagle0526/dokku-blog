---
title: ROR/HTML&CSS(05)
sidebar_label: "DAY10. [ASTROCamp] HTML&CSS-5"
description: HTML&CSS
last_update:
  date: 2022-10-17
keywords:
  - 5xRuby
  - HTML
  - CSS
sidebar_position: 11
---





1、order屬性
-----

(1) 所有子物件的order值都預設為0  
(2) 只要要給一個order值，他就會跑到後面去  
(3) 排序位置，跟主軸方向有關  

### 1-1、實際測試 - 把div 2 換到最右邊

```HTML
<div class="flex">
    <div class="item">1</div>
    <div class="item">2</div>
    <div class="item">3</div>
</div>
```

```css
.flex {
    width: 500px;
    height: 500px;
    background-color: aquamarine;
    display: flex;
    flex-wrap: wrap;
}
.item {
    width: 100px;
    height: 100px;
    margin: 10px;
    background-color: aliceblue;
    font-size: 20px;
}
.item:nth-child(2) {
    background-color:blanchedalmond ;
    order: 1;
}
```


2、flex續
------

### 2-1、flex-grow

(1) 伸展值  
(2) 拿取所有剩餘空間  
(3) 誰擁有flex grow 越多，就可以分到越多剩餘空間  

```markdown
> .item:nth-child(1) {
>     flex-grow: 1;          -> 剩餘空間拿1份
> }
> 
> .item:nth-child(3) {
>     flex-grow: 2;          -> 剩餘空間拿兩份
> }
```

### 2-2、flex-shrink

(1) 縮減值  
(2) 每個物件都有flex-shrink屬性  
(3) 如果設定 flex-shrink 設定為0，flex 就會失去收縮  


```HTML
<div class="flex">
    <div class="item">1</div>
    <div class="item">2</div>
    <div class="item">3</div>
</div>
```

```css
.flex {
    width: 500px;
    height: 500px;
    background-color: aquamarine;
    display: flex;
    /* flex-wrap: wrap; */              -> 把flex換行關掉 
}

.item {
    width: 200px;
    height: 100px;
    margin: 10px;
    background-color: aliceblue;
    font-size: 20px;
    flex-shrink: 0;                     -> 讓三個物件的伸縮值變0(會突破父層的寬度)
}

.item:nth-child(2) {
    background-color:blanchedalmond ;
    flex-shrink: 1;                     -> 讓子層的第二個div，flex值設定為1，該div值會收縮到整個可以被父層包住
}
```


### 2-3、flex-grow + max-width + justify-content

**另一種比較少見的排版法 - 不用算寬度的寫法**

```HTML
<div class="flex">
    <div class="item">1</div>
    <div class="item">2</div>
    <div class="item">3</div>
</div>
```


```css
.flex {
    /* width: 500px; */              -> 父層不設定高度
    height: 500px;
    background-color: aquamarine;
    display: flex;
    /* flex-wrap: wrap; */           -> 不要讓子物件換行
    justify-content: space-evenly;   -> 讓子層物件置中
}

.item {
    width: 0px;                      -> 每個子物件不要設定寬度
    height: 100px;
    background-color: aliceblue;
    font-size: 20px;
    flex-grow: 1;                    -> 讓每個物件都可拿取剩餘寬度
    max-width: 400px;                -> 限制子層的最大寬度
}

.item:nth-child(2) {
    background-color:blanchedalmond ;
}

.item:nth-child(3) {
    background-color:brown ;
}
```



### 2-4、flex-basis

基本值是子物件在主軸上的長度  
ps. 主軸是左到右的時候，基本值是寬、主軸是上到下的時候，基本值是高度  

**basis跟寬高衝突的時候，basis優先**  

:::tip
flex-grow、flex-shrink 是不衝突的，兩種作用情境不同  
:::



3、後台管理的切版設定 
------


```markdown
> HTML

> <div class="flex">
>     <div class="item">側邊欄</div>    -> 左側固定側邊欄
>     <div class="item">工作列</div>    -> 右側彈性工作列
> </div>
```



### 3-1、方法一
(1) 一個父層 100%  
(2) 左邊側欄固定寬 300px  
(3) 右側寬度為0 - w: 0px + flex-grow: 1  (右側把剩下寬度都吃掉)  


```css
.flex {
    width: 100%;
    height: 500px;
    background-color: aquamarine;
    display: flex;
}

.item:nth-child(1) {
    background-color:blanchedalmond ;
    width: 300px;
    outline: 1px solid tomato;
}

.item:nth-child(2) {
    background-color:blanchedalmond ;
    outline: 1px solid brown;
    width: 0%;
    flex-grow: 1;
}
```


### 3-2、方法二

(1) 父層100%  
(2) 左側固定寬 200px - flex-shrink: 0; -> 不收縮    
(3) 右側100% - flex-shrink: 1; -> 收縮      

```css
.flex {
    width: 100%;
    height: 500px;
    background-color: aquamarine;
    display: flex;
}

.item:nth-child(1) {
    background-color:blanchedalmond ;
    width: 300px;
    outline: 1px solid tomato;
    flex-shrink: 0;
}

.item:nth-child(2) {
    background-color:blanchedalmond ;
    outline: 1px solid brown;
    width: 100%;
    flex-shrink: 1;
}
```




:::info
寬、高度優先 - 當多種寬度設定衝突時，哪個優先權最高  
min-width > max-width > flex-basis > width  
:::




4、position
-----

(1) static      -> 靜態 -> 把所有定位無效化(超級少用到)  
(2) relative    -> 相對定位 -> 偏移顯示  
(3) fix         -> 固定定位  
(4) absolute    -> 絕對定位  
(5) sticky      -> 黏貼  


```markdown
> <div class="flex">
>     <!-- .item.item${$}*40 -->           -> 下面所有東西的快速鍵
>     <div class="item item1">1</div>
>     <div class="item item2">2</div>
>     <div class="item item3">3</div>
>     <div class="item item4">4</div>
>     <div class="item item5">5</div>
>     <div class="item item6">6</div>
>     <div class="item item7">7</div>
>     <div class="item item8">8</div>
>     <div class="item item9">9</div>
>     <div class="item item10">10</div>
>     <div class="item item11">11</div>
>     <div class="item item12">12</div>
>     <div class="item item13">13</div>
>     <div class="item item14">14</div>
>     <div class="item item15">15</div>
>     <div class="item item16">16</div>
>     <div class="item item17">17</div>
>     <div class="item item18">18</div>
>     <div class="item item19">19</div>
>     <div class="item item20">20</div>
>     <div class="item item21">21</div>
>     <div class="item item22">22</div>
>     <div class="item item23">23</div>
>     <div class="item item24">24</div>
>     <div class="item item25">25</div>
>     <div class="item item26">26</div>
>     <div class="item item27">27</div>
>     <div class="item item28">28</div>
>     <div class="item item29">29</div>
>     <div class="item item30">30</div>
>     <div class="item item31">31</div>
>     <div class="item item32">32</div>
>     <div class="item item33">33</div>
>     <div class="item item34">34</div>
>     <div class="item item35">35</div>
>     <div class="item item36">36</div>
>     <div class="item item37">37</div>
>     <div class="item item38">38</div>
>     <div class="item item39">39</div>
>     <div class="item item40">40</div>
> </div>
```




### 4-1、relative 相對定位

top - 要距離原本物件的上緣多少  
left - 要距離原本物件的左緣多少      

如果今天同時設定left、right，系統會依照你語言文字的方向，來決定哪一個要作用     
ex.如果是左~右，left會觸發，right會無效     


```css
.flex {
    display: flex;
    flex-wrap: wrap;
}
.item {
    width: 100px;
    height: 100px;
    background-color: gainsboro;
    margin: 5px;
    font-size: 20px;
}
.item20 {
    background-color:brown ;
    position: relative;
    top: 100px;
    left: 100px;
}
```

:::tip
**物件排序規則**  
1、如果今天有一個物件設定了定位，會蓋住沒定位的物件  
2、有定位的物件，z-index會比沒定位的還高  
3、如果兩者都有定位，會依照原始碼的順序  
:::


:::info
**只要Z軸調不了，就要想到這幾個事情**  
1、 只要你的定位**不是static**，你的z-index都有效  
2、 當你的父層是flex 你的 z-index都有效  
3、 當你的父層是grid，也是有效  
4、 擁有opacity，z-index是比較高的   
:::



### 4-2、fixed

(1) 設定fixed的物件，會脫離資料流去排列，而他參考的空間是視窗   
(2) 如果今天沒有設定left、top，會依據目前位置黏在視窗上     
(3) 如果有設定left top，就會從視窗左上角來開始排列   


*由於資料量太多->這邊只打上3個box，其他上下都是一堆的lorem
```markdown
> HTML

> <div class="box box1">
>     box1
>     <div class="box box2">
>         box2
>         <div class="box box3">box3</div>
>     </div>
> </div>
```


```markdown
> .box1 {
>     border: 10px solid tomato;
>     padding: 30px;
> }
> .box2 {
>     border: 10px solid tomato;
>     padding: 30px;
> }
> 
> .box3 {
>     width: 100px;
>     height: 100px;
>     background-color: gainsboro;
>     position: fixed;                -> 設定 fixed
>     top: 100px;                     -> 視窗最左上角開始算
>     left: 0px;                      -> 視窗最左上角開始算
> }
```

#### 4-2-1、fixed地雷點

(1) 假設今天忘記設定top、left，有可能會導致物件一直存在視窗外，會以為視窗壞掉   
(2) 假設今天對一個物件設定fixed，原本是block父層寬的話，設定fixed會變成內容寬   


#### 4-2-2、如果同時有fixed、relative物件，前面的會蓋掉後面的

這個常發生在做navbar的時候，如果發生這件事，記得對nav設定高一點z-index，讓nav層級變高就可以蓋過其他的relative

```CSS
.nav {
    width: 100%;
    background-color: beige;
    position: fixed;
    z-index: 10;
}

.box2 {
    border: 10px solid tomato;
    padding: 30px;
    position: relative;

}
```




### 4-3、absolute 絕對定位

(1) 絕對定位當你設定上右下左，會去尋找具有定位設定的父層    
(2) 他的特性和fixed很像，不過他並不會捲動，而且當上層父層都沒有定位時，他會幫物件定位在第一視窗視角(viewpoint)的地方    


```markdown
> <div class="box box1">
>     box1
>     <div class="box box2">
>         box2
>         <div class="box box3">box3</div>
>     </div>
> </div>
```


```markdown
> .box1 {
>     border: 10px solid tomato;
>     padding: 30px;
> }
> .box2 {
>     border: 10px solid tomato;
>     padding: 30px;
> }
> 
> .box3 {
>     /* width: 100%; */
>     height: 100px;
>     background-color: gainsboro;
>     position: absolute;                -> 設定絕對定位(目前父層沒有設定定位)
>     bottom: 0px;                       -> 設定第一視窗viewpoint的距離
>     right: 0px;                        -> 設定第一視窗viewpoint的距離
> }
> 
> .nav {
>     width: 100%;
>     background-color: beige;
>     position: absolute;
>     z-index: 10;
> }
```





### 4-4、sticky 定位

**sticky的特性**    
(1) 跟absolute的特性一樣，只能作用在父層空間    
(2) 脫離父層就會像 relative 一樣，被捲動捲走    

Ps. 完整code在day11.html    
```markdown
> .right {
>     width: 200px;
>     padding: 10px;
>     background-color: #fa8;
>     flex-shrink: 0;
> }
> .blue{
>     height: 200px;
>     margin-bottom: 20px;
>     background-color: #00f;
>     position: sticky;             -> 幫這個物件加上 sticky 屬性
>     top: 10px;                    -> 當此物件距離螢幕上方 20px 時，觸發此position
> }
> .red{
>     height: 200px;
>     background-color: #f00;
> }
```


5、z-index 踩雷
------
box3 在 box1 裡面、sp在box1 下面
```markdown
> <div class="box box1">
>     box1
>     <div class="box box2">
>         box2
>         <div class="box box3">box3</div>
>     </div>
> </div>
> 
> <sp>Lorem ipsum dolor sit amet consectetur adipisicing elit. Vero, ea quisquam voluptatum aperiam > facere repellendus ducimus ipsum iure quas dolorum similique iusto quo, a eius labore, saepe odit. > Quos, deleniti?</sp>
```



### 5-1、照下面這樣設定z-index，sp會蓋住box3
```markdown
> z-index
> box1 : z-index:2
> box3 : z-index:10
> sp : z-index:3
```

### 5-2、這下面這樣設定，box3會蓋住其他兩個
```markdown
> z-index
> box1 : z-index:auto
> box3 : z-index:10
> sp : z-index:3
```

如果父層是auto，子層就會跟整個頁面的z-index值比較
如果父層是值，子層就會被父層限制，跟其他區塊的z-index比較時，就要以父層的z-index為主



### 5-3、z-index負值 - 會有空間概念的問題

**如果今天box3的父層box1 的 z-index: 0，這樣box3怎麼設定都會下不去**  
```markdown
> box3 : z-index: -1
```



### 5-4、實戰演練 - 製作蝦皮官網的熱賣商品標籤

```markdown
> HTML

> <div class="card">
>     <div class="sale">熱賣商品</div>
>     <img src="https://fakeimg.pl/300x200/200">
>     <h3>tittle</h3>
>    <p>Lorem ipsum, dolor sit amet consectetur adipisicing elit.</p>
> </div>
```

```markdown
> .card {
>     width: 220px;
>     outline: 1px solid tomato;
>     margin: 20px;
>     padding: 10px;
>     position: relative;          > 父層要做定位，他的子區塊才會被限制
> }
> 
> .card img {
>     width: 100%;
> }
> 
> .card .sale {
>     background-color: aqua;
>     padding: 5px 10px;
>     position: absolute;          > sale區塊設定絕對定位，會根據父層來做定位
>     left: -5px;
>     top: 20px;
> }
```




6、製做三角形
------

### 6-1、方法一
參考day10的triangle.html檔案


### 6-2、方法二 - 使用clip-path
```markdown
> .arrow {
>     width: 300px;
>     height: 300px;
>     background-color: black;
>     clip-path: polygon(0px 0px, 300px 0px, 300px 300px)  > 這個簡單說就是每個 , 都是一個x軸和y軸點
>                                                            總共可以給四個點，看你想要怎麼做
> }
```




7、偽元素::before、::after
------

裝飾性物件都可以利用before、after來處理，像是蝦皮的三角形


```markdown
> <p>13123</p>
```

```markdown
> p {
>     background-color: antiquewhite;
> }
> 
> p::before {                         # 在p之前增加一段文字
>     content: "我好棒";                 增加content中的內容
>     color: green;                     更改文字顏色
> }
> 
> p::after {                          # 在p之後增加一段文字 
>     content: "我好爛";
>     color: gray;
> }
```



### 7-1、處理蝦皮三角形

不用增加div，直接抓到熱賣商品區塊，在該區塊加上偽元素
```markdown
> .card:nth-child(2) .sale::before {
>     content: "";
>     width: 0px;
>     height: 0px;
>     position: absolute;                         -> 絕對定位
>     left: 0px;
>     top: 100%;                                  -> 根據父層，把此區塊往下擠一整個父層高度
>     /* border-top: 10px solid gold; */          -> 從這裡開始下面四行是一個直角三角形
>     /* border-right: 10px solid ghostwhite; */
>     border-bottom: 8px solid transparent;
>     border-left: 10px solid gold;
>     
> }
```
