---
title: ROR/Ruby
sidebar_label: "DAY24. [ASTROCamp] Ruby-2"
description: Ruby
last_update:
  date: 2022-11-10
keywords:
  - 5xRuby
  - Ruby
sidebar_position: 25
---


1、ruby hash
------

### 1-1、新增hash

舊版
```ruby
h = { :name => "ccc", :age => 18 }
```
  
新版
```rb
h = { :name: "ccc", :age: 18 }
```

不過其實把 h 印出來，其實還是跟舊版一樣，新版的寫法只是語法糖，實際是沒改的
```rb
p h             # { :name => "ccc", :age => 18 }
```


### 1-2、hash取值

記得取值要特別注意要放符號，要不然value印不出來

錯誤示範
```rb
p h["name"]  # X
```

正確示範    
```rb
p h[:name]   # "ccc"
```

### 1-3、取出所有的key、values

先創一個新hash
```rb
profile = {:name: 'ccc', :age: 18}
```

取keys或者values值
```rb
p profile.keys   #[:name, :age]
p profile.values   #["ccc", 18]
```

2、Symbol 符號
------
  
**ruby中的的數字object_id，所有數字的object_id都是奇數(2n+1)，因此可以推斷出所有字串的object_id都是偶數**  
  
### 2-1、字串的object_id  
```rb
p "hello".object_id         # 60
p "hello".object_id         # 80
p "hello".object_id         # 100
```

### 2-2、符號的object_id  
```rb
p :name.object_id           # 71068
p :name.object_id           # 71068
p :name.object_id           # 71068
```


### 2-3、數字的object_id -> 2n + 1  
```rb
p 100.object_id             # 201
p 100.object_id             # 201
p 100.object_id             # 201
```

符號介紹詳解 - [點我觀看](https://fintechrich.com/what-is-symbol/)



3、簡易測驗 - 計算陣列中，所有字數出現次數
------
```md
> a = [1, 2, 3, 1, 2, 1, 3, 1, 2, 3, 4, 5, 6]
> 
> p a.group_by{ |n| n }.transform_values(&:size) -> 比較複雜的解法
> 
> p a.tally   -> 正確答案是這個
> 
> 會印出 {1=>4, 2=>3, 3=>3, 4=>1, 5=>1, 6=>1}
```


4、區域變數的nameError
------

下面會印不出來喔!!!
因為a是在外面定義變數，ruby不會從外面找答案

```rb
a = 1
def hi
  p a
end

hi            # nameError
```


:::tip **面試題**    
```md
Book.find      => find如果錯誤，會噴出錯誤警告，而且find後面只能接數字  
Book.find_by   => find_by錯誤，只會給一個nil，find_by後面可以接其他的欄位  
Book.find_by!  => Exception，加一個驚嘆號就如果錯誤的話，就會跳錯誤警告給你  
```
::: 




5、rake
------

(1) man = manual 是看某個指令的描述     
(2) 在終端機輸入，可以得知make指令在電腦是幹嘛的     
```md
> man make
```
(3) 要介紹rake錢，為啥會提到make，因為 rake = ruby make，輸入rake，電腦會要你產生一個rakefile的檔案      
  

### 5-1、default task
如果今天有設定預設任務，那假設直接下rake指令，會直接執行hi這個task  
```md
> task :default => :hi
```

### 5-2、hello任務設定
```rb
desc "見面"                 # task的描述
task :hello do             # 任務名稱
    p "hello world"        # 任務執行內容
    p "1"
    p "2"
end
```

### 5-3、任務相依性
做hi之前，先做hello動作
```rb
desc "這是測試"
task :hi => :hello do      # 執行hi前，先執行hello
    p "hello world"
end
```


### 5-4、namespace

實作一個db:migrate   
用namespace把task包住，呼叫此任務就變成這樣  
```rb
namespace :db do       # namespace是用來分類的
  desc "migrate"       # 描述
  task :migrate do     # 任務名稱
    p "good"
  end
end

rake db:migrate        # 印出good(呼叫任務)
```



6、Block
------

:::tip **Block是什麼？**
Block其實就是一段程式碼，更精確的說   
**block是一段不會被主動執行的程式碼**    
::: 



### 6-1、yield

Block會依附在方法後面，會不會執行要看原本宿主的臉色

#### 6-1-1、沒有yield
```md
> 我們先來定義一個方法（還沒有加上block）

> def say_hello            #控制權"2"
>   puts "hi，早安"         #控制權"3"
> end
> say_hello                #控制權"1"
> puts "午安"               #控制權"4"              #hi, 早安
>                                                  午安
>                                                  (照控制權印出這兩個)

> 接著讓方法加上block
> 
> def say_hello            #控制權"2"
>   puts "hi，早安"         #控制權"3"
> end
> say_hello {              #控制權"1"
>   puts "晚安"             # -沒有執行- 原因是def say_hello end沒有yield
> }                
> puts "午安"               #控制權"4"              #hi, 早安
>                                                  午安
```
#### 6-1-2、加上yield
```md
> 加上yield後的執行流程

> def say_hello            #控制權"2"
>   puts "hi，早安"         #控制權"3" ->印出 1 
>   yield                  #控制權"4"
>   puts "睡覺拉"           #控制權"6" ->印出 3
> end
> say_hello {              #控制權"1"
>   puts "晚安"             #控制權"5" ->印出 2
> }
> puts "午安"               #控制權"4" ->印出 4       
>                                                   #hi, 早安
>                                                    晚安
>                                                    睡覺拉
>                                                    午安
```


:::tip **yield是什麼？**
就是把控制權轉讓給方法後面的block    
:::



#### 6-1-3、yield帶參數
用yield轉讓的同時，也可以帶上其他的變數 or 數字
```md
> def say_hello            
>   puts "hi，早安"         
>   yield 3                # yield 帶著其他東西
> end

> say_hello { |num|        # 控制權轉移到Block時， 3也被帶下來到 num
>   puts num               # 印出 num = 3                           # 3
> }
```


#### 6-1-4 block回傳結果給yield
Block完成後，會自動回傳Block裡面的最後一行執行結果(傳回yield那邊)，接著可以用if判斷要印出的東西
```md
> 觀察控制權

> def test_three                     #      2
>   if yield(3)                      #      3
>     puts "it's 3"                         5  -true的話就是這個
>   else
>     puts "it's not 3"                     5  -false就是這個
>   end
> end

> test_three { |n|                   #控制權 1
>   n == 3                           #      4  -這一步驟會判斷true/false
> }                                            
>                                    # it's 3 - 最終印出結果
```


#### 6-1-5、沒block，有yield
假設今天沒有Block，又有寫出yield的話會發生什麼事 — 會噴錯
```md
> def say_good
>   yield
> end
> say_good                           # X (No block given)
```






### 6-2、block + yield 實戰練習

#### 6-2-1、實作一個map功能
```rb
def my_map(arr)
    lst = []
    arr.each do |n|
        lst << yield(n)
    end
    lst
end

result = my_map([1,2,3,4,5]) {|x| x * 2}

p result
```

#### 6-2-2、實作一個filter功能
```rb
def my_filter(arr)
    lst = []
    arr.each do |n|
        lst << n if yield(n)
    end
    lst
end

list = [1, 2, 3, 4, 5]
result = my_filter(list) { |x| x > 2 }

p result # [3, 4, 5]
```



### 6-3、block面試題


:::tip **do...end、`{}`兩種block哪邊不一樣？**  
Ans.  
結合率強度不同，do...end的結合率強度比較低，像是加號，`{}`的強度比較強，像是乘號  
強度比較弱會發生什麼事，會變成被p搶走  
:::


這樣講有點抽象，舉個例子，先給一個陣列  
```md
list = [1,2,3,4,5]
```

#### 6-3-1、使do...end 
使用do..end的關係，p會把list.map搶走，因此會印出Enumerator這個  
```rb
p list.map do |item|                # <Enumerator: [1, 2, 3, 4, 5]:map>
  item * 2
end
```
  
#### 6-3-2、使用`{}`
```rb
p list.map { |item| item * 2 }      # [2, 4, 6, 8, 10]
```
  





7、Rails Scope設定
------

### 7-1、lambda沒給參數
```rb
class Book
  scope :cheap, -> {  where ("price < 50") }
end

Book.cheap
```

### 7-2、lambda有給參數
```rb
class Book
  scope :cheap, -> (n){  where ("price < n") }
end

Book.cheaper_than(100)
```


### 7-3、預設 Scope

使用預設Scope後，之後這個Book所有的查詢，都會被加上order id這一段搜尋
```rb
class Book
  default_scope {order('id DESC')}
end
```


### 7-4、取消預設 Scope

如果今天設定了預設scope，但是有一個地方的查詢不想用到預設scope怎麼辦？
可以這樣設定取消預設搜尋
```rb
Book.unscope(:order)
```

也可以這樣取消預設後，再增加想要新增的搜尋
```rb
Book.unscope(:order).order(:title)
```
