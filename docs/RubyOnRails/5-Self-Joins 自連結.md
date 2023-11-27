---
title: ROR/Self-Joins 自連結
sidebar_label: "5. [ROR] Self-Joins 自連結"
description: Self-Joins 自連結
last_update:
  date: 2023-11-07
keywords:
  - ROR
  - Self-Joins
sidebar_position: 5
---


:::tip
**前情提要：**    
由於不太清楚自連結要怎麼設定，就算看完官方手冊，還是不知道他在講啥，因此打算以自己的方式解釋他的運作原理  
:::  


1、自連結 Self-Join
------

雖然看了官方文件，還是不了解，但是還是先從官方文件做起手，所以我先把官網的原文放上來並附上翻譯      
[官方文件](https://guides.rubyonrails.org/association_basics.html#self-joins)   


### 1-2、基本概念
:::tip  
**英文原文**  
In designing a data model, you will sometimes find a model that should have a relation to itself. 
For example, you may want > to store all employees in a single database model, but be able to trace   
relationships such as between manager and > subordinates. This situation can be modeled with self-joining associations:   
  
**翻譯**  
在設計model的時候，你有時候會發現一個model應該要與自身欄位有關聯。    
舉個例子，你今天想要把所有員工存在一個資料表中，但是你想要這張表可以追蹤上司、下屬間的關係    
這種情況就可以使用到自連結    
:::


### 1-2、Model關聯
接著到Employee的model來設定一些關聯 
```md
> class Employee < ApplicationRecord
>   has_many :subordinates, class_name: "Employee",
>                           foreign_key: "manager_id"
> 
>   belongs_to :manager, class_name: "Employee", optional: true
> end
```


:::tip
**英文原文**      
With this setup, you can retrieve @employee.subordinates and @employee.manager.   
In your migrations/schema, you will add a references column to the model itself.  
  
**翻譯**  
經由剛剛上面那樣設定，我們可以得到下面兩個方法，取得一些資料  
(1) @employee.subordinates    
(2) @employee.manager     
並且在你的migration、schema中，你將會在自己的欄位中，加上一個檢索值   
:::


### 1-3、Migration內容
再來處裡migration   
```md
> class CreateEmployees < ActiveRecord::Migration[7.0]
>   def change
>     create_table :employees do |t|
>       t.references :manager, foreign_key: { to_table: :employees }
>       t.timestamps
>     end
>   end
> end
```


也因為官方文件這只有說到這，所以一開始看完覺的滿頭霧水，因此下面會用自己的方式解釋Self-Join


***


2、換成自己的話解釋Self-join
------

範例一樣用員工、上司、下屬，三者的關係來解釋自連結


### 2-1、基本概念詳解    
首先我有一家公司，而我這一家公司的員工所有資料都在一張Employee表裡面，而公司理所當然的都有管理層、基層員工.....等等的分別   
因此當我把員工所有資料都放在這張表單的時候，我想要知道到底誰是管理層，誰是基層員工呢？  

這邊有兩個情境，來解釋員工中的關係從屬  
(1) 一名員工(Employee)，會向另外一名員工(Employee)報告，後面這個聽報告的員工，另外一個身份就是主管(manager)  
(2) 一個員工(Employee)，會監督其他的員工(Employee)，後面這個被監督的員工，另外一個身份就是下屬(subordinates)    


:::tip
**自連結其實就是同一張表內，自己的Primary Key，會被當作其他人FOREIGN KEY**    
舉個例子，假設我今天有一個員工(Employee)，他的id是20051，接著我有另外一個員工，他的id是20073，這個員工有一個上司，他上司的        
ID就是剛剛前一號員工，因此這一張表有另一個欄位叫做EMP_SUPV，後面那個員工EMP_SUPV欄位的值，就是20051   
:::

如果看不懂上面那段的話，可以這個連結，裡面有自連結很詳細的介紹：[自連結介紹](https://medium.com/@asuthamm/self-join-in-rails-8e3fc99c0634)





### 2-2、產生Model    

首先創造一個Employee的model，裡面會有一個name欄位，代表這個員工的名字   
```md
> rails g model Employee name
```

### 2-2、更新migration
接著到migration加上manage_id的欄位，這個欄位就是之後要存放每一個員工的主管是誰(用Employee的id來表示)
```md
> def change
>   create_table :employees do |t|
>     t.string :name
>     t.references :manager, foreign_key: { to_table: :employees }       # 就是這一行！！
>     t.timestamps
>   end
> end
```

### 2-3、具現化
新增好後就可以把表格具現化了
```shell
$ rails db:migrate
```

### 2-4、Model寫關聯
再來可以到model把關聯寫上去了，這邊會有兩個主要的關聯，一個是subordinates、一個是manager
```md
> class Employee < ApplicationRecord

>   // 一個員工有很多下屬，對到的表是自己這張表 -> 使用的外鍵是manager_id (存放資料下屬id的地方)
>   has_many :subordinates, class_name: "Employee", foreign_key: "manager_id"
 
>   // 一個員工會有一個老闆，對到的表是自己這張表   
>   belongs_to :manager, class_name: "Employee", optional: true  

> end
```

### 2-5、console測試、創造多個員工
那實際應用的狀況是怎麼樣呢？我們來在console的環境跑一次。   
進到環境後，我們先創造出五個員工
```shell
$ rails c

------
e1 = Employee.create(name: "魯夫")
e2 = Employee.create(name: "索隆")
e3 = Employee.create(name: "那美")
e4 = Employee.create(name: "喬巴")
e5 = Employee.create(name: "羅賓")
```

### 2-6、測試subordinates方法
先來試試看subordinates這個方法，此方法其實代表的就是，如果今天e1是主管，那我可以用這方法，找他的下屬有哪些人    
```md
> e1.subordinates               # 這樣打會傳一個空陣列給你，原因是因為我們還沒把下屬給e1主管

> e1.subordinates << e2         # 我把e2員工指定給e1，這樣的意思就是，"索隆"的主管是"魯夫"
> e1.subordinates << [e3, e4]   # 這個意思是我要一次指定e3、e4這兩位員工，當作e1的主管，也就是那美、喬巴的主管都是魯夫
```


### 2-6、測試manage方法
再來試試manager的方法，此方法代表的則是，如果我今天是一個員工，我要指定我的老闆是誰(跟剛剛的邏輯相反)   
```md
> e5.manage = e1
> e5.save

> 這的意思是，e5的老闆是e1，也就是羅賓的老闆是魯夫，在羅賓的manage_id那邊，顯示的id就會是魯夫的id
```



3、心得結論
------

翻了其他英文文章、自己實際動手寫關聯後，才終於了解self-join的強大之處，因為可以直接在同一張表上，就讓資料有階層的關係，查找起來也非常的方便。
