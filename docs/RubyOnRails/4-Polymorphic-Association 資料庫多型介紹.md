---
title: ROR/Polymorphic-Association 資料庫多型介紹
sidebar_label: "4. [ROR] Polymorphic-Association 資料庫多型介紹"
description: Polymorphic-Association 資料庫多型介紹
last_update:
  date: 2023-11-07
keywords:
  - ROR
  - Polymorphic-Association
sidebar_position: 4
---


:::tip
**前情提要：**    
由於之前在跟助教討論專案的時候，有提到如何把 `Model` 的複雜度提高，那時候就有提到"多型"這個關鍵字  
因此決定來寫一篇文章，來搞懂這個東西到底是什麼？為什麼可以提高Model的複雜度   
:::



1、多型態 Polymorphic Associations
------

### 1-1、名詞定義    
要了解一個名詞是什麼，最好的方式就是從官方文件下手，所以我先把官網的原文放上來並附上翻譯        
[官方文件連結](https://guides.rubyonrails.org/association_basics.html#self-joins)   


:::tip
**英文原文**  
A slightly more advanced twist on associations is the polymorphic association.    
With polymorphic associations, a model can belong to more than one other model, on a single association.  
For example, you might have a picture model that belongs to either an employee model or a product model.  
Here's how this could be declared:    
  
**翻譯**  
稍微更加複雜度更高的關聯表，就是多態性的關聯。    
使用多態性的關聯得時候，在單個關聯上，一個model，可以屬於其他多個model    
舉個例子，你可能會有一個圖片model，而這個圖片model是屬於員工model、產品model  
:::


### 1-2、官網範例    
實際來寫model試試，首先讓大家知道，圖片、員工、產品之間的關聯   
(1) 圖片model有一個imageable欄位，並且model自己是屬於imageable，並且開啟多型態  
(2) 一個員工會有很多張圖片，以imageable當作基準  
(3) 一個產品會有很多張圖片，以imageable當作基準  

```md
> class Picture < ApplicationRecord
>   belongs_to :imageable, polymorphic: true
> end
> 
> class Employee < ApplicationRecord
>   has_many :pictures, as: :imageable
> end
> 
> class Product < ApplicationRecord
>   has_many :pictures, as: :imageable
> end
```


:::tip
**英文原文**  
You can think of a polymorphic belongs_to declaration as setting up an interface that any other model can use.    
From an instance of the Employee model, you can retrieve a collection of pictures: @employee.pictures.    
  
Similarly, you can retrieve @product.pictures.    
  
If you have an instance of the Picture model, you can get to its parent via @picture.imageable.   
To make this work, you need to declare both a foreign key column and a type column in the model   
that declares the polymorphic interface:  

**翻譯**  
你可以把多型belongs_to的寫法，當作其他model都可以使用的連接處
以員工產生的實體為例，你可以在終端機這要打@employee.pictures，把所有員工的圖片拉出來

同樣的，你也可以對產品下一樣的指令@product.pictures   

如果你的圖片有一個新的實體，你可以藉由輸入@picture.imageable，來獲取這個實體的父層級  
為了要讓多型態啟動，你需要在設定多態連接處的model中，多設定一個外鍵(foreign key)、類別(type)  
:::



### 1-3、Picture table設定   
接著我們來設定Picture的Migration，照片會有以下這些欄位  
```md   
> class CreatePictures < ActiveRecord::Migration[7.0]
>   def change
>     create_table :pictures do |t|
>       t.string  :name                                  
>       t.bigint  :imageable_id                             -> 多設定一個外鍵，bigint是比較大數字的意思
>       t.string  :imageable_type                           -> 多設定一個type欄位
>       t.timestamps
>     end
>     
>     add_index :pictures, [:imageable_type, :imageable_id]
>   end
> end
```

接著我們來把上面的Migration具現化，也就是rails db:migrate   
會發現imageable那個欄位，後面會有polymorphic: true  
```md
> class CreatePictures < ActiveRecord::Migration[7.0]
>   def change
>     create_table :pictures do |t|
>       t.string :name
>       t.references :imageable, polymorphic: true
>       t.timestamps
>     end
>   end
> end
```

### 1-4、官網圖片範例
我們來看看這樣設定完後，Table的關聯實際上長的樣子   
![多型架構](./img/polymorphic.png)

***



2、換成自己的話解釋多型態
------

不過怕上面看官網的說明還是不太懂，這邊再做另一個例子。          

每個消費者有很多則留言，每個商店老闆也有很多則留言，因此這時留言就同時屬於消費者和老闆的model，但是留言model又不像多對多的第三張表(第三張表是紀錄另外兩張表的關係，而留言model是單純都屬於另外兩個的model)，這時候就把留言的model，當作一個公開的連接處，提供給其他的model來使用       

### 2-1、範例解析
(1) 有一個留言的Model，自己有一個欄位是commentable，這個欄位就是當作公開的連接處    
(2) 消費者有很多的留言，他就是用comment裡面的commentable來存取留言(消費者自己的欄位有一個name欄位)  
(3) 老闆也是同上    

### 2-2、Model關聯設定
```md
> class Comment < ApplicationRecord             
>   belongs_to :commentable, polymorphic: true
> end
>  
> class Customer < ApplicationRecord
>   has_many :comments, as: :commentable     # 透過as 來串接
> end
>  
> class Manager < ApplicationRecord
>   has_many :comments, as: :commentable     # 透過as 來串接
> end
```


### 2-3、實際寫入資料庫

這樣建好關聯後，我們就可以來查找資料了  

(1) 首先我想建立消費者的留言    
```md
> c1 = Customer.create(name: "魯夫")            # 先創造一個消費者
> c1.comments.create(content: "我要成為海賊王")   # Comment裡面有一個欄位是content，因此把資料存去進

> < Comment id: 1, content: "我要成為海賊王", commentable_id: 1, commentable_type: "Customer" >
> Ps. c1.comments create後，會發現Comment的model，寫進了這些資料、還有就是後面有時間標記，我省略掉了 
```

(2) 接著我們再來寫進老闆的留言，老闆的留言，我們這邊創造兩個實體(也就是老闆留言兩次)    
```md
> 第一則留言
> m1 = Manager.create(name: "紅髮")         # 先創造一個老闆
> m1.comments.create(content: "給個面子")    # Comment裡面有一個欄位是content，因此把資料存去進
> < Comment id: 2, content: "給個面子", commentable_id: 1, commentable_type: "Manager" >

> 第二則留言
> m2 = Manager.create(name: "雷神")         # 先創造一個老闆
> m2.comments.create(content: "給我黃金")    # Comment裡面有一個欄位是content，因此把資料存去進
> < Comment id: 3, content: "給我黃金", commentable_id: 2, commentable_type: "Manager" >

> Ps. 這樣可以發現，存在Comment資料表裡面的commentable_id，會因為type不同，而有不同的計數
```


前面我從消費者、老闆的角度建立留言，接下來我想要透過commentable來反查老闆、消費者都留了哪些留言  
```md
> c1 = Comment.first              # 先抓出第一則留言
> c1.commentable                  # 我要知道是誰的留言，是老闆還是消費者的

> < Customer id: 1, name: "魯夫" >
```




3、心得結論
------

看起來多型真的蠻方便的，假如今天很多表格都要放圖片，又或者是今天有很多不同的角色，都會做留言的動作，就可以透過多型的model來製作。   

再來回到一開始寫這一篇文章的目的，主要是了解多型是否可以增加model的複雜度，自己理解過後，看起來是真的可以把model弄的很複雜，因為在自己的model裡面做一個外鍵，等於可以對自己的欄位做查找，如果今天有很多角色做留言的話，model的確可以變得很複雜。    




