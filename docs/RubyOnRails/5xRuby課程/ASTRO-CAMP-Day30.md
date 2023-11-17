---
title: ROR/Ruby
sidebar_label: "DAY30. [ASTROCamp] Ruby-8"
description: Ruby
last_update:
  date: 2022-11-21
keywords:
  - 5xRuby
  - Ruby
sidebar_position: 31
---



1、軟刪除
------
:::tip
**什麼是軟刪除？？**      
假設今天要刪除許願卡，正常要刪除這個卡片，就是直接對這個卡片destroy就好了，不過通常我們在做刪除的時候  
都不會讓資料真的消失，那要怎麼辦到呢？要怎麼讓消費者覺得真的有刪掉東西，但是其實是假的，資料庫其實還是有那一筆資料   
:::


要怎麼做到呢？   
首先我們在在許願卡身上多開一個is_deleted的欄位，此欄位的預設值是false，假設今天許願卡被刪掉了，我們更改的    
是這個欄位，把此欄位改成true，之後我們撈資料，要記得撈所有的許願卡，該欄值為false的許願卡，要不然會把已經    
被刪除(is_deleted: true)的許願卡也撈出來喔！！！！！！   

```md
> def index
>   @wishlists = WishList.all     -> 撈全部資料的時候，要記得撈is_deleted: false的許願卡
> end
```

撈資料範例，這樣就可以撈出已經被刪掉的資料(假裝被刪掉)
```md
> def find_wish_list
>   @wish_list = current_user.wish_lists.find_by!(id: params[:id], is_deleted: false)
> end
```


不過我們今天換一種軟刪除的方法，在欄位上面加這個
給一個刪除的時間欄位，預設值給空，這種方式的好處是，可以知道該筆資料啥時被刪除了
```md
> t.datetime "deleted_at", default: nil
```
  
  
### 1-1、第一步驟：開欄位  
那接下來我們就來開欄位了
```shell
$ rails g migration add_deleted_at_to_wishlist
```
  
migration的內容，增加欄位並且加上index
```rb
class AddDeletedAtToWishlist < ActiveRecord::Migration[6.1]
  def change
    add_column :wish_lists, :deleted_at, :datetime
    add_index :wish_lists, :deleted_at
  end
end
```

:::tip
幫欄位加上index的好處是什麼？？  
可以幫助加速查找資訊  
:::


接下來具現化表單
```shell
$ rails db:migrate
```


為許願卡加上欄位後，要來處理刪除功能了，我們預計要做到的是，點下刪除的按鈕/連結，可以把剛剛新增的欄位，變成true的狀態


### 1-2、第二步驟：更新deleted_at
因此首先到wish_list的controller更改東西
```md
> 點下刪除的時候，去更新這個欄位(直接更新此欄位的時間 -> 意思是刪除的時候，可以知道是啥時刪除的)

> def destroy
>   @wish_list.update(deleted_at: Time.current)
> end
> redirect_to root_path, notice: "資料已刪除"
```


### 1-3、第三步驟：index過濾刪掉的東西
並且讓前台顯示的東西，我們去抓deleted表格為nil的值(為啥是這樣抓呢？因為只要這個欄位是有被寫進時間的，代表是被刪掉的)
```rb
@wish_lists = current_user.wish_lists.where(deleted_at: nil)
```


### 1-4、第四步驟：find_wish_list過濾
find方法也要改，讓show..等功能也可以過濾刪除功能
```rb
def find_wish_list
    @wish_list = current_user.wish_lists.find_by!(id: params[:id], deleted_at: nil)
end
```

  
但是我們用第三、四步驟來做篩選，會有一個問題  
假設今天專案不是你一個人做的，是很多人協作的，如果有人看不懂你寫的，不小心把沒有過濾好  
不小心把沒過濾的假刪除資料給放出去，那不就出大事了嗎！！？  
  
所以我們現在要來做一些嚴謹的設定 -> 使用scope在model來做設定  
  

#### 1-4-1、寫scope
:::tip
寫scope有兩個好處  
(1) 可以重複使用  
(2) 可以賦予意義 -> 這個很重要，設定的好，可以一眼就看出來篩選了哪些欄位  
:::


到wishlist的model來設定scope
```md  
> wishlist model

> **定義not_deleted此scope，意義是沒刪除的許願卡**
> scope :not_deleted, -> {where(deleted_at: nil)}  
```

接下來到controller改寫，只要套用not_deleted，就可以達成過濾的目的，用find_wish_list舉例
```md
> def find_wish_list
>     @wish_list = current_user.wish_lists.not_deleted.find_by!(id: params[:id])
> end
```
Ps. 在model做限制，就可以讓controller都套用  
  
  
#### 1-4-2、default_scope
但是，我們雖然寫了一個過濾方法，還是有可能會忘記用，我們來直接用default_scope，來強制使用
```md
> 用default_scope 用這種方法，所有搜尋方法都會經過他(find、find_by、where)

> default_scope -> {where(deleted_at: nil)} 
```





再優化一下wish_list destroy的功能，直接再寫一個新的destroy在model，這樣可以讓controller那邊使用
這個destroy是特別的destroy，他可以更新deleted_at(簡單說就是把目前寫得搬到model)
這樣的好處是，可以讓別人看此action，就像是一般的destroy，實際上是有規則的destroy

```md
> WishList的model

> def destroy
>   update(deleted_at: Time.current)
> end
```

controller正常寫法就可以
```md
> def destroy
>   @wish_lists.destroy
> end
```


### 1-5、paranoia
11:36 paranoia教學
插件paranoia，可以做到我們軟刪除這件事  

#### 1-5-1、首先安裝paranoia
```shell
$ bundle paranoia
```

再來直接到WishList的model的model增加這一段，這樣就可以做到軟刪除
```rb
class WishList < ApplicationRecord
  acts_as_paranoid

  # ...下面忽略
end
```

:::tip
可以不一定要取名deleted_at 可以自己創造    
:::






2、許願卡留言功能
------

留言功能製作流程
1. 先整理留言功能有哪些欄位
```md
> user_id、wish_list_id、deleted_at、content
```
2. 開Comment的model
```md
> rails g model Comment content:text user:references deleted_at:datetime:index wish_list_id:integer
```
3. 開路徑
```md
> resources :comments, shallow: true, only: [:create, :destroy]
```
4. 建表單(先決定留言要在哪留言、誰可以留言)，表單post到comment的controller
5. comment的create、destroy製作(有很多細節要注意)
6. 許願卡show的action，要補上留言的顯示、刪除功能、scope可以直接避免軟刪除的資料不會被別人抓出來



### 2-1、第一步驟：整理留言的欄位

- belongs_to user
  - user_id
- belongs_to wish_list
  - wish_list_id
- content: text  
- deleted_at:datetime, index (軟刪除、假刪除 soft delete - 前台看不到、後台看不到，但是資料庫還是有)


#### 2-1-1、留言model 

|欄位名稱|資料型態|說明|
|:-:|:-:|:-:|
|content|text|留言內容|
|wish_list_id|integer|哪一張許願卡的留言|
|user_id|integer|哪個使用者的留言|
|deleted_at|datetime|假刪除欄位|


:::tip
做每一個model得時候，只要專注自己的model就好，要不然會太雜  
:::

### 2-2、第二步驟：開留言的model
```md
> rails g model Comment content:text deleted_at:datetime:index user:belongs_to wish_list:belongs_to
```

### 2-3、第三步驟：開留言功能的路徑

:::tip
首先我們要來想一下，要把留言功能放在哪邊？  
雖然我們還沒有做可以讓多人可以看到同一個許願卡的頁面   
因此，就先做只有自己能留言的許願卡吧    
Ps. 未來要做到可以讓多人對同一個許願卡留言的功能  
:::

#### 2-3-1、在show頁面放留言的form   
接著來看頁面，想要在wishlist的show頁面放一個表單(form)，這個form可以讓大家留言(此show頁面，就是comment的index頁面)   
     
所有我們到wish_list的show.html.erb，來加上留言功能的表單     
```md
> 不過我們很快就遇到問題了！！留言表單的model、url是蝦咪  

> <%= form_with model: ??, url: ?? do |form| %>
> <% end %>
```

首先model應該很好找，就是到wish_list的action，把comment的model新創造的實體傳到html.erb這邊   
```md
> @comment = Comment.new
```

不過重點來了！！！！路徑是啥鬼
```md
> 首先我們來看一下目前show頁面的路徑

> /wish_lists/24                      # 後面24是許願卡的id(第24張許願卡)
```



照之前的經驗，我們直接到routes這樣寫，這樣就可以造出關於留言的路徑了
```md
> resources :comments
```

但這樣留言不就跟許願卡的連結脫鉤了嗎？照rails慣例，應該要改成下面這樣
```md
> resources :wish_lists do
>   resources :comments
>   end
> end
```


這樣是兩個model是有連結了，但是遇到下一個問題，就是路徑太雜亂了！！！
用留言的編輯路徑舉例
```md
> routes的路徑長這樣
> wish_lists/:wish_list_id/comments/:id/edit

> 真實連結
> wish_lists/24/comments/2/edit

> 上面連結這樣寫出來應該蠻好認的，就是第24張許願卡的第2則留言(但你看我要拆解成這樣才看得出來，證明太長了)
```


#### 2-3-2、routes：shallow介紹

:::tip
有看rails官方文件，看到shallow的應該很好奇那是什麼東西的縮寫  
現在馬上介紹一下根據前面有提到，路徑太長會很醜，並且也不好閱讀    
所以是可以把一些路徑拆開的(有些可以跟著許願卡連結，有些可以分開)  
:::


舉個例子：  
假設我今天想要把一則一則的留言show出來，就不需要許願卡的ID路徑，因為我只在意留言的ID  

反之像是new、create、index，我就會需要知道他是跟在誰的ID後面  
ex. 今天想要new一個新留言，我不知道許願卡的ID，怎麼知道這則留言是誰的  

因此可以把分類拆成下面兩段
```md
> resources :wish_lists do
>   resources :comments, only: [:new, :create, :index]
> end
> 
> resources :comments, only: [:show, :edit, :update, :destroy]
```


那shallow到底是什麼呢？其實就是上面拆成兩段的縮寫，像下面那樣寫上shallow，他可以自動幫你拆成上面兩段
```md
> resources :wish_lists do
>   resources :comments, shallow: true
> end
```


#### 2-3-3、Comment的action設定
回到原本的許願卡路徑，既然知道路徑會太長會很醜，我們先來決定留言功能有哪些action是需要用到的，決定好有哪些action後，再來寫上路徑  


|action|是否需要|說明|
|:-:|:-:|:-:|
|index|X|許願卡show = 留言index|
|show|X|除非你會需要把每則留言點開來看|
|new|X|現在是建在許願卡show的erb上|
|create|O|一定要|
|edit|X/O|看情況，看你會不會允許修改留言|
|update|X/O|同上|
|destroy|O|通常可以刪除留言|
  
  
  
使用shallow把剛剛決定好的留言的action、路徑製作好
```md
> resources :wish_lists do
>   resources :comments, shallow: true, only: [:create, :destroy]
> end
```



### 2-4、第四步驟：建立留言表單
因此可以來把留言的表單寫完了，要特別注意，因為巢狀結構的關係，create的要抓的該許願卡當下的id，下面的表單會用到的@wish_list，真正抓到的是，目前登入者的id
```md
> @wish_list = current_user.wish_lists.find(params[:id])
```

把表單model、url寫上
```md
> <%= form_with model: @comment, url: wish_list_comment_path(@wish_list) do |form| %>
>     <%= form.label :content %>
>     <%= form.text_field :content %>
> 
>     <%= form.submit %>
>     
> <% end %>
```


表單做好後，預期按鈕點下去，會送到comment的controller，因此來建立    
建好後，我們先把create表單送過來的資料，印到畫面上   
```md
> class CommentsController < ApplicationController
>   def create
>     render html: params
>   end
> end
```

#### 2-4-1、接下來在controller補上一些方法
1. 一定要登入才能留言
```md
> before_action :authenticate_user!
```
2. 先找到該張許願卡
```md
> find_wish_list -> 要使用current_user來找:wish_list_id -> (想知道為啥wish_list_id，可以用params把create印出來)
```
3. 設定留言的強參數
```md
> comment_params
```
4. 到許願卡的model設定
```md
> has_many :comments
```


create那一段意思是，從許願卡的角度，產生出一個新的留言，並且要帶過濾後的參數進去  
Ps. new新留言的時候，記得要帶wish_list_id、user_id進去，要不然會寫不進去  

```md
> class CommentsController < ApplicationController
>   def create
>     @comment = @wish_list.comments.new(comment_params)    # 這一行寫進wish_list_id
>     @comment.user = current_user                          # 這一行寫進user_id
>   
>     if @comment.save
>       redirect_to wish_list_path(@wish_list), notice: "留言成功         # 成功存進留言，要返回許願卡show頁面
>     else
>     
>       # app/views/wish_lists/show.html.erb              # 要render許願卡的show介面
>       render "wish_lists/show"
>     end
>   end
> end
```


#### 2-4-2、merge寫法介紹
不過上面那兩條id帶進進去的方式有點難看，來換一種寫法
使用merge可以直接把一個值，塞進hash裡面，並回傳新的hash，該hash是有塞進新值的

一個新hash
```md
> h = { :name=>123, :age=>18 }
```

原始把id塞進hash的方法
```md
> h[:user] = 1111

> p h               # { :name=>123, :age=>18, :user=>1111 }
```

新方法，使用merge，他可以跟另一個hash合併，並會回傳一個新hash
```md
> h.merge(cc: 123)

> p h               # { :name=>123, :age=>18, :cc=>123 }
```



因此最後的整留言個controller長這樣
```md
> class CommentsController < ApplicationController
> 
>   before_action :authenticate_user!                             # 驗證登入才能留言
>   before_action :find_wish_list only: [:create]                  
>   
>   def create
>       @comment = @wish_list.comments.new(comment_params)
>       # @comment.user = current_user
>   
>       if @comment.save
>         redirect_to wish_list_path(@wish_list), notice: "留言成功"
>       else
>         #render "wish_list/show"                                # 這一行可以修改成下面那樣
>         redirect_to wish_list_path(@wish_list),  alert: "請填寫留言"
>       end
>   
>   end
>   
>   
>   private
>   def find_wish_list
>       @wish_list = current_user.wish_lists.find(params[:wish_list_id])
>   end
>   
>   def comment_params
>       params.require(:comment).permit(:content).merge(user: current_user)      # 使用merge方法塞進去
>   end
>   
> end
```




### 2-5、第五步驟：增加限制validates

對留言的content欄位增加必填限制
```md
> validates :content, presence: true
```

### 2-6、第六步驟：把留言顯示出來

已經可以創造新留言後，該來把留言印出來了，首先我們剛剛決定把留言顯示在許願卡的show頁面，也就是新增留言表單的正下方   
那要怎麼做呢？首先當然是到許願卡controller的show，把所有留言用實體傳到view   

下面的意思是，該許願卡的id，把所有留言抓出來
```md
> def show
>   @comment = Comment.new              // 這個是設定給form model的變數
>   @comments = @wish_list.comments     // 這個是抓出所有留言的變數
> end
```


再來到show的view頁面，用迴圈印出全部東西
```md
> <ul>
>   @comments.each do |comment|
>     <li><%= comment.content %></li>
>   end
> </ul>
```



#### 2-6-1、留言顯示反轉

使用ord的功能，在留言抓出來的當下就是反轉的
```md
> def show
>   @comment = Comment.new
>   @comments = @wish_list.comments.order(id: :desc)      -> 這樣就可以讓最新的留言在最上面
> end
```


### 2-7、第七步驟：優化redirect_to

:::tip 
我們現在是用redirect_to來進行換頁，雖然有turbolinks的幫助，已經感覺不太到有換頁了，但是還是要等一下    
因此我們今天想用一個更快的方式，當使用者留完言後，我們不要用重導的方式回網頁，我們用Ajax的方式   
把新增的留言透過JS，寫進留言區的欄位，這樣不就不用redirect_to了嗎！！！！  
:::


#### 2-7-1、用create.js.erb設定

要做到這一件事，首先在controller那邊要先動一點手腳，當我們在做create的時候，通常會在後面加上重導或者是render     
但是假設今天我沒有給controller這兩樣呢？     

:::tip   
如果兩樣都沒有給的話，controller就會去view的地方，找這個檔案 -> create.html.erb    
不過今天我們小改一下檔案名稱，把html改成JS，最後名稱就變這樣 -> create.js.erb      
這樣我們就可以在這個檔案上面寫JS了     
:::

#### 2-7-2、form設定改local: false
不過這樣還不夠喔！   
還要處理表單送出那邊的設定，我們要把表單送出的資料，變成是由**XML**送出去的，因此我們要這樣改     
把表單那邊的model，加上這一段 -> local: false，記得要加在model的小括號裡面   

```md
> <%= form_with(model: @comment, url: wish_list_comments_path(@wish_list), local: false) do |f| %>      -> 加在這邊
>   <div>
>       <div>留言：</div>
>       <%= f.text_area :content %><br />
>       <%= f.submit "新增留言" %>
>   </div>
> <% end %>
```


#### 2-7-3、JS設定

接著我們可以開始寫JS了  
(1) 先抓到ul的class(先前有幫他加上class了)      
(2) 寫一個li，li包住的是留言的內容  
(3) 把這個li用insertAD塞進ul裡面  
(4) 記得要多寫一個，當使用者留言完，要把該區塊清空  

```md
> var comments = document.querySelector(".comments")
> var li = `<li><%= @comment.content %></li>`
> 
> comments.insertAdjacentHTML("afterbegin", li)
> document.querySelector("#comment_content").value = ""
```


### 2-8、第八步驟：留言功能的軟刪除

接下來把軟刪除掛上comment的model
```md
> acts_as_paranoid          # 這個寫在comment的model上
```


#### 2-8-1、加上刪除的按鈕
掛上去後，可以在show頁面加上一個刪除按鈕，刪除的路徑是comment的destroy
```md
> <% @comments.each do |comment| %>
> <li>
>   <%= comment.content %>
>   <%= link_to "刪除", comment_path(comment), method: "delete", data: {confirm: "sure????"} %>
> </li>
> <% end %> 
```

:::tip  
寫好link_to，來寫destroy的action，刪除那邊要導回去的路徑，是每一張卡片show頁面，因此後面帶的id，可以用留言、許願卡的關聯去找      
:::

因此我們先來寫關聯，每一個user都有很多留言
```md
> User model

> has_many :comments
```  

一則留言是屬於一個許願卡的
```md
> comment model

> belongs_to :wish_list
```


寫好關聯後，來寫comment的controller
```md
> before_action :find_comment, only: [:destroy]
> 
> def destroy
>   @comment.destroy
>   redirect_to wish_list_path(@comment.wish_list), notice: "留言刪除"   # 刪除後，重導回許願卡的show頁面
> end
> 
> def find_comment                                          # 先找出這則留言的使用者是誰(寫法上是用使用者來找留言)
>   @comment = current_user.comments.find(params[:id])
> end
```


#### 2-8-2、用render partial來寫留言功能

下面這一段的意思是，把@comments轉出來的所有留言，設定成comment，並把此comment傳給_comment.html.erb  
```md
> wishlist show

> <% @comments.each do |comment| %>
>   <%= render "comments/comment", comment: comment %>
> <% end %>

> Ps. 這樣寫只是換用選染的方式顯示喔！！還不會讓用JS新增的留言就有刪除功能     
```


:::tip
**rails有一個慣例**  
```md
假設今天render partial是這樣 <%= render "comments/comment", comment: comment %>  
可以簡寫成這樣 <%= render comment %>  
```
:::


#### 2-8-3、render collection(很重要)  

:::info
不過這樣寫會有另外一個問題，效能很很爛，rails有說不建議再迴圈裡面使用render partial    
Ps. 因為目前是在迴圈裡面render，我們要來改另外一個寫法 - render collection     
:::


```md
> <%= render partial: "comments/comment", collection: @comments, as: :comment %>
> 1. 前面是指我要帶進的檔案
> 2. collection是指我有一群東西(迴圈再轉的)
> 3. as是指我要帶進去的變數名稱，通常不用寫(記得要用符號或字串表示)

> 上面是完整型態，可以直接這樣寫就好
> <%= render @comments %>
```

#### 2-8-4、在create.js.erb寫render


原本寫法var li的寫法是寫一段留言的內容，並塞進留言欄位，之後我們來用render寫法試試
```md
> create.js.erb

> var comments = document.querySelector(".comments")
> 
> var li = `<li><%= @comment.content %></li>`
> 
> comments.insertAdjacentHTML("afterbegin", li)
> document.querySelector("#comment_content").value = ""
```




改寫這段(這是原始欄位)
```md
> var li = `<li><%= @comment.content %></li>`
```

改成這個 - 記得要用`才能執行，這個寫法的意思是
```md
> create.js.erb

> var li = `<%= render "comments/comment", comment: @comment %>`
```

或者是用一樣用雙引號，但是加上J，這個J是跳脫符號，可以幫你加上反斜線
```md
> var li = "<%= j render "comments/comment", comment: @comment %>"
```

上面render partial的檔案內容是這個   
共同的東西都寫在_comment.html.erb這個資料夾，並且把他那一部分渲染過來，渲染過來的時後，裡面的comment變數是@comment，這樣寫的好處是，不管li寫的在複雜，我都可以把li透過render的方式渲染出來   
```md
> _comment.html.erb

> <li>
>   <%= comment.content %>
> </li>
```


#### 2-8-5、ajax處理刪除

:::tip
**前情提要**      
destroy等等會用到@comment，先把它寫出來怕忘記  
:::

```md
> def find_comment
>   @comment = current_user.comments.find(params[:id])
> end
```

#### 2-8-6、link_to掛上remote: true
前面我們是用普通的rails刪除，今天我也想把刪除功能，跟剛剛create一樣，是用xhr刪除，那要怎麼做呢？     
首先到刪除的連結那邊，這時候可以幫link_to掛上使用XML送的方式(remote: true)       
```md
> <% @comments.each do |comment| %>
> <li>
>   <%= comment.content %>
>   <%= link_to "刪除", comment_path(comment), method: "delete", data: {confirm: "sure????", remote: true} %>
> </li>
> <% end %> 
```


接著一樣使用把原本的html.erb，改成destroy.js.erb，並在這邊寫刪除li的JS，這樣就可以做到用XHR刪除留言了    
Ps. dom_id等等會提到     
```md
> var li = document.querySelector("#<%= dom_id(@comment) %>")
> li.remove()
```


#### 2-8-8、_comment檔案增加刪除
不過還記得我們剛剛新建的_comment資料夾嗎？我們把刪除的連結加進這個資料夾     
這樣就可以在新增留言的時候，刪除的按鈕就一起出來了，原因就是這一包檔案裡面的資料在你新增留言時，都會一起渲染出來    

```md
> _comment.html.erb

> <li>
>   <%= comment.content %>
>   <%= link_to "刪除", comment_path(comment), method: "delete", data: {confirm: "sure????"} %>
> </li>
```


#### 2-8-9、`dom_id` vs `comment_<%= comment.id %>`

:::tip
在介紹dom_id前，先介紹不用dom_id的話，我們要如何為要被刪除的留言，附上各個留言的編號      
:::

首先我們先假設，我們每一個刪除的id，他的id是"comment_??"，??是有規律的數字
到時候去抓留言的id，只要抓到此id就可以刪掉了

```md
> _comment.html.erb檔案(這邊的區域變數，是從別的地方送過來的，要記得)

> <li id="comment_<%= comment.id %>">           # 預期是"id = comment_2 之類的"
>     <%= comment.content %>
>     <%= link_to "刪除", comment_path(comment), method: "delete", data: {confirm: "sure????", remote: true} %>
> </li>
```


接下來可以到destroy.js.erb，抓住id並刪掉，用@comment.id，來抓到指定要刪除了留言
```md
> document.querySelector("#<%= @comment.id %>").remove()
```


#### 2-8-10、dom_id
不過這邊有個特別的寫法，叫做dom_id，這個helper會幫我們長出一段id出來，可以實際把它印出來看看，他會自動幫你長出id     


```md
> _comment.html.erb檔案(這邊的區域變數，是從別的地方送過來的，要記得)

> <%= dom.id(comment) %>        #這邊可以寫區域變數的原因是因為別的地方送變數過來喔

> comment_54
> comment_53
> comment_52
> comment_51 ...... 等等
```

這樣我們就可以使用dom_id來寫了，用這個寫就可以不用自己串字串 -> ex. comment_id  
```md
> _comment.html.erb

> <li id="<%= dom_id(comment) %>">
>     <%= comment.content %>
>     <%= link_to "刪除", comment_path(comment), method: "delete", data: {confirm: "sure????", remote: true} %>
> </li>
```

destroy也適用dom_id改寫
```md
> destroy.js.erb

> document.querySelector("#<%= dom_id(@comment)%>").remove()
```




3、stimulus JS
------

首先在show頁面，加上一個按鈕
```md
> <button id="btn">
>   NO
> </button>
```

並且在同一個頁面最下面寫script，簡單的JS，點按鈕後，印出hi
```md
> var btn = document.querySelector("#btn")
> btn.addEventListener("click", (e) =>{
>    e.preventDefault()
>    console.log("hi") 
> 
> })
```


不過JS放在同一個資料夾非常的醜，要放到哪呢？？rails有一個放JS的地方  
JavaScript檔案夾裡的packs -> application.js，我們就把檔案搬過去  
  
:::info
不過搬過去後，發生了另外一個問題 -> script先後渲染問題      
:::
  

所以我們要加一段DOMContentLoaded，這樣可以讓事件發生在網頁載入後  
```md
> document.addEventListener("DOMContentLoaded", () =>{
>     var btn = document.querySelector("#loveBtn")
> 
>     if (btn) {
>         btn.addEventListener("click", (e) => {
>             console.log("hi");
>         })     
>     }
> 
> })
```
  
  
#### 記得看[turbolinks的深度解析](https://www.writershelf.com/article/rails-turbolinks%E2%84%A2-5-%E6%B7%B1%E5%BA%A6%E7%A0%94%E7%A9%B6?locale=zh-TW)
:::tip
要在rails寫JS，遇到兩個大問題  
(1) JS寫哪裡   
(2) 怎麼面對Turbolinks的生命週期   
**Stimulus.js可以解決上面兩件事，這個是一個微框架**   
:::

  

這一段是Stimulus.js官網的CODE，來解析一下，這樣寫，之後整段的生命週期都交由hello_controller管理
```md
> <div data-controller="hello">                     # 在這個div掛controller，之後要創hello_controller的檔案
>   <input data-hello-target="name" type="text">    
> 
>   <button data-action="click->hello#greet">       # 當發生click事件，請你去hello_controller找greet的action
>     Greet
>   </button>
> 
>   <span data-hello-target="output">               # data-controller名字-target="span名字"
>   </span>                                         # 上面這行取代的是querySelector("#id")
> </div>
```

另一段
```md
> // hello_controller.js                            # hello_controller資料夾
> import { Controller } from "stimulus"
> 
> export default class extends Controller {
>   static targets = [ "name", "output" ]           # 上面傳下來的target要寫在這
> 
>   greet() {
>     this.outputTarget.textContent =               # outputTarget的由來是，只要你有寫target，他會自動幫你生出一個"名字Target"
>       `Hello, ${this.nameTarget.value}!`
>   }
> }
```



### 3-1、實際練習stimulus

首先要引入stimulus的套件，可以這樣打，rails會印出目前內建的一堆外掛
```md
> rails
```
接著會看到stimulus指令，只要在最前面加上rails就可以成功啟用
```md
>  rails webpacker:install:stimulus
```

:::tip
跑完之後會在JS那個資料夾下面多一個controller的資料夾，之後stimulus都寫在這個資料夾裡面     
:::
  

下面的connect比較特別的生命週期，他是一個固定寫法，當這個controller掛載到div身上後，就會做connect下面的事情
```md
> import { Controller } from "stimulus"
> 
> export default class extends Controller {
>   static targets = [ "output" ]
> 
>   connect() {
>     console.log("hello");                     # 掛載controller後，就會自動觸發
>   }
> }
```

如何掛載controller？？隨便寫一個地方寫上data-controller就可以
```md
> <section data-controller="hello"></section>
```



#### 3-1-1、webpacker解決打包過慢的問題  

這一段執行會有一段時間是正常的，執行的時候會做一堆事情   
因此要來請一個新工具，來幫我們打包，會快很多  
有一個webpacker的gem，我們要用裡面的webpack工具   


使用此指令，再開一個伺服器，可以幫助你打包東西，這樣會快超級多
```md
> bin/webpack-dev-server
```


#### 3-1-2、foreman一次執行兩個server
foreman這個套件可以一次執行rails s、bin/webpack-dev-server  
  

先安裝foreman後
```md
> bundle add foreman -> 這樣可以裝這個插件，他可以幫助我們一次執行上面兩個東西
```

到rails建立一個資料夾 - Procfile，裡面放的code為
```md
> web: bin/rails server -p 3000       # 執行rails s
> webpack: bin/webpack-dev-server     # 執行webpack
``

最後再執行，這樣就可以一次執行兩個server了
```md
> foreman start/s
```



### 3-1、按鈕加上data-controller
把data-controller加在按鈕上，並新建一個資料夾叫做love_btn_controller.js，這邊加的controller，可以幫助把資料傳到JS那邊，並開始寫JS
```md
> <button style= "padding: 10px 20px", data-controller="love-btn">
>     no
> </button>
```



### 3-2、按鈕加上data-action
我們來在按鈕那邊多增加幾個屬性 - data-action => 被點擊後，此按鈕會觸發gogo的fc

:::tip
**data-action有幾點要注意**    
(1) click是指對這個按鈕做的事情(就是addEventListener的click行為)      
(2) 後面的love-btn就是對到剛剛的controller    
(3) 並且最後面會有一個toggle的action，這個用處就是，當你點擊此按鈕，會發生的事情寫在這邊      
:::

```md
> <button style= "padding: 10px 20px", data-controller="love-btn"
>                                      data-action="click->love-btn#toggle">
>
>     <span>no</span>
> </button>
```

```md
> toggle() {
>   console.log("good");
> }
```


### 3-3、目標加上target
接下來幫按鈕裡面的文字增加target，想要等等點擊下去，就可以更改no
這個target很好用，在這邊設定，就可以直接在JS那邊改他的內容
```md
> <button style= "padding: 10px 20px", data-controller="love-btn"
>                                      data-action="click->love-btn#toggle">
>     <span data-love-btn-target="text">no</span>                               # 幫文字增加target
> </button>
```



### 3-4、簡單改變文字
加上target後，我們來簡易的改變一下按鈕中的文字(還沒有用到狀態喔)
```md
> import { Controller } from "stimulus"
> 
> export default class extends Controller {
>   static targets = [ "icon" ]
> 
>   connect() {
>     // console.log("right now");
>   }
> 
>   change() {> 
>     this.iconTarget.textContent = "Yes"           # 直接改變target的文字
>   }
> }
```


### 3-5、JS存放狀態的變數

在寫stimulus的時候，可以用this來存放物件的狀態
PS. 在ruby可以用實體變數來存放狀態

```md
> import { Controller } from "stimulus"
> 
> export default class extends Controller {
>   static targets = [ "icon" ]
> 
>   initialize() {
>     this.btnState = false                   -> 用this來存放狀態，那個btnState可以隨便亂寫(重點是前面的this)
>   }
> 
>   connect() {
>     // console.log("right now");
>   }
> 
>   change() {
>     // 如果按鈕狀態是true，那就讓他等等狀態變false，這時候按鈕的文字要變no
>     if(this.btnState) {
>         this.iconTarget.textContent = "no"    
>     } else {
>         this.iconTarget.textContent = "yes"    
>     }
>     this.btnState = !this.btnState          -> 切換狀態
>   }  
> }
```



### 3-6、connect VS initialize
:::tip
controller掛上去，會觸發connect    
connect是指當他掛載到某一個DOM元素身上的時候，會發動事情，connect會做很多次    
   
stimulus 掛上去的時候，就會觸發initialize
initialize是指他剛被new出來，就發生的事情，而且這個只會做一次，這裡很適合放狀態  
**懶人包：initialize比connect的發生順序還要早**  
:::


### 3-7、form掛controller

```md
> <%= form_with model: @user, url: users_path, class: "form", data: {controller: "reservation-form"} do |form| %>
> <% end %>
```

### 3-8、form-field掛action、target、掛兩個action
```md
> <%= form.text_field :name, class: 'input-border-click', data: {reservation_form_target: "nameInput",
>                                                                action: "input->reservation-form#input blur->reservation-form#nameFieldBlur"} %>
```

