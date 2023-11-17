---
title: ROR/Ruby
sidebar_label: "DAY25. [ASTROCamp] Ruby-3"
description: Ruby
last_update:
  date: 2022-11-11
keywords:
  - 5xRuby
  - Ruby
sidebar_position: 26
---


1、物件導向概念  
------

### 1-1、函式編程 function programming(FP)  
跟物件導向相反的概念，扁平化的程式設計  
像是React這套JavaScript框架就是使用FP來開發程式，因此若想學習React.js勢必也要熟悉FP的基本概念，而JavaScript程式語言也需要符合FP的編程理念   

### 1-2、物件導向 Object Oriented Programming(oop)  
物件導向是上下階層的程式設計，物件也就是類別的實例，也就是說有了類別這張藍圖我們可以在程式中產生許多汽車類別的資料，而這些資料彼此之間不互相影響，每一個皆是獨立的。  

**物件導向有以下幾個特性**  
#### 1-2-1、(1) 封裝
即是將物件內部的資料隱藏起來，只能透過物件本身所提供的介面(interface)取得物件內部屬性或者方法，物件內部的細節資料或者邏輯則隱藏起來，其他物件即無法瞭解此物件的內部細節，若不經過允許之窗口(即此物件提供之方法)便無從更動此物件內之資料。   

簡白的說，對一件事情只需要理解他的外在就好，不需要了解裡面內部的構造。

#### 1-2-2、(2) 繼承
在某種情況下，一個類別會有「子類別」。子類別比原本的類別(稱為父類別)要更加具體化，也就是說子類別繼承了父類別。例如：計程車(子類別)繼承了汽車(父類別)原有的屬性以及方法，也新增了自己特有的屬性(driverName)。


#### 1-2-3、(3) 多型
簡單來說就是相同名稱的方法(Method)，多個相同名稱的方法，傳入不同的參數，會執行不同的敘述。
多型(Polymorphism)則包含多載(Overloading)和複寫(Overriding)。



:::tip
ruby為二不是物件的  
(1) block     
(2) method     
::: 




2、類別與實體
------

### 2-1、定義類別
```rb
class Cat
  def hi                # 這個hi是實體方法
    puts "hello"
  end
end

kitty = Cat.new         # 用Cat類別生出實體
kitty.hi                # 印出 hello
```

:::tip
Cat.new   ->  這個new是Cat的類別方法  
::: 


### 2-2、繼承(inheritance)

把共通的特徵，放在上層分類(class)
```rb
class Animals
  def eat
    p "好吃"
  end
end

class Cat < Animals     # Cat繼承Animals所有技能
end

kitty = Cat.new
kitty.eat               # 好吃 (因為繼承的關係，所以可以使用Animal的方法)
```


### 2-3、初始化(initialize)

new是去記憶體要一個位置，new完後，馬上執行的事情  
要有初始化這個概念，是因為一執行的時候，就帶一些參數給他  
ps. 有些程式語言把初始化叫做建構子 constructor  

```rb
class Cat
  def initialize
    p "hi"
  end
end

kitty = Cat.new           # hi ()
```

今天想要帶引數給初始化(給生出來的貓咪名字)
```rb
class Cat
  def initialize(name)             # 初始化的時候，預先帶東西給他
    p name
  end
end

kitty = Cat.new("nancy")           # nancy
```



### 2-4、特殊例子 class Object
  
物件創造時，類別都不放initialize會發生什麼事  
  
```md
> class Animals < Object  -> 這個Object是預設隱藏的的
> end
> 
> class Cat < Animals
> end
> 
> kitty = Cat.new         -> 不會壞掉 (因為Animals上層還有一個隱藏class，那個是ruby內建的)
```



### 2-5、實體變數

```md
> class Cat
>   def initialize(name)
>     @name = name
>   end
> end

> kitty = Cat.new("nancy")
> p kitty                     # { 記憶體位置、@name: nancy } - 這一串就是new的時候，去要記憶體位置
```




實體變數可以傳給類別中的其他方法
```md
> class Cat
>   def initialize(name)
>     @name = name
>   end

>   def say_my_name
>     @name                 -> 把初始化的實體變數傳到這個方法
>   end
> end

> kitty = Cat.new("nancy") 
> p kitty.say_my_name        # nancy    
```





### 2-6、attr_reader、writer、accessor

讓ruby可以像JS一樣的寫法，
getter、setter  

```md
> class Cat
>   def initialize(age)
>     @age = age
>   end

>   // getter
>   def age                   # 製作出age方法，可以像JS屬性一樣取到age的值
>     @age
>   end

>   // setter
>   def age=(n)               # 製作出age=方法，這樣可以更新age的值
>     @age = n
>   end
> 
> end

> kitty = Cat.new(18)
> p kitty.age                 # 18 
> p kitty.age=20              # 20
```
  
  
上面寫法太囉唆了，可以用attr_reader、attr_writer簡寫
```md
> class Cat
>   attr_reader :age          # 這一串簡寫了剛剛的 age方法
>   attr_writer :age          # 這一串簡寫了剛剛的 age=方法

>   def initialize(age)
>     @age = age
>   end
>
> end

> kitty = Cat.new(18)
> p kitty.age                 # 18 
> p kitty.age=20              # 20
```
  
  
如果想要更簡短的寫法，可以使用attr_accessor
Ps.剛剛原本的一大串，現在變得超級短

```md
> class Cat
>   attr_accessor :age         # 這一串簡寫了剛剛的writer、reader 

>   def initialize(age)
>     @age = age
>   end
>
> end

> kitty = Cat.new(18)
> p kitty.age                 # 18 
> p kitty.age=20              # 20
```



3、實體方法、類別方法
------

實體方法 = 做用在實體身上的方法
類別方法 = 做用在類別身上的方法


### 3-1、實體方法

```md
> class Cat
>   def fly                  -> 這個目前是實體方法
>     p "I can fly"         
>   end 
> end
> 
> Cat.fly                    # error (貓咪類別沒有fly方法)
```


### 3-2、類別方法
這個self是單體方法來的，下面會提到單體方法
```md
> class Cat
>   def self.fly         -> 這個加上self就是類別方法
>     p "I can fly"
>   end
> end
> 
> Cat.fly                # I can fly
```


:::info **Q. 什麼時候要用實體方法？**  
Ans. 需要生實體出來的時候，就用實體方法  
:::


:::info **Q. 什麼時候要用類別方法？**  
Ans. 不想new一個實體，直接對類別呼叫，並把參數塞進去就好  
Ex. Book.all、Book.where(這兩個都是做用在Book類別上的類別方法)  
:::


### 3-3、單體方法 singleton method

這個方法很少用，直接設定在一個特定物件身上的方法，不過類別方法就是用此法衍伸出來的
```md
> class Cat
> end
> 
> def kitty.hi        => hi只會作用在kitty上
>   p "12323"
> end
> 
> kitty = Cat.new
> kitty.hi            # 12323
```




4、類別變數
------

做用在類別上的變數(這個蠻少用到的)

```md
> class Cat
>   @@count = 0
> 
>   def initialize
>     @@count += 1
>   end
> 
>   def self.total
>     @@count
>   end
> 
> end

> p Cat.total     # 0 (目前count還是0) 
> 
> Cat.new         # 幫Cat類別生出三個實體
> Cat.new
> Cat.new
> 
> p Cat.total     # 3 (最後@@count是3)
```



5、define_method 動態定義方法
------

手動做一個attr_reader方法出來，會用到define_method、instance_variable_get
```md
> class Animals
>   def self.my_attr_reader(attr_name)
>     define_method(attr_name) do
>       instance_variable_get(:"@{attr}")
>     end
>   end
> end
 
> class Cat < Animals
>   my_attr_reader :age
> end

> kitty = Cat.new(18)
> p kitty.age                 #18
```




6、存取控制 public、private
------

為什麼要設計private，因為有些功能不想要人碰到，就像手機，手機有很多功能，但是有些是寫在裡面的商業機密  
就會有private的方法，這些放在private的，都是商業機密，那要怎麼使用這些private方法呢？通常商品會有一些按鍵  
可以觸發這些方法，只是你不會知道內部的方法邏輯  
  
```md
> class Cat
>   def hi                -> 公開的方法
>     hey                 -> 公開方法可以使用私人方法
>   end
> 
>   private               -> 下面的方法都會變成私人方法
>   def hey
>   end
> end

> kitty = Cat.new
> kitty.hey               -> XXX (外部無法使用私有方法)
> 
> kitty.hi                -> 但是使用公開方法(公開方法裡面有使用私有方法)
```


### 6-1、private 額外特色
BUT，因爲private的特性，他是不能在物件前面用加上.，去取用私有方法  
所以假設我們今天用.send去取private中的hey，是可以取到的  

```md
> kitty.send(:hey)
```

7、module
------

先創建一個xx模組，在includes到某一個類別裡面，之後那個類別生出來的物件，都會有該模組的方法  

```md
> module Flyable            # 創建一個模組
>   def fly
>     p I can fly""
>   end
> end
> 
> 
> class Cat
>   include Flyable         # 在這個類別引入模組，新實體就可以使用引入模組的方法
> end
> 
> kitty = Cat.new
> kitty.fly
```




8、superclass、object class
------

利用class、superclass來找尋所有物件的前後層關係

```md
> class Animals
>   def eat
>     p "好吃"
>   end
> end
> 
> class Cat < Animals     # Cat繼承Animals所有技能
> end
> 
> kitty = Cat.new

> kitty.class             # Cat         -> kitty物件的造物主Cat類別
> Cat.superclass          # Animals     -> Cat類別繼承自Animals
> Animals.superclass      # Object      -> Animals繼承自Object類別(這一層開始為ruby內建)
> Object.superclass       # BasicObject -> Object繼承自BasicObject類別(這一層開始為ruby內建)
```




9、ancestors
------

### 9-1、include做了什麼事
先給一個貓類別
```rb
class Cat
end
```

我們用ancestors來查看貓類別，的繼承關係  
這個Kernel就是includes進來的，如果你用superclass來翻找，就會發現你找不到，因為他是用includes插進來的   
  
```rb
Cat.ancestors       # [Cat, Object, Kernel, BasicObject]
```

這樣是不是還有點不明白？沒事，我們親自設計一個module給貓類別  
```rb
module Flyable
  def fly
    p "I can fly"
  end
end

class Cat
  include Flyable
end
```

接著再印一次ancestors   
這邊就可以發現，新創的模組被差進Cat和Object類別之間     
```rb
Cat.ancestors       # [Cat, Flyable, Object, Kernel, BasicObject]
```




10、ruby物件導向model
------

改天把筆記做完
11/11 15:30




11、include、extend的差異
------

include -> 會變成該物件的實體方法  
extend  -> 會變成類別方法  
  
### 11-1、include

使用include插入module，可以用貓類別產生新物件，並讓該物件可以使用此module -> 實體方法
```rb
module Flyable
  def eat
    p 'good eat'
  end
end

class Cat
  include Flyable

end

kitty = Cat.new
kitty.eat              # 實體方法
```


### 11-2、extend

使用extend插入module，可以讓該貓類別，可用該module -> 類別方法

```rb
module Flyable
  def eat
    p 'good eat'
  end
end

class Cat
  extend Flyable

end

Cat.eat                # 類別方法
```


