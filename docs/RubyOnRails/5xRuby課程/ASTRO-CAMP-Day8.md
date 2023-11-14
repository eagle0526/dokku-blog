---
title: ROR/JavaScript(02)
sidebar_label: "DAY8. [ROR] JavaScript-2"
description: JavaScript
last_update:
  date: 2022-10-13
keywords:
  - 5xRuby
  - JavaScript
sidebar_position: 9
---




1、函數 function
------

### 1-1、什麼是函數？
f(x) = 3x + 2

f -> function的縮寫
x -> 參數 parameter

### 1-2、為什麼要寫函式
(1) 為一函式定義一個行為   
(2) 重複利用   


### 1-3、function hello world
```markdown
> // name -> 參數parameter

> function sayMyName(name) {
>     console.log("hi222222");
>     console.log(name);
> }


> // "Yeeeeee" -> 引數argument
> // () -> 呼叫、使用、啟動

> sayMyName("Yeeeeee")
```



### 1-4、有參數的函式，呼叫時不給引數的話會印出什麼

如果函數有一個參數，但是如果引用的時候啥都沒給，最後會給你undefined

```markdown
> function sayHi(name) {
>     console.log(name)
> }
> 
> sayHi()                              -> undefined
```


### 1-5、參數給一個，引數給兩個，最後會給你第一個
```markdown
> function sayHi(name) {
>     console.log(name)
> }
> 
> sayHi(1, 2)                         -> 1
```

### 1-6、參數設定預設值
```markdown
> function sayHi(name, age = 18) {    -> 預設為18
>     console.log(name)
>     console.log(age)
> }
> 
> sayHi("kk")                         ->  印出"kk"、18
```


### 1-7、匿名函數anonymous

常數設定 + 函數設定融合 (後面function整段叫做匿名函數)

```markdown
> const sayMyName = function ccc(name) {    -> 裡面ccc可寫可不寫，因為ccc出function範圍就沒有意義
>     console.log(name);
> };
> 
> 隱藏寫法
> const sayMyName = function (name) {       -> 裡面ccc可寫可不寫，因為ccc出function範圍就沒有意義
>     console.log(name);
> };
```

### 1-8、函數呼叫寫在前面 - 是可以執行的

```markdown
> sayHi("kk")                         ->  印出"kk"、18
> 
> function sayHi(name, age = 18) {    -> 預設為18
>     console.log(name)
>     console.log(age)
> }
```


### 1-9、變數宣告函數寫在前面 - 無法執行(會噴出ReferenceError)

```markdown
> say("kk")                         ->  噴錯 ReferenceError
> 
> let say = function sayHi(name, age = 18) {
>         console.log(name)
>         console.log(age)
> }
```

:::tip
undefined() 呼叫，會造成typeError     
:::




2、變數提升
------

### 2-1、JS分兩階段運作
 - 第一輪是掃瞄 - 建立期    
  -- 1A - 註冊名稱(identifier)      
  -- 1B - 進行初始化(檢查語法、建立變數)    
    
 - 第二輪式執行 - 執行期    
  -- 執行函數、賦值     


```markdown
> 情境算式
> var a = 1
> console.log(a)
>
> ----------
> 建立期         執行階段  
> var a         1A、1b
> 
> 執行期
> a = 1           2
> console.log(a)  2       
```

:::info
**兩階段運作**  
建立期間把所有程式碼掃過，並把名稱都設定好  
執行期間才把值給出去  
:::

### 2-2、把宣告寫在log下方

```markdown
> 情境算式
> console.log(a)
> var a = 1
> console.log(a)

> 第一階段
> console.log(a)     # 不執行
> var a = 1          # 執行 1A、1B -> 已初始化，但是還沒賦值
> console.log(a)     # 不執行

> 第二階段
> console.log(a)     # 執行，印出undefined
> var a = 1          # 執行，賦值
> console.log        # 執行，印出1
```


### 2-3、只宣告，不賦值
```markdown
> 情境算式
> var a = 1
> var a
> console.log(a)         # 印出1

> 第一階段
> var a = 1              # 執行 1a、1b
> var a                  # 執行 1a、1b
> console.log(a)         # 不執行

> 第二階段
> var a = 1              # 執行 2，給值變1
> var a                  # 執行 2，但是啥都沒給，所以忽略
> 
> console.log(a)         # 執行，印出a是1
``` 

### 2-4、在判斷式裡面用var宣告
```markdown
> 情境算式
> if (false) {
>     var a = 1
> }
>
> console.log(a)                    # 印出 undefined

> 第一階段
> if (false) {                      # 沒執行
>     var a = 1                     # a = undefined
> }
> 
> console.log(a)                    # 沒執行
 
> 第二階段
> if (false) {                      # 執行
>     var a = 1                     # 因為if裡面是false 所以不執行
> }
> 
> console.log(a)    -> undefined    # 執行，最後印出undefined
```
  
Ps. if(false)雖然第一階段沒執行，但是裡面的var a 第一階段有執行，不過還沒賦值，等到第二階段，因為if直接錯誤，所以就不會進到if裡面，因此var a就是undefined狀態
  
  
### 2-5、let 解決變數提升

**let 怎麼解決變數提升 => 1b不執行**

```markdown
> 第一階段
> console.log(a)     # 不執行
> let a = 1          # 不執行1b, 被一個罩子蓋住了(TDZ)
> 
> 第二階段
> console.log(a)     # 執行, 但是因為，1b沒執行，噴出referenceError(尚未初始化)
```

### 2-6、TDZ暫時死區(Temporal Dead Zone)

給值的話，暫時死區(蓋子)就會被拿開

```markdown
> **正常情況 let 的執行狀況**
> 
> 第一階段
> let a = 1          # 執行，不執行1B，並給一個蓋子
> console.log(a)     # 不執行

> 第二階段
> let a = 1          # 執行，把蓋子拿掉
> console.log(a)     # 執行，印出 a = 1
```


### 2-7、函數宣告順序


#### 2-7-1、範例第一題 - 用const宣告一個fc，並在此之前呼叫該變數
```markdown
> 第一階段
> sayHi()                                    # 第一輪不執行
> const sayNyName = function sayHi() {       # 第一輪做宣告，但是1b沒做 
>     console.log()
> }

> 第二階段
> sayHi()                                    # 第二輪呼叫變數
> const sayNyName = function sayHi() {       # 第二輪做事，印出裡面的東西
>     console.log()                  
> }
>
> BUT!!!!!!!，上面這樣會出錯，會發生initialization的問題，原因就是TDZ的問題
```
  
#### 2-7-2、範例第二題 - 不用const宣告函數

JS執行function的順序，在function裡面，1a、1b、2都會一起做好
  
```markdown
> 第一階段
> sayHi()                   # 不執行
> function sayHi() {        # 執行 - sayHi執行1a、1b   {}執行2 
>     console.log("hello") 
> }
> 
> 第二階段
> sayHi()                   # 執行
>
> 印出 hello
```  
=> 因爲第一階段{}會執行2(執行函數賦值)，因此第二階段執行的時候，才可以成功執行  
    

:::info
如果今天你是直接設定一個fc，不管在哪裡呼叫都可以使用  
但假如今天你是用const / let 設定一個變數，記得要後呼叫才能成功使用該fc  
:::

  
  
  
3、scope是什麼
------

### 3-1、var = function scope  
var 會被 function 包住，但是無法被 {}block 包住   

### 3-2、let/const = block scope  
let/const 只要有一個大括號就可以把他包住  



### 3-3、情境題一
在 if 回圈裡面設定 const 會怎麼樣

```markdown
> var age = 20
> if (age >= 18) {
>     const message = "未成年"          # is not defined
> } else {
>     const message = "已成年"          # is not defined
> }
> 
> console.log(message) -> 因為const被{}包住，所以噴會出is not defined
```

這一題解法很簡單，只要在if前面先宣告message     
```markdown 
> var age = 20
> let message;               -> 先宣告，不要給值
> if (age >= 18) {
>     message = "未成年"
> } else {
>     message = "已成年"
> }
> 
> console.log(message) -> # 未成年，這樣就可以跑了，因為前面先宣告過
```


### 3-4、情境題二
在 function 裡面設定 var 、 let 會怎麼樣

```markdown
> function hello() {
>     var a = 1
>     let b = 2
> }
> hello()
> 
> console.log(a)    會出錯 is not defined，因為var和let都被限定在function中
> console.log(b)    會出錯 is not defined，因為var和let都被限定在function中
```



### 3-5、全域變數

瀏覽器裡面有一個全域物件叫做 window     
node 世界裡面也有一個全域物件叫做 global    
```md
> window.console.log("aaa")  
> window.alert("123")  
```

如果今天在網站上宣告一個 var ccc = 123      
這個變數ccc會存進window裡面     
    
但是，如果今天在網站上宣告一個 let ddd = 123     
這個變數ddd **不會** 存進window裡面     

:::tip
簡單的說var宣告會造成全域物件的污染   
:::

:::info
**那如果我們直接使用賦值 -> age = 20 呢**  
假設window裡面原本沒有age這個變數     
age = 20 這個動作會直接生出一個 變數age 丟在window環境裡面    
  
**這樣會造成很多問題，所以千萬不要輕易的宣告全域變數**    
  
:::




4、let/const vs var
------

### 4-1、三個比較表
|特性|var|let/const|
|:-:|:-:|:-:|
|scope|function scope|block scope|
|變數宣告|可|不可|
|全域屬性|會|不會|
|變數提升|會|會(但是有TDZ壓住)|




### 4-2、函數中丟引數設定變數
```markdown
> 情境題
> function aaa(x) {
>     x = 1
> }
> 
> aaa(123)
> 
> console.log(x)
> 
> 會印出ReferenceError: x is not defined，因為引數123又丟回去x
```




5、多種函數寫法
------

### 5-1、函式function + console.log

#### 5-1-1、寫法一: 普通寫法
```markdown
> const sayMyName = function (name) {
>     console.log(name);
> }
```

#### 5-1-2、寫法二: () => 箭頭函數 Arrow Function
```md
> const sayMyName = (name) => {
>     console.log(name);
> };
```


### 5-2、函式function + 回傳值return - 多種寫法

```markdown
> 下面三個寫法答案都一樣

> 寫法一 -> 最完整寫法
> const isAdult = function (age) {
>     return age >= 18
> }

> 寫法二 -> 省略function + 箭頭寫法
> const isAdult = (age) => {
>     return age >= 18
> }
 
> 寫法三 -> 最簡略寫法
> const isAdult = (age) => age >= 18;
```


:::tip
箭頭函數不單只是function的簡寫  
:::



6、回傳值 return
------

(1) 回傳值是最終產出的值，console.log，不是回傳值，只是把過程印出來  
(2) return後面接的東西都會被忽略，因為return完，函數就結束  


### 6-1、return 範例
```markdown
> function isAdult(age) {
>     if (age>=18) {
>         return "已成年"
>     } else {
>         return "未成年"
>     }
> }
> 
> console.log(isAdult(20))       -> 用return後，在外部呼叫記得
```

:::info
在function裡面要有回傳值，一定要加一個 return ，要不然外面呼叫函數的話，會是undefined  
:::


### 6-2、REPL是什麼
:::tip
REPL 是一種環境 -> 輸入一串程式碼，會自動幫你把東西印出來(ex. 瀏覽器、irb)    
read evaluate print loop      
      
如果今天在瀏覽器的console輸入 console.log("123")      
會印出 "123"、undefined   
  
後面那個undefined就是REPL幫你印出來的，因為console沒有回傳值      
    
:::



### 6-3、實戰 - function + return練習

通常在函數裡面，只會設定true or false，印出值會在另外的判斷式寫

```md
> function isAdult(age) {
>     if (age>=18) {
>         return true
>     } else {
>         return false
>     }
> }
> if (isAdult(20)) {
>     console.log("已成年")
> }
```



### 6-4、實戰 - 寫一個BMI計算器，並四捨五入到小數點第二位

```md
> function BMI(weight, height) {
>     let w = weight
>     let h = height / 100
>     return Math.round(w / (h**2)*100)/100
> }
> console.log(BMI(70, 170))
```
Ps. 記得Math.round只會四捨五入取到整數位，所以如果想取到小數第二位，只要先乘100，再使用Math.round，最後再除以100就可以完成.


7、陣列
------

### 7-1、陣列語法

#### 7-1-1、陣列裡面可以放各種形態
```markdown
> let data = [123, ["2","233"], "kfs"]
```

#### 7-1-2、印出陣列的長度
```md
> data.length    -> 3
```

#### 7-1-1、取陣列最後一個元素
```md
> console.log(data[data.length - 1]);
> Ps. 先算整個陣列長度，在-1，就可以取到最後一個數
```




### 7-2、push 語法 - 陣列尾部新增

原始陣列
```md
> let data = [123, ["2","233"], "kfs"]
> console.log(data)                      # [123, ["2","233"], "kfs"] - 原始陣列值
```

使用push塞一個值
```markdown
> data.push("x")                        -> 在最後面塞一個值
> console.log(data)                      # [123, ["2","233"], "kfs", "x"]
```

**push方法的回傳值是給新值加進去後的length，要注意**
```markdown
> console.log(data.push("x"))            # 4 (回傳 data.length) 
```

### 7-3、unshift 語法 - 陣列頭部新增
```markdown
> data.unshift(1)                        -> 在最前面塞一個值
> console.log(data)                      # [1, 123, ["2","233"], "kfs", "x"]
```

### 7-4、pop 語法 - 從尾巴抽掉元素
```markdown
> data.pop()                             -> 尾巴拿掉一個值
> console.log(data)                      # [1, 123, ["2","233"], "kfs"]
```

### 7-5、shift 語法 - 從頭部抽掉元素
```markdown
> data.shift()                           -> 頭部拿掉一個值
> console.log(data)                      # [123, ["2","233"], "kfs"]
```

### 7-6、Array[x] - 取代陣列中的某個值
```markdown
> data[1] = "帥哥"                        -> 替換掉索引值為1的元素
> console.log(data)                      # [123, "帥哥", "kfs"]
```

### 7-7、splice(start, deleteCount, item1, item2......) - 切片語法

(1) 第一個是起始值      
(2) 第二個是想要刪除的數量      
(3) 第三個以後是想要在刪除值的地方塞進哪些值    


#### 7-7-1、第一個參數使用
```markdown
> result = data.splice(1)                    -> 回傳值 = 取從指定索引值後面的所有元素
> console.log(result)                        # ["帥哥", "kfs", 324]
> 
> **原始陣列值**
> console.log(data)                          # [123]
```

#### 7-7-2、第二個參數使用
```markdown
> result = data.slice(1, 2)                  -> 回傳值 = 從指定索引值後，刪除指定元素數量
> console.log(result)                        # ["帥哥", "kfs"]
> 
> **原始陣列值**
> console.log(data)                          # [123, 324]
```

#### 7-7-3、第三個參數使用
```markdown
> result = data.slice(1, 2, "xx", "孫悟空")   -> 回傳值 = 從指定索引值後，刪除指定元素數量
> console.log(result)                        # ["帥哥", "kfs"]
> 
> **原始陣列值**
> console.log(data)                          # [[123, "xx", "孫悟空", 324]
```



### 7-8、重點 - 要記住所有方法後都會有一個回傳值

以下範例是pop的回傳值，就是拔掉的該值
```markdown
> let data = ["1232", 1,2,"3", "巴巴"]
> const result = data.pop()
> 
> console.log(data);                      # ["1232", 1,2,"3", "巴巴"]
> console.log(`新陣列本身 = ${data}`);      # 新陣列本身 = "1232",1,2,"3"
> console.log(`回傳值 = ${result} `)       # 回傳值 = 巴巴
```

