---
title: ROR/DOM 練習
sidebar_label: "2. [ROR] DOM 練習"
description: DOM 練習
last_update:
  date: 2023-11-07
keywords:
  - ROR
  - DOM
sidebar_position: 2
---


insertAdjacentHTML() & removeChild()
------

本次練習的有兩件事情  
1. 點擊按鈕1，增加ul中的li數量  
2. 點擊按鈕2，減少ul中的li數量  
 


:::tip
**使用querySelector抓取物件的小提醒~~~~**  
nth-child 使用方式
ex. 有兩個button元素，被一個div.btn包住  
:::

```md
> HTML

> <ul>
>     <li><a href="">1</a></li>
>     <li><a href="">2</a></li>
>     <li><a href="">3</a></li>
>     <li><a href="">4</a></li>
>     <li><a href="">5</a></li>
> </ul>
> <div class="btn">
>     <button>點我增加li</button>
>     <button>點我減少li</button>
>     <button>點我更改ul顏色</button>
>     <button>點下去ul 不見</button>
>     <button>點下去ul 出現</button>
> </div>
```

任務一 - 增加ul中li數量
```md
> 首先抓到按鈕，並對按鈕做監聽，接著宣告一個變數為li最後一個元素，在把該元素用insertAdjacentHTML，塞到ul裡面

> const addLlBtn = document.querySelector(".btn button:nth-child(1)") 
> 
> addLlBtn.addEventListener("click", (e) => {
>     
>     const lastElement = document.querySelector("ul li:last-child")
>     const el = `<li><a href="">5</a></li>`
> 
>     lastElement.insertAdjacentHTML("afterend", el)
> })
> 
```


任務二 - 減少ul中li數量
```md
> 首先抓到按鈕，並對按鈕做監聽
> 這邊使用removeChild方法，因此要宣告兩個變數
> 1、父層元素 = ul     2、子層元素 = li:last-child
> Ps. removeChild方法一定要給一個引數喔~~引數就是子層元素 = 想要刪掉的元素

> minusLlBtn.addEventListener("click", (e) => {
>     // console.log(e.target);
>     const nodeElement = document.querySelector("ul")
>     const lastElement = document.querySelector("ul li:last-child")
> 
>     // liList.insertAdjacentHTML("afterend", el)
>     
>     nodeElement.removeChild(lastElement)
>     
> })
```


:::tip
**關鍵提醒**  
如果今天是用 last-child 抓取要新增、刪除的物件，要避免在監聽器會宣告  
如果在監聽器外宣告，會導致抓取的事件被刪掉後，就導致整個壞掉  
ex. const lastElement = document.querySelector("ul li:last-child")  
上面這這一行要在監聽器裡面才宣告   
:::


classList Vs. className
------


3. 點擊按鈕3，用classList，改變ul的顏色  
4. 點擊按鈕4，用classList，隱藏ul  
5. 點擊按鈕5，用className，讓ul出現(還原到ul初始css) 

:::tip
**classList的特色：**
會把class全部加在class尾部，而因為優先權的關係  
後面加的會覆蓋掉前面的，雖然class可以瘋狂疊加(設定多個class，而每個class只有1. 2個屬性)  
但是要小心覆蓋的問題  

**className的特色：**
可以直接把某個物件的class只設定該名稱，這樣就可以強致某物件的class只能是一種  
:::




```md
> CSS

> ul {
>     margin: 200px auto;
>     width: 500px;
>     display: flex;
>     outline: 1px solid tomato;
>     
> }
> 
> .tomato {
>     background-color: tomato;
>     color: blueviolet;
> }
> 
> .dis-none {
>     display: none;
> }
> .dis-block {
>     display: flex;
> }
```


任務三 - 用classList，改變ul的顏色 
```md
> JS

> 幫ul增加新的class - 改變ul顏色
> addUlClassBtn.addEventListener("click", (e) => {
>     ulList.classList.add("tomato")
> })
```

任務四 - 用classList，隱藏ul 
```md
> JS

> 幫ul增加新的class - 隱藏ul
> ulNoneBtn.addEventListener("click", (e) => {
>     ulList.classList.add("dis-none")
> })
```

任務五 - 用className，讓ul出現(還原到ul初始css) 

```md
> JS

> 還原成本來的ul class - 出現ul
> ulBlockBtn.addEventListener("click", (e) => {
>     ulList.className = "ul"
> })
```






parentElement vs. parentNode - 父元素、父節點
------


### parentElement -> 取得父層元素 - element元素

印出ul的父層的element - body
```md
> console.log(ulList.parentElement)  -> 這個會抓到UL的上層，也就是body
  
> 印出結果 -> <body></body>                      -> 還有因為是用element的關係，不會抓到node    
```

印出li的父層的element - il
```md
> console.log(liList.parentElement)  -> 這個會抓到li的上層，也就是ul  

> 印出結果 -> <ul></ul>                -> 還有因為是用element的關係，不會抓到node  
```




### parentNode -> 取得父層元素 - node元素

印出Ul父層的node - BODY
```md
> console.log(ulList.parentNode)

> 印出結果 -> BODY
```


印出Li父層的node
```md
> console.log(liList.parentNode)

> 印出結果 -> UL
```



children vs. childNodes - 子元素、子節點
------


### children -> 取得子層元素 - element元素

印出Ul子層的element
```md
> console.log(ulList.children)

> 印出結果 -> HTMLCollection(5) [li, li, li, li, li]
```

印出li子層的element
```md
> console.log(liList.children)

> 印出結果 -> HTMLCollection [a]
```




### childNodes -> 取得子層元素 - node元素

印出Ul子層的node
```md
> console.log(ulList.children)

> 印出結果 -> NodeList(11) [text, li, text, li, text, li, text, li, text, li, text]0: text1: li2: text3: li4: text5: li6: text7: li8: text9: li10: textlength: 11[[Prototype]]: NodeList
```

印出li子層的node
```md
> console.log(liList.children)

> 印出結果 -> NodeList [a]0: alength: 1[[Prototype]]: NodeList
```




Sibling - 兄弟姊妹元素、節點
------

```md
> ### element系列
> item.previousElementSibling  
> item.nextElementSibling


> ### node系列
> item.previousSibling  
> item.nextSibling
```


### previous為兄弟姐妹的前一個元素

UL前面沒有元素了，所以回傳null
```md
> console.log(ulList.previousElementSibling)

> 印出結果 -> null
```

LI前面沒有元素了，所以回傳null
```md
> console.log(liList.previousElementSibling)

> 印出結果 -> null
```




### nextElement為兄弟姐妹的後一個元素

UL同一層元素的後一個就是div btn
```md
> console.log(ulList.nextElementSibling)

> 印出結果 -> <div class="btn"></div>
```


li同一層元素的後一個就是第2個li
```md
> console.log(liList.nextElementSibling)

> 印出結果 -> <li></li>
```



### Sibling兄弟姐妹的節點語法

用節點來看兄弟姊妹元素，會發現所有下面四個印出來都是text  
仔細點進去看的話，會發現抓到節點的值，都是空格，因此使用node的時候，要注意

```md
> console.log(ulList.previousSibling)
> console.log(liList.previousSibling)
> console.log(ulList.nextSibling)
> console.log(liList.nextSibling)

> 印出結果 -> #text
```

:::tip
**node 和 element 的差異**  
1. Node 註解、空格都會被抓到    
2. element 也是一種 Node  
3. Node 有的功能 Element都有  
該用哪一種 -> element 推薦，這樣不會拿到不該拿的物件  
:::




:::tip
**額外補充 - nodetype是什麼**  
  
console.log(ulList.parentNode.nodeType)  
console.log(liList.parentNode.nodeType)  
    
上面兩者都印出 **1** -> 1代表的意思是"Node.ELEMENT_NODE"，一個元素的節點，例如 `<p>` 和 `<div>`。  
可以到以下連結看其他數字的意思  
[nodeType中的數字，代表的意義][nodetype]  
:::




Active Button
------

進階應用 - 製作Active Button(點擊不同按鈕，點下的那個按鈕active被持續啟動，前一個按鈕會回按鈕前狀態)  
  
  
簡易解釋 - 首先給三個按鈕，並給基本css設定，按鈕有兩個特別屬性要提到，一個是button普通顏色狀態(有一個按鈕先給他active的屬性)，我們目標是，點擊另外一顆按鈕時，原本active的按鈕的屬性會被移除，並且新增加的按鈕，會被增加className，會是有新的啟動狀態的按鈕顏色。  
  
[codepen實際畫面](https://codepen.io/Yeegle/pen/NWzreXO)  
  
```md
> HTML

> <div class="part2">
>     <h2>Dom btn-hightlight效果</h2>
>     <div class="btns2">
>         <button class="dom">按鈕3</button>
>         <button class="dom active">按鈕4</button>
>         <button class="dom">按鈕5</button>
>     </div>
> </div>
```

```md
> css

> .part2 {
>     margin: 100px auto;
>     width: 800px;
>     outline: 1px solid black;
> }
> 
> h2 {
>     text-align: center;
> }
> 
> 
> button {
>     margin: 20px;
>     padding: 15px 30px;
>     background-color: antiquewhite;
> }
> 
> .btns2 {
>     
>     display: flex;
>     outline: 1px solid tomato;
>     justify-content: center;
>     
> }
> 
> .active, .active:hover ,.dom:hover {
>     background-color: #666;
>     color: white;
> }
```

```md
> JS

> const btns = header.querySelectorAll(".dom")
> for(let i = 0; i < btns.length; i++) {
>     btns[i].addEventListener("click", (e)=>{
>         let current = document.querySelector(".active")
>         
>         current.className = current.className.replace("active", "")
>         btns[i].className = "active"        
>     })
> }
```


[active範例網頁][activeButton]



[nodetype]: https://developer.mozilla.org/zh-CN/docs/Web/API/Node/nodeType
[activeButton]: https://www.w3schools.com/howto/tryit.asp?filename=tryhow_js_active_element