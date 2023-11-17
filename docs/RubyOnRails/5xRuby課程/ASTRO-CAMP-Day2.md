---
title: ROR/HTML&CSS(01)
sidebar_label: "DAY2. [ASTROCamp] HTML&CSS-1"
description: HTML&CSS
last_update:
  date: 2022-10-04
keywords:
  - 5xRuby
  - HTML
  - CSS
sidebar_position: 3
---




1、表格製作
-----

快捷鍵方式產出一堆的table、tr、td
```markdown
> 使用 table>tr*3>td*4，即可拉出下面的資料

>    <table border="3" width="500">
>        <tr>
>            <td>資料1</td>
>            <td>資料1</td>
>            <td>資料1</td>
>            <td>資料1</td>
>        </tr>
>        <tr>
>            <td>資料1</td>
>            <td>資料1</td>
>            <td>資料1</td>
>            <td>資料1</td>
>        </tr>
>        <tr>
>            <td>資料1</td>
>            <td>資料1</td>
>            <td>資料1</td>
>            <td>資料1</td>
>        </tr>
>    </table>
```

### 1-1、表格切分成三大區

```md
> 表頭 -> thead -> tr -> th(列標題) + td
> 表身 -> tbody -> tr -> th(列標題) + td
> 表尾 -> tfoot -> tr -> th(列標題) + td
> 
> Ps. 表格的title是caption，寫在table裡面
```


### 1-2、表格實際操作

表格實際操作稍微難的地方在於rowspan、colspan的控制，下面有範例可以參考
```markdown
> <table border="3" width="300px">
>     <caption>五倍小學成績單</caption> -> 表格的title
>     <thead>
>         <tr>
>             <th>姓名</th>
>             <th>國文</th>
>             <th>英文</th>
>             <th>數學</th>
>        </tr>
>     </thead>
>     <tbody>
>          <tr>
>            <td>小明</td>
>            <td>100</td>
>            <td>50</td>
>            <td>40</td>
>         </tr>
>         <tr>
>             <td>小華</td>
>             <td>33</td>
>             <td>11</td>
>             <td>90</td>
>         </tr>
>         <tr>
>            <td>大雄</td>
>            <td>50</td>
>            <td>90</td>
>            <td>22</td>
>        </tr>
>    </tbody>
>    <tfoot>
>        <tr>
>            <th rowspan="2">平均</th>  ->"直向"合併儲存格語法 > rowspan = row + span
>                                                             意思是跨幾個欄位
>            <td>0</td>
>            <td>0</td>
>           <td>0</td>
>        </tr>
>        <tr>
>            <th>備註</th>
>            <td colspan="2"></td>     -> "橫向"合併儲存格語法 > colspan = column + span
>                                                             意思是跨幾個欄位
>        </tr>
>    </tfoot>
> </table>
```


2、表單的input類型
------

### 2-1、checkbox
```markdown
> <div>
>     <label for="">
>         <label>
>             <input type="checkbox" id="123"> 釣魚
>         </label>
>         <label>
>             <input type="checkbox" id="123"> 吃飯
>         </label>
>         <label>
>             <input type="checkbox" id="123"> 睡覺
>         </label>
>     </label>
> </div>
```


### 2-2、radio
```markdown
> <div>
>     <label>
>         <label>
>             <input type="radio" id="" name="1"> 男   -> 這個 name 屬性設定相同可以變成單選
>         </label>
>         <label>
>             <input type="radio" id="" name="1"> 女
>         </label>
>         <label>
>             <input type="radio" id="" name="1"> 雙性
>         </label>
>     </label>
> </div>
```

### 2-3、select下單選單
```markdown
> <div>
>     <label for=""> 地址
>         <select name="" id="">
>             <option value="">台北</option>
>             <option value="">台中</option>
>             <option value="">台南</option>
>         </select>
>     </label>
> </div>
```

### 2-4、input 有些特別的屬性  
:::info
min 輸入最小值    
max 輸入最大值  
maxlength 輸入長度最大值  
minlength 數入長度最小值  
placeholder 輸入前顯示在格子上的提示字      
:::


3、切版標籤設定建議
------

如果今天你在HTML標籤沒有頭緒的時候，可以用無障礙網站為目標，因為那些網站就是把標籤設定的很完善，並且讓程式去讀取，最後給需要的人"看"



4、section 跟 article 的差異
------
section、article是大家最常搞混的兩個屬性，有時候都不確定哪個區塊要用哪個屬性，因此下面會稍微介紹一下兩個差異

:::tip    
首頁、清單頁(圖片)、內容頁，使用主文的時候通常用article  
其他小區塊用section  
  
側邊欄可以用section 、 div 都可以  
老師在切版的時候  設定這些區塊都會依照想要呈現給seo的權重來設定  
因此這些區塊的標籤沒有一定   
  
在切版前，要先想好要 section 包 article 還是 article 包 section   
誰包誰都取決你想要怎麼解釋你的網頁  
 
最簡單切版的方式，就是以無障礙(盲人)網站為主，你要做出一個標籤非常精準的網站      
:::






5、css 介紹
------

### 5-1、HTML檔案連結CSS檔案

用 link 把css放到另外一個資料夾，這樣就可以不用都把CSS寫在同一個資料夾裡面了
```css
> <link rel="stylesheet" href="first.css">  
```


### 5-2、css包含三個重點 - 選取器、屬性、優先權  
  

### 5-2-1、選取器、屬性
```markdown
> 你要設定的對象 {
>   屬性: 值;
>   屬性: 值;
>   屬性: 值;
> }
> 
> 選取器 {
>   屬性: 值;
>   屬性: 值;
>   屬性: 值;
> }
```


### 5-2-2、優先權
3種的優先度權限，優先權比較低的CSS會被蓋掉
```md
> (1)、寫在html裡面的style(稱為行內樣式-inline style)，優先權比較高  
> Ex. <h1 style="color: red";>123</h1>  

> 這個優先權會蓋掉寫在css style裡面的資料  
>  
> (2)、後寫的會蓋掉前面寫的  
> (3)、ID > class > Tag(p標籤)   
```

