---
title: ROR/Ruby
sidebar_label: "DAY23. [ASTROCamp] Ruby-1"
description: Ruby
last_update:
  date: 2022-11-08
keywords:
  - 5xRuby
  - Ruby
sidebar_position: 24
---


1、工作分配
------

```md
> 看板便利貼

> Todo Doing Done
> 口口口 > 拿票放到右邊 口口 > 做完放到這 口
> 口口口

> 每一個任務都會有一個便利貼，功能盡量拆分小一點
```

### 1-1、難度計量表

每一個 task 便利貼都會有主題、描述、預計完成時程(diff)

會有一個計量表，1~5，拿這個便利貼的時候，就代表你覺得完成此功能會耗時多久

1 - 一天內
2 - 一天半
3 - 兩天內
4 - 兩天半
5 - 三天

### 1-2、注意事項

1. 如果今天你拿便利貼，又寫上 5 分的話，代表完成任務時間大約要耗時 2.5 天

2. 給點數的時候，要讓大家討論，不是一個人決定就是這樣，如果今天兩個人給點數差距過大，要討論一下做法

3. 打點數的時候，不要看別人臉色，給自己覺得分數

4. 任務評分完後，會有一個總點數(是大家估出來的)，並用這個總點數，跟距離 demo 天數，算一下完成點數的總點數理想值(1 個人一天可以耗盡 2 點)，如果最後超過完成日，記得先把功能拔掉

5. 用燃盡圖來看專案進行狀況

6. 功能系統如果沒有太細分，會發生點數 4, 5 分太多。ex. 會員系統如果就是一張票，那就會超級難，把此功能拆得少一點，就可以讓大家都參與進來，並把點數降低

7. 最重要的是，每張卡片都有點數後，開始拿點票後，要怎麼分呢?? !!!每個人一次拿一張票，做完後再去拿新票

8. 如果有人在時限沒做完，怎麼辦 -> 站立會議

9. 會員系統基本會先做，因為他是路障型的票(一定要先做)，路障型票要先清掉，後續才能繼續進行(這種票基本上由熟練的人來做，要不然後續無法接上)

### 1-3、站立會議

每日開會的時候，每個人回報三件事

1. 昨天在幹嘛
2. 遇到的困難
3. 今天要做啥

如果拿了某張票，但是後來覺得做不出來，就要在站立會議提出，並把票放回去，去領新票

:::tip
todo 欄位的票是動態新增的  
:::


***

2、rails
------

rails 強項  
  
1. CRUD = Create, Read, Update, Delete  
2. 關聯性 has one, has many, belongs to  

:::tip
找工作用 rails 6 比較好，7 刪除的動作改了  
:::

### 2-1、開啟新專案

*6.1.7*的意思是，指定開啟新專案的 rails 版本

```md
> rails _6.1.7_ new WishSite
```

Ps. 這次專案我建立在 day23 的資料夾

:::tip
> 如果今天想看電腦有哪些版本的 ruby  
> gem list | grep rails  
> `|` -> 意思是 pile ，透過管線，把輸出結果丟給下面的程式  
:::


#### rails 檔案介紹

1. .ruby-version -> 這個檔案，只要 cd 進來，可以控制你進來的 ruby 版本  
2. gem.file 會幫你整理所有用到的套件，如果要新安裝套件，請放在這裡  
  
```md
> bundle install    # 會直接把整個 gem file 的檔案掃過並安裝相容的檔案
```
  
:::info
ruby gems => ruby 所有的套件都在這個網站  
:::



3、gem、bundle
------

gem 版本介紹、和bundle有什麼不一樣、Gemfile.lock是用來做啥的  

### 3-1、gem版本代表的意思
設計多版本的原因是，如果今天加小物件，跳patch版號，如果今天是加新功能，跳minor版號，如果大更新，跳major版號

```md
> gem 'xxx' 3.1.7
> 3 => major => 完全不同的產品  
> 1 => minor => 有可能會壞掉    
> 7 => patch => 增加不太重要的功能 （可以隨便更新來用）

> gem 'xxx', '~> 3.1.7' 3.2.0 不會裝
> gem 'xxx', '>3.1.7' 3.2.0 會裝
```

:::tip
`~` 的意思是，會去裝比較安全的版本，所以今天如果改動的是patch就會安裝，跳minor版號就不會安裝  
::: 

### 3-2、gem vs bundle  

(1) gem 是一次裝一個指定的套件  
(2) bundle 是一次下載寫在 gemfile 的描述檔  

```md
> bundle指令會做以下事情

> (1) 找 gemfile  
> (2) gem install ...  
> (3) 產生 Gemfile.lock  
```
  
### 3-3、Gemfile.lock 是啥

假設今天A套件需要B套件2.0版本，C套件需要B套件3.0版本，bundle完後，可以解決上面這個問題      

```md
> Gemfile.lock => 會產生一個“多個檔案相依性”的檔案，這可以解決版本不同的相依性問題  
```



4、ruby for rails
------

### 4-1、gem 是什麼

(1) rvm -> ruby 系統管理工具    
(2) gem -> ruby 套件安裝工具    
  
:::tip
> **ruby 安裝套件是直接裝在系統裡面**  
> gem 完套件會直接裝在環境裡面  
> 可以輸入 gem env 看環境裡面有哪些變數   
>  
> gem env = gem environment  
::: 
  
  

5、Repl(Read-eval-print loop)是啥
------
### wiki
「讀取-求值-輸出」循環（英語：Read-Eval-Print Loop，簡稱REPL）  
也被稱做交互式頂層構件（英語：interactive toplevel），是一個簡單的，交互式的編程環境。  
這個詞常常用於指代一個Lisp的交互式開發環境，也能指代命令行的模式。  
  
### 懶人包
REPL對於學習一門新的程式語言具有很大的幫助，因為它能立刻對初學者做出回應。  
  
  
  
  
6、ruby常數和變數的差異
------
  
(1) 常數是大寫字母，變數是小寫開頭  
(2) 常數可以re-assign(但是不建議，會跳出警告)   


**為什麼ruby要讓常數可以修改，原因是物件導向的class，常數命名且可以修改**
```ruby
class Cat    # 這個Cat是常數，在ruby，我們可以幫class新增、修改功能
end
```
    
  
:::tip
`` -> ruby這個符號是執行某一段指令  
result = `ls -al`   
puts result => 可以把所有檔案列出來  
::: 

  


7、p、puts、print 的差異
------

先創一個陣列
```md
> lst = [1,2,3]
```

### 7-1、p
(1) p會保留要印出來的殼，並印在一行     
(2) 會有回傳值    
```md
> p lst        # [1,2,3]
```

### 7-2、puts
(1) puts 會整個拆掉，並逐行印  
(2) 沒有回傳值  
```md
> puts lst     # 1
>                2
>                3
```

### 7-3、print
(1) 跟puts很像，不過不會換行    

這邊在宣告第二個陣列    
```md
> lst2 = [4,5,6]
```

同時把兩個陣列印出來
```md
> print lst 
> print lst2        # [1,2,3][4,5,6]   -> 因為print關係印在同一行    
```

:::info
如果今天p用在方法裡面，他會有回傳值的效果，但是千萬不易這樣用，不要用p來取代return  
:::



8、early return
------
  
能不寫else就不寫else  
```ruby
def is_greater_than_20(n)
  if n > 20
    return true
  end
  return false
end
```




9、錯誤訊息
------

### 9-1、JS的錯誤訊息
JS中，抓取錯誤訊息 - catch
```md
> function hi() {
>     // Exception
>     throw "aaaa"               -> throw是把一個錯誤丟出來
> }
> 
> try {                          -> 嘗試執行下面的fc
>     hi()
>     catch (e) {                -> 如果有錯，做下面的事情
>         console.log("error")
>     }
> }
```

### 9-2、Ruby
Ruby中，抓取錯誤訊息 - rescue

```md
> def bmi_calc(w, h)
>   w / h                     -> 不要把錯誤判斷寫在這，因為讓其他人來幫忙判斷是否錯誤
> end
> 
> begin
>   puts bmi_calc(180, 70)
> rescue                      -> 救援功能，如果輸入值有問題，執行下面那一段
>   puts "error"
> end
```
  
  
rescue簡化寫法
```md
> def bmi_calc(w, h)
>   w / h                   
> rescue => exception          -> exception寫法 
>   puts "error"
> end
```



10、幫陣列、字串擴充功能
------

### 10-1、JS的prototype
  
JS用prototype可以幫物件、陣列、字串...加功能  
```md
> String.prototype.is_valid = 123
> 
> console.log("a03456".is_valid)                  # 123
```

### 10-2、ruby的class
  
使用class可以幫任何物件加上功能  
```md
> class String
>   def is_valid
>     p  123
>   end
> end
> 
> p "a03456".isis_valid                           # 123
```



