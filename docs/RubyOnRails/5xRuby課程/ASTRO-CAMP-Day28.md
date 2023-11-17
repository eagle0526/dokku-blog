---
title: ROR/Ruby
sidebar_label: "DAY28. [ASTROCamp] Ruby-6"
description: Ruby
last_update:
  date: 2022-11-17
keywords:
  - 5xRuby
  - Ruby
sidebar_position: 29
---


1、find vs. find_by vs. where
------

### 1-1、find的用法

find只能抓一個，()裡面的就是model的流水編號
```md
> WishList.find(3) 
> 
> 這個find就是抓到該model流水編號為3的實體
```

:::tip
find只能抓流水編號  
:::


#### 1-1-1、find如果抓不到東西
用find，找不到東西的話，會噴ActiveRecord::RecordNotFound


### 1-2、find_by用法
find_by可以抓id以外的東西，除了id以外都可以
```md
> WishList.find_by(id: params[:id]) 
> 
> find_by可以抓id以外的東西，如果今天是要抓model裡面name的欄位，就是下面這樣寫
> WishList.find_by(name: "good")
```

#### 1-2-1、find_by如果抓不到東西
但是用find_by找不到東西的話，只會給你一個Nil

BUT！！如果今天是這樣寫(find_by!)，也會噴錯誤給你
```md
> WishList.find_by!(id: 999)
```


### 1-3、where用法
where抓到的資料是一個**陣列**(給你一堆結果塞到陣列裡面)

```md
> WishList.where(id: 2)

> 印出 [#<WishList id: 2, title: "sdf", description: "qwe">]
```

#### 1-3-1、where如果抓不到東西
where如果找不到東西，會給一個空陣列




2、ApplicationController 在做啥事
------

如果今天所有controller，都有共通行為，就可以直接寫在ApplicationController

### 2-1、狀況一 
如果以後會員的登入狀態，就可以放在這邊

### 2-2、狀況二 
把遇到ActiveRecord的狀況修改都放到這一層，這樣所有controller遇到都可以解決

```md
> class ApplicationController < ActionController::Base
>     rescue_from ActiveRecord::RecordNotFound, with: :record_not_found
> 
>     def record_not_found
>         #   render html: "找不到", status: 404
>           render file: "#{Rails.root}/public/404.html", status: 404, layout: false
>     end
> end
```


3、抽象類別
------

```md
> class Animal        -> 這個東西是抽象類別
> end
> 
> class Cat < Animal
> end
> 
> kitty = Cat.new
> a = Animal.new      -> 用抽象類別產生新實體，ruby沒有支援此用法
```
Ps. 有些語言有支援抽象類別，但是ruby沒有




rails model裡面這個檔案 -> application_record.rb裡面有寫這一句   
他的意思是，不要用此類別產生新實體喔!~   
```md
> class ApplicationRecord < ActiveRecord::Base
>   self.abstract_class = true
> end
```



4、WishList製作功能 - UD
-----

### 4-1、第十三步驟：設定edit
路徑設定routes   
     
今天要來做更新之前存取的檔案，所以先來設定編輯路徑      
如果今天是自己客製路徑，只要在直接用前面action show寫過的路徑，在網址的最後面，加一個edit就好    
```rb
get "/wish_card_info/:num/edit", to: "wish_list#edit_wish", as: "edit_wish_card_info"
```

設定好之後，到index那邊加上link_to  
記得要帶路徑給這過path喔  
```md
> <td><%= link_to "edit", edit_wish_card_info_path(wish_list.id) %></td>    
```


接下來把edit的action、view做好 
因為我們要抓每個實體，所以記得要抓id  
```rb
def edit_wish
    @wish_list = WishList.find(params[:num])
end
```

接下來我們直接複製new的表單，並在controller把find放上去  
```rb
# edit controller

def edit_wish
    @wish_list = WishList.find(params[:num])
end
```

edit view(這邊的路徑後面要改過，要改成update路徑，這邊先放錯誤的，只是要給大家看edit的演變)
```md
> <%= form_for @wish_list, url: "/wish_card_list" do |form| %>
>     <div>
>       <%= form.label :title %>
>       <%= form.text_field :title %>
>     </div>
> 
>     <div>
>       <%= form.label :description %>
>       <%= form.text_area :description %>
>     </div>
> 
>   <%= form.submit %>
> <% end %>
```
     
此時會發現表單都有數據了！！！！原因就是因為假設今天實體是有料的，rails就會判斷你是要來更新這個實體

### 4-2、第十四步驟：設定update

再來更改update的路徑，記得update的動詞是用patch喔
```rb
patch "/update_wish_card_info/:num", to: "wish_list#update_wish", as: "update_wish_card_info"
```



先把剛剛複製到edit view的form_for路徑改掉
```md
> <%= form_for @wish_list, url: update_wish_card_info_path(@wish_list) do |form| %>  #改新路徑，記得要帶id
>     <div>
>       <%= form.label :title %>
>       <%= form.text_field :title %>
>     </div>
> 
>     <div>
>       <%= form.label :description %>
>       <%= form.text_area :description %>
>     </div>
> 
>   <%= form.submit %>
> <% end %>
```


:::tip
**html表單的動詞只有兩種**   
(1) 一個是get  
(2) 一個是post  
但是其他的語言有另外的動詞  
:::




#### 如果html只有兩種動詞，那為何rails這邊可以寫其他動詞呢？？  
原因就是如果今天有patch的動詞，rails會在html塞一個隱藏欄位
就是下面這個，這一段是form_for幫忙新增的
```md
> <input type="hidden" name="_method" value="patch" autocomplete="off">
```
Ps. 所以當html更新資料的時候，還是用get/post，只是rails會多塞一個動詞，假裝是用patch方法




接著實際按下更新按鈕，並在action寫下update的params，我想要看實際送出後，抓到了哪些東西
```rb
# update的controller

def update_wish
    render html: params
end
```
 
上面我們把update的params印出來，就會印出這一串，可以發現有一個_method: "patch"，這個就是前面提到的，rails會塞一串到html
```md
{"_method"=>"patch", "authenticity_token"=>"1qEcF-hbYEieLQCTndFM5mnDOxa6qZgEGG9AqHb45GvJoc7FodZqZlZtOEVkL9OKUecZXPcrS6Xv9V2NXzSn4A", "wish_list"=>{"title"=>"sdf", "description"=>"qwe"}, "commit"=>"Update Wish list", "controller"=>"wish_list", "action"=>"update_wish", "num"=>"2"}
```

確定update後，確實有抓到東西，我們就可以把資料更新進資料庫了   
Ps. 直接拿create來用就好，同樣的概念，update裡面要帶clean_params  
```rb
def update_wish
    @wish_list = WishList.find_by(id: params[:num])

    if @wish_list.update(clean_params)
        redirect_to make_a_wish_path, notice: "success update"
    else
        render :edit
    end
end
```




### 4-3、第十五步驟：設定刪除

因為rails的慣例，會直接跟show的路徑一樣，只是記得要改動詞，改成delete
```rb
delete "/wish_card_info/:num", to: "wish_list#destroy_wish", as: "destroy_wish_card_info"
```


接著到首頁把刪除的link_to加上去(記得要加上method: "delete")，才能跟show的路徑做區別
```md
> <td><%= link_to "delete", destroy_wish_card_info_path(wish_list.id), method: "delete" %></td>
```


#### data-method是啥？？？
這些data的屬性，在html是沒有效果的  
但是rails發現，假如你有data的屬性，而且又是站內連結，他會幫你用post這個動作幫你送連結  
  
但是他是怎麼弄的呢？rails會動態幫你產生表單(JS用createElement產生)  
document.createElement("form action method input type hidden")  
   
  
Ps. 這很重要，因為一般HTML能用post動作的，只有表單能用post，其他人都不能用，一般a-link只能用get送東西   
  
  
接下來到controller設定資料刪除，這樣就可以刪掉了  
```rb
def destroy_wish
    @wish_list = WishList.find_by(id: params[:num])

    @wish_list.destroy
    redirect_to make_a_wish_path, notice: "data delete"
end
```
  
  

不過這樣是不是覺得丟點危險？不小心點到就直接刪除了，我們現在新增一個警示
```md
> <td><%= link_to "delete", destroy_wish_card_info_path(wish_list.id), method: "delete", data: {confirm: "確定嗎？？？"} %></td>
```

這樣寫後，html會增加一個東西，就是data-confirm，他可以幫助你做出confirm警示
```md
> HTML

> <a data-confirm="確定嗎？？？" rel="nofollow" data-method="delete" href="/wish_card_info/4">delete</a>
```


5、優化程式碼
------

首先我們先把兩個方法寫在private的地方，原因是因為這兩個方法使用過兩次以上   
```rb
private
def clean_params
    params.require(:wish_list).permit(:title, :description)
end

def find_wish_list
    @wish_list = WishList.find_by(id: params[:num])
end
```




### 5-1、優化一：before_action

把在action重複做的事情，用before_action來寫，並在一開頭的地方，加上before_action
```rb
before_action :find_wish_list, only: [:edit, :update, :show, :destroy]
```
  

before_action另外一個用法，except  
這個意思是，除了這三個不做用，其他都要做用  
但是這個邏輯多了一圈，所以不習慣的話，可以先不用  
```rb
before_action :find_wish_list, except: [:index, :new, :create]
```


### 5-2、優化二：clean_params
  
clean_params剛剛已經寫成方法，直接塞進create、update裡面就  
```rb
# new action
@wish_list = WishList.new(clean_params)

# update action
if @wish_list.update(clean_params)
```
  
### 5-3、優化三：_form

表單重複的話，放到另外一個資料夾、檔案，這個概念叫做render partial(部分渲染)
```md
> <%= render "form" %>
```

今天想要在_form 的地方，給一個區域變數，這個區域變數是被new or edit的html地方餵食來的  
為什麼要這樣做呢？？原因就是因為，假如你今天一個頁面，有三塊廣告版面，想當然這三塊版面不可能圖片都一樣  
所以就要用到區域變數的概念，在_form設一個區域變數，值都是由render partial那邊餵食來的  
  
   
new、edit的view介面
```md
> <%= render "form", wish_list: @wish_list %>
```
  
  
上面聽不懂的話，下面舉一個更好的案例，_form先給表單，注意這邊的form_for後面接的是區域變數
```md
> _form的view

> <%= form_for ad do |form| %>          # ad 是區域變數喔
> <% end %>
```

接著實體變數在這邊設定，並把區域變數餵食給_form  
Ps. @ad1、@ad2、@ad3這些的值，是由action丟過來的，並且再送到_form  
```md
> new的view

> <%= render "ad", ad1: @ad1 %>         # 區域變數接住實體變數後，把區域變數丟給_form
> <%= render "ad", ad2: @ad2 %>
> <%= render "ad", ad3: @ad3 %>
```


6、Model.save.errors
------
假設今天new一個新實體，不過假如有一個欄位是必填的，在model.save時，會噴錯誤，實際情況到底發生什麼事呢？？  
```md
> aa = WishList.new           > new新實體
> 
> aa.errors?                  # false  >  現在還沒有錯(還沒寫進資料庫) 
>         
> aa.save                     # false  >  儲存發生錯誤(寫不進資料庫)
> aa.errors?                  # true   >  有錯了(再次詢問是否有錯誤，有錯了)
> 
> 
> aa.errors.full_massages     # name must exist  > 把錯誤印出來
```




:::tip
> **rel="nofollow"**   
> html的屬性，基本上是給爬蟲或是SEO抓的  
:::






7、非侵入式JavaScript(UJS = Unobtrusive JavaScript)
------

非侵入式JavaScript是一種將JavaScript從HTML結構抽離的設計概念，避免在HTML標籤中夾雜一堆onchange、onclick等屬性去掛載JavaScript事件，讓HTML與JavaScript分離，他的幾本原則包括

(1) 將網頁的行為層和表現層分離開    
(2) 是解決傳統JavaScript編程問題（瀏覽器呈現不一致，缺乏擴充性）的最佳實踐    
(3) 為可能不支援JavaScript進階特性的使用者代理（通常是瀏覽器）提供漸進增強的支援    



:::tip
**拉一個新專案的時候，要做兩件事**  
(1) bundle    
(2) yarn install  
:::



8、會員系統
------

接下來要做會員系統，先來訂model規格  

### 8-1、Model = User  
|欄位名稱|資料型態|補充|
|:-:|:-:|:-:|
|nickname|string|暱稱|
|email|string|信箱(必填)|
|password|string|密碼|
  
設計好規格後，可以產生model，記得要具現化表格
```shell
$ rails g model User nickname email password
$ rails db:migrate
```


16:20
流水編號問題

訂單編號問題