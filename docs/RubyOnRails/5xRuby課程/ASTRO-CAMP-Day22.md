---
title: ROR/React
sidebar_label: "DAY22. [ASTROCamp] React"
description: Vue
last_update:
  date: 2022-11-05
keywords:
  - 5xRuby
  - React
sidebar_position: 23
---


React
------
先安裝三個延伸外掛  

1. Todo Tree  -> 等一下要講的重點  
2. ESLint  -> 免費有人幫你code review的工具，可以自動幫你查看code哪邊錯誤(ex. 很多業界是如果ESlint有錯誤，是不能進版控的)  
3. Code Spell Checker (optional, 英文 typo 好幫手)  



### React優勢、優勢  
1. 職缺比較多，不過兩個會其一，就可以  
2. React是由facebook主導的  
3. react專用標籤語法 -> JSX -> 瀏覽器認不得，經由JS編譯後，就可以被認得，JSX長得很像HTML，但他不是   
4. 一個render只能有一個根節點，所以今天要塞一個h1、一個h2，要用一個div包住才行   
5. 開發快速

:::tip
直譯語言 - ruby、JS - 不需要加工  
編譯語言 - react - 需要經過加工，電腦才懂你在寫什麼  
:::



為什麼要用react
------
react中所有東西都是component化  
優點是，開發快速 -> 可以多人一起協作  
某個人可以寫一個component  
某個人可以寫一個component  
某個人可以寫一個component  
  
最後再由一個人把這些component組建起來  



開始實作前一些介紹
------
  
1. npm i -> 可以把module所有檔案從遠端拉過來，可以使用npm指令，是因為有node js  
  
2. 檔案副檔名，叫做jsx or js，都沒有差別，是個人coding習慣，副檔名是給軟體看的，對電腦沒差  
  
3. React就只是JavaScript的fc，不過他return的是JSX，component的變數名稱一定要大寫字母，檔案名也建議要大寫開頭  
  
4. component本身是fc(其實就是setState)，把App加上去後，萬物皆為component
  
5. 潛規則，檔案要多，檔案不能太長!!!!!!!!  
  
6. props本身的型別是物件  
  
7. 物件導向的核心是"繼承"，如果今天超級瑪莉吃到大蘑菇，不用在新寫一個超級瑪莉  
  
8. 大寫字母開頭的都會變成component  
  
9. children是jsx內建的關鍵字，只能用來傳參數，不能用變數來定義  


:::info
JS奇耙點  
NAN === NAN -> false  (非數字型別不等於非數字型別)  
:::

:::tip
React  
component接收參數的唯一方法，就是從props取得  
:::



children
------

1. 只要是你自己寫的React component裡面包的元素，一律會被放到props底下的children，一定會叫children  
2. children使用時間，一開始可以直接刻一個公版，中間留一塊放children  
3. 這個children中間可以隨後面的人想要塞任何內容物  
  
```md
> <FunctionalCard01                           -> 我是component的頭(自己寫的component)
>     price={200}
>     img="http://fakeimg.pl/500x300/e74c3c/"
>     name="奶綠茶"
>     
>     <h1 className="bg-info">我是子元素</h1>   -> 我是children，我被component包住了
> </FunctionalCard01>                         -> 我是component的尾

> <h1 className="bg-info">我是子元素</h1>  -> 這個就是children
```

前後端分離的意思
------

### 負責工作  
前端負責刻所有的頁面  
後端負責出API -> 網頁的network -> json檔案  

### 責任分開  
頁面跑版->前端責任  
資料出錯->後端責任  
  
1. 前後端分離的優點->責任好釐清  
2. 傳統都是直接把dom全部render到頁面上  
3. 使用react或vue，通常都會做到前後端分離  


React的私有變數
------

今天要怎麼在react的component裡，做一個私有變數，不過私有變數是啥？  
ex. 現在有兩隻怪物，現在有一隻死了，不會影響到另外一隻還活著，這就叫做物件導向的私有變數  



### React.useState
此為react專門的寫法，為了解決React不是class架構基底的問題(class基底有繼承概念，並且有this可以用)  
  
假設我今天要讓一個component有一個自己的私有變數，要使用useState，並把初始值塞進去  
```md
> const [count, setCount] =  React.useState(0)

> count是變數
> setCount是函式
```

上面那一段的意思是，我要用useState宣告count變數，並且初始值為0
後面setCount是什麼意思呢？那個是用來改變count變數狀態的  

Ps. 在react世界，要更改變數狀態，只能用setCount這個函式，要用這個，才能讓react知道變數更新  

如果用JS來宣告，會長什麼樣呢？  
```md
> const [count, setCount] = React.useState(0);    # React宣告
> 
> /* 上面一行等於三面這三行                           # JS宣告
> const stateArr = React.useState(0);
> const count = stateArr[0];
> const setCount = stateArr[1];
> */
```

如果上面還看不懂，也可以改寫成這樣  
```md
> const [aaa, bbb] =  React.useSate(0)

> aaa = 0
> bbb = fc (改變aaa狀態的fc)
```
不過這樣寫會被罵，所以記得要照著規則走  
  
aaa, setBBB  -> 後面那個函式很重要，如果今天要改變aaa的狀態，不能這樣寫 aaa++  
因為這樣react會不知道該物件改變了，要用後面的fc，React才會知道  
  
Ps. 不能用let宣告，要用React.useSate(0)宣告一個私有變數  
  
  
:::tip
**面試題**  
請試著跟非程式人員解釋啥是API  
:::


會員登入
------
Q. 如何用react切換兩種會員狀態的看到不同的頁面？  
A. 先宣告兩種不同的變數。登入狀態和顧客狀態  
  
```md
> const UserGreeting = () => <h1 className="user">登入成功</h1>;  
> const GuestGreeting = () => <h1 className="guest">Please sign up.</h1>;  
```
  
Ps. 實務上，不同的component放在不同的檔案，正常來說麵兩個要拆開  
Ps. 狀態切換，老師公司規定用data-active，這樣就可以讓大家更改某個東西的狀態可以統一寫在這邊  





React css
------
react的css也改成駝峰式命名，下面的backgroundColor就是
```md
> style={{
>     width: 200,
>     height: 200,
>     backgroundColor: isGreen ? 'green' : 'red',
> }}
```


React for迴圈
------
使用for迴圈產生的虛擬dom元素，要用下面那個key接住，先死記
```md
> return <li key={text}>{text}</li>;
```



setList陣列記憶體
------

### push
用push方法，這個陣列是傳記憶體位置
```md
> let a = [1,2,3]
> let b = a
> b.push(4)

> consol.log(a)   => [1,2,3,4]

> Ps. =(等於) 有傳數值、傳記憶體位置兩個意思
```
  
### 繼承字串
這個字串是傳數值  
```md
let c = "123"
let d = c
```
  
會提到這個，就是因爲，push + for迴圈 + setList會出問題  
```md
> list.push(new Date().toString());
> setList(list);            -> 用set函式，他會在同一個記憶體位置，所以react不會理你(以為沒改變)
```

### setList正確用法
用concat會幫你開一個新的記憶體空間，react就會知道你更改了東西
```md
const newList = list.concat(new Date().toString());
setList(newList);
```


useEffect
-----
每一個react都有自己的生命週期  
```md
> React.useEffect(() => {
>     console.log('%cmounted', 'background:#2ecc71');
>     return () => {
>         console.log('%cunmount', 'background:#e67e22');
>     };
> }, []);       # useEffect最後塞空陣列
```
當useEffect最後面塞一個空陣列，就可以讓useEffect變成有生命週期  
生命週期可以幹嘛，可以去跟後端拿API  
  
  
useEffect可以模擬某個component的變化  
```md
> React.useEffect(() => {
>     console.log('effect count', count);
>     if (count === 5) {
>         alert('hi, React');
>     }
> }, [count]);  # useEffect最後面塞一個值
```
當useEffect最後面塞一個count，如果今天原本的空陣列有塞東西，等待count變化的時候，就會再執行一次


:::tip
useEffect只能關注state、props的值，其他都不能關注  
:::




前後端分離打API
------

一開先給一個空陣列
```md
> const [data, setData] = useState([]);
```

接著用useEffect查看狀態，把後端的json接上來，之後用迴圈渲染資料到想要的地方
```md
> useEffect(() => {
> fetch('https://jsonplaceholder.typicode.com/todos')
>     .then((res) => res.json())
>     .then((res) => {
>     setData(res);
>     });
> }, []);
```
  
  
兩個component的溝通
------
  
1. component跟component溝通只有props，不過可以透過fc來呼叫  
2. 當你上層的餵值給這個component，這個component就會自動更新畫面  
3. 老爸傳值給兒子是透過props，那反過來，兒子身上有一個按鈕，按下去後要怎麼呼叫老爸的函式？可以透過函式  
Ps. component跟component溝通，只有透過props，但是我們可以傳fc進來，就是說，雖然component跟component只能透過props溝通，但是可以傳fc近來給兒子呼叫。  
  
**潛規則**  
如果今天props是fc的話，記得要on開頭，像下面這樣  
```md
> const {onCallParent, count} = props
```
上面這一句的意思是，props可以接收一個變數count，再接收props傳進來的onCallParent


