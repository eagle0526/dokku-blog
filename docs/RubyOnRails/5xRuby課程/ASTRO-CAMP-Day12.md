---
title: ROR/JavaScript(04)
sidebar_label: "DAY12. [ASTROCamp] JavaScript-4"
description: JavaScript
last_update:
  date: 2022-10-20
keywords:
  - 5xRuby
  - JavaScript
sidebar_position: 13
---



1、ES6 / ES2015語法補充
------

### 1-1、字串與變數的組合
```md
> `${} 123 ${}`
```
### 1-2、箭頭函數
```md
> const addNumber = () => {
>     return 
> }
```


2、物件簡寫
------

先設定兩個變數  
```markdown
> const age = 18;
> const attack = () => console.log(123)
```

在設定物件的時候，如果key跟值一樣，key值可以直接寫該變數就好   
```md
> const hero = {
>     name: "悟空",
>
>     age: age,          如果key跟value一樣，可以寫一個就好
>     age,               <= 像這樣
>
>     attack: attack     函式也可以
>     attack,            <= 像這樣 
> }
```

3、物件解構
------

### 3-1、首先給一個物件
```markdown
> const hero = {
>     name: "悟空",
>     age,                
>     attack,             
>     location: "地球",
> }
```


### 3-2、開始解構
```markdown
> const { name, age, location, hello } = hero  -> 這一行取代下面兩行
> 
> const name = hero.name
> const age = hero.age
```

:::tip
**如果在解構裡面解構一個物件中沒有的key，會印出undefined**  
上面的hello就會印出undefined  
:::


### 3-3、解構後再拿出來用
創新的物件，可以幫原本的key值換名字

```markdown
> const newHero = {
>     name,
>     age,
>     place: location,
> }
```

### 3-4、在解構時順便給新名字

這很常用於，引用別人的json檔案後，把別人給的key值重新解構後，再給予一個新的名稱   

原本是location，解構並賦予新名字place    
```markdown
> const { name, age, **location: place** } = hero
```

上面location會多做一個設定，是因為要為下面的縮寫    
```md 
> const newHero = {
>     name,
>     age,
>     place,            => 這邊簡寫成 place
> }
```


### 3-5、解構實際範例
  
#### 3-5-1、從別人給的檔案拿資料來用(我們引入了一份此種格式的檔案)
```markdown
> bike = {
>     sno: [1,2,3,4,5,6,7],
>     txt: [sdfs,sfsdf,eewr,2342,sff,tyjt].
> }
```
  
#### 3-5-2、解構並換成新名字
sno、txt是別人的key，並順便改成number、age

```markdown
> const { sno: number, txt: age } = bike
```

#### 3-5-3、設定新物件變數
```markdown
> const newName = {
>     number,
>     age,
> }
```


### 3-6、用函數也可以用解構

有一個物件，裡面有兩個key
```md
> const user = {
>     name: "悟空",
>     age: 18,
> }
```

有一個函數，並且把剛剛的物件丟進這個函數
```markdown
> function printUser({name, age}) {     -> 解構物件裡面值
>     console.log(name),                -> 印出悟空
>     console.log(age),                 -> 18
> }

> 把物件帶進上面的函數
> printUser(user)
```




4、點點點...
------

兩個陣列結合(可以使用concat結合在一起)

### 4-1、JS array flat(輾平)    
這個方法可以把二維陣列轉成一維陣列(降維)    

```markdown
> const comicHero = ["悟空", "魯膚"]
> const comicHero2 = ["22", "33"]
> 
> const allHeroes = [comicHero, comicHero2].flat()
```

### 4-2、...跟flat一樣效果
用...也可以，這個結果跟上面的flat()結果一樣

```markdown
> const allHeroes = [...comicHero, ...comicHero2]
> ...可以把一個陣列炸開，不過記得要把該陣列包起來
```

### 4-3、...應用在函式裡面
```markdown
>                       下面的d可以自己取名(這個動作就是把剩下的函數包起來)
> function hi(a, b, c, ...d) {
>     console.log(a, b, c)
>     console.log(d)       -> 這個會印出剩下所有的數，包成一個陣列
> }
> 
> hi(1,2,3,4,5,6,7,7)
```



### 4-4、...實戰 - 實作一個concat方法

```markdown
> const a = [1, 2, 3];
> const b = [4, 5, 6];
> const c = [7, 8, 9];
> const d = ["a", "b", "c"];
> 
> function myConcat(...other) {
>   // ....
>     return other                      # 2、直接用...把所有陣列炸開
> }
> 
> const all = myConcat(a, b, c, d);     # 1、把所有陣列丟回函式裡
> 
> console.log(all);
```




### 4-5、陣列解構
  
(1) 給一個陣列  
```markdown
> const list = [1, 2, 3]
```
  
(2) 解構此陣列  
```markdown
> const [ first, second, last ] = list
```
  
(3) 如果今天不想要拿中間第二個，可以用底線來代替    
```markdown
> const [ first, _, last ] = list     > 這個會印出1, 3
```
  
(4) ...也可以用在陣列解構
```markdown
> const [first, ...rest] = list

> console.log(first)   # [ 1 ]
> console.log(rest)    # [2, 3]
```
  
### 4-6、展開和解構很像，但是是不一樣的東西，要注意
```markdown
> const [first, ...rest] = list  => 這是解構
> const a = [first, ...rest]  => 這是展開 
```


5、更多的DOM操作
------

### 5-1、新增dom - createElement、appendChild

```markdown
> const hi = document.querySelector("#id");
> const newBtn = document.querySelector("#newBtn")
> const body = document.querySelector("body")
> 
> newBtn.addEventListener("click", () => {
>     // 先創一造一個元素 這算不會馬上儲存
>     const el = document.createElement("div")  
> 
>     // 給div中間塞文字
>     el.textContent = "123";
> 
>     // 在指定地點產生出上面兩個動作
>     hi.appendChild(el);
> })
```



### 5-2、刪除DOM - remove()

```HTML
<ul>
    <li>悟空</li>
    <li>克林</li>
    <li>達爾</li>
</ul>

<button id="removeBtn">抓最後一個</button>
<button id="addBtn">新增一個按鈕</button>
```

```markdown
> const removeBtn = document.querySelector("#removeBtn")
> const addBtn = document.querySelector("#addBtn")
> const ulBlock = document.querySelector("ul")
> 
> addBtn.addEventListener("click", () => {
>     // 設定el變數 -> 這個變數可以創造一個 li標籤 
>     const el = document.createElement("li")
> 
>     // 在這個標籤裡面塞字
>     el.textContent = "456"
>
>     // 在指定的地方，用appendChild增加剛剛的 li 標籤
>     ulBlock.appendChild(el)
> 
> })
> 
> removeBtn.addEventListener("click", () => {
>     //先抓最後一個li元素
>     const lastElement = document.querySelector("ul li:last-child")
>
>     //用if增加邊界值，防止當所有元素被刪掉的時候，就會壞掉
>     if (lastElement) {
>         // 抓到此元素刪掉          
>         lastElement.remove();
>     }
> 
> })
```

:::tip
也可以先抓ul元素，並在ul這邊新增、刪除資料，再用removeChild   
ul.removeChild    
:::  


### 5-3、removeChild實作

```markdown
> removeBtn.addEventListener("click", () => {
>     // ul - 父層
>     const nodeElement = document.querySelector("ul")
> 
>     // 最後一個li - 子層
>     const lastElement = document.querySelector("ul li:last-child")
>     
>     //增加邊界值，防止當所有元素被刪掉的時候，就會壞掉
>     if (lastElement) {
>         //父層.removeChild(子層)
>         nodeElement.removeChild(lastElement);
>     }
> })
```


### 5-4、取得上層DOM - parentElement、parentNode


#### 5-4-1、node 跟 element 的差異
(1) Node 註解、空格都會被抓到   
(2) element 也是一種 Node   
(3) Node 有的功能 Element都有   

該用哪一種 -> parentElement推薦，這樣不會拿到不該拿的物件

:::tip
如果今天專案要做文字編輯器，就需要 node，因為會需要抓到空格那些東西   
:::

  
### 5-5、取得子層DOM - children、childNodes

```markdown
> children   - 取得子層element
> childNodes - 取得子層NODE
```


### 5-6、取得兄弟姊妹層Dom

先給一個li物件
```markdown
> const item = document.querySelector("li:nth-child(2)")
```

```markdown
> #### element兄弟姐妹系列
> item.previousElementSibling  
> item.nextElementSibling

> #### node兄弟姐妹系列
> item.previousSibling  
> item.nextSibling
```


### 5-7、在指定位置安插DOM

(1) insertAdjacentElement
(2) insertAdjacentHTML


#### 5-7-1、insertAdjacentElement置說明
```markdown
>    ----------> before begin
> ul
>    ----------> after begin
>     li
>     li
>     li
>    ----------> before end
> ul
>    ----------> after end
> 
> 
> 
> 分別在四個位置插入物件
> ul.insertAdjacentElement("beforebegin", el)
> ul.insertAdjacentElement("afterbegin", el)
> ul.insertAdjacentElement("beforeend", el)
> ul.insertAdjacentElement("afterend", el)
```

#### 5-7-2、點擊後，把內容塞進指定位置
```markdown
> // 抓ul物件
> const newUl = document.querySelector("ul")
> 
> // 點擊物件
> const btn = document.querySelector("btn")
> 
> //點擊按鈕後的動作
> btn.addEventListener("click", () => {
>     const el = document.querySelector("li")
> 
>     el.textContent = "123"
> 
>     newUl.insertAdjacentElement("afterbegin", el)
> })
```


#### 5-7-3、另一種寫法insertAdjacentHTML
```markdown
> btn.addEventListener("click", () => { 
>     const el = "<li>123</li>"
>     newul.insertAdjacentHTML("afterbegin", el)
> })
```


6、實戰 - 製作代辦事項處理
------

檔案實作在 JS 訓練裡面的 TODOApp 裡面













