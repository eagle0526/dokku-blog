---
title: ROR/Rails 巢狀留言功能
sidebar_label: "15. [ROR] Rails 巢狀留言功能"
description: Rails 巢狀留言功能
last_update:
  date: 2023-11-07
keywords:
  - ROR
  - 巢狀留言功能
sidebar_position: 15
---


:::tip
**前情提要：**         
本篇將教學巢狀留言怎麼設定
:::



### 最終流程

(1) 先在house中的 show，把留言全部抓出來，@house是一定有的
```rb
def show
  @commetns = @house.comments
end
```

這樣可以把這間房子的全部留言抓出來

(2) 到show.html.erb，把留言渲染出來
```html
<div class="flex w-full p-10 mb-8 border-gray-100 border-y">
     <% @comments.each do |comment| %>
           <ul>
                <li>留言ID:<%= comment.id %></li>
                <li>內容:<%= comment.content %></li>
            </ul>
     <% end %>
</div>

```


(3)  加上刪除按鈕
```html
 <div class="flex w-full p-10 mb-8 border-gray-100 border-y">
     <% @comments.each do |comment| %>
           <ul>
                <li>留言ID:<%= comment.id %></li>
                <li>內容:<%= comment.content %></li>
                <li>刪除:<%= link_to ‘刪除’, comment_path(comment), data: {turbo_method: “delete”, confirm: “sure?”} %></li>
            </ul>
     <% end %>
</div>
```



(4) 不過這樣把留言都寫在 `house` 的 `show` 有點亂，所以這邊改一下，我們先新增一個erb，也就是新增這個檔案 `views/comments/_comment`

我們等等把新增的留言都寫在這個檔案，並且在house 的show頁面用render 的方式，把留言渲染出來

因此這樣修改
```html
<!-- House/show.html.erb -->
 <div class="flex w-full p-10 mb-8 border-gray-100 border-y">
     <% @comments.each do |comment| %>
	<%= render “comments/comment”, comment: comment %>
     <% end %>
</div>

```

新增那一行的意思是，我要渲染comments/comment 這個頁面的檔案，並且把comment這個參數傳過去，當作comment區域變數

此時重新整理頁面，並且看一下LOG資訊，會發相印出這幾行

```shell
數據
15:40:46 web.1  |   ↳ app/views/comments/_comment.html.erb:20
15:40:46 web.1  |   Rendered collection of templates [0 times] (Duration: 0.0ms | Allocations: 13)
15:40:46 web.1  |   Rendered comments/_comment.html.erb (Duration: 3.6ms | Allocations: 3014)
15:40:46 web.1  |   Rendered houses/show.html.erb within layouts/application (Duration: 97.4ms | Allocations: 29815)
```

(5) render partial

不過上面那樣寫會遇到另外一個問題，因為rails不建議再迴圈中，渲染畫面(會造成效能緩慢)，所以我們用另外一個寫法 `Render collection`

改成這樣，下面這個意思是，前面一樣渲染指定的檔案，collection意思是，原本迴圈在跑的那一包，as意思是要帶進去的變數名稱

```html
<div class="flex w-full p-10 mb-8 border-gray-100 border-y">
  <%= render partial: "comments/comment", collection: @comments, as: :comment %>
</div>
```


上面是完整型態，通常這樣寫就可以了
```html
<div class="flex w-full p-10 mb-8 border-gray-100 border-y">
  <%= render @comments %>
</div>
```

此時再看向LOG，會發相時間渲染時間確實變短了
```shell
數據
15:40:03 web.1  |   ↳ app/views/comments/_comment.html.erb:20
15:40:03 web.1  |   Rendered collection of templates [0 times] (Duration: 0.0ms | Allocations: 13)
15:40:03 web.1  |   Rendered collection of comments/_comment.html.erb [3 times] (Duration: 38.9ms | Allocations: 15555)
```


(5) 接著我們來處理留言顯示的部分，也就是 `comments/comment` 這個檔案

我們先把留言渲染出來，這邊的 `comment` 就是剛剛 `render @comments` 這個傳過來的

```html
<article>
  <p><%= comment.content %></p>
  <small class="mr-4">by <%= comment.user.name %></small>
  <small class="mr-4">回覆</small>
  <small class="mr-4">刪除留言</small>
  
</article>
```





(6) 我們在每個留言下方，增加一個回覆留言表單，一開始先把他隱藏，我們之後來抓每則留言的dom_id ，這樣當我點擊該則留言的回覆時，就可以打開該則留言

```html
<article>
  <p><%= comment.content %></p>
  <small class="mr-4">by <%= comment.user.name %></small>
  <small class="mr-4">回覆</small>
  <small class="mr-4">刪除留言</small>

  <div class="hidden">
  <%# 多渲染一個留言表單，這個表單會傳一些數值到留言表單那邊，把 comment.house當作house傳過去，comment 當作 parent 傳過去 %>
  <%# <div class=""> %>
    <%= render partial: 'comments/form', locals: { house: comment.house, parent: comment }%>          
  </div>
  
</article>
```


  

(7) 把每則留言的子留言渲染在下方

```html
<article>
  <p><%= comment.content %></p>
  <small class="mr-4">by <%= comment.user.name %></small>
  <small class="mr-4">回覆</small>
  <small class="mr-4">刪除留言</small>

  <div class="hidden">
  <%# 多渲染一個留言表單，這個表單會傳一些數值到留言表單那邊，把 comment.house當作house傳過去，comment 當作 parent 傳過去 %>
  <%# <div class=""> %>
    <%= render partial: 'comments/form', locals: { house: comment.house, parent: comment }%>          
  </div>
  
  <hr>

 <%# 這裡的意思是，把該留言的子留言全部選染出來，大家還記得這個渲染方式的原型長什麼樣嗎 %>
  <div class="m-10">
    <%= render comment.comments %>
  </div>
  
</article>
```



渲染子留言詳解，原型差不多就是下面那樣，這是最一開始的樣子，我們用 `render partial` 的方式寫

```html
  <div class="m-10">
    <% comment.comments.each do |sub_comment| %>
      123:<%= sub_comment.content %>
    <% end %>
    <%#= render comment.comments %>
  </div>
```


```md
Render
  <div class="m-10">
    <% comment.comments.each do |sub_comment| %>
        <%= render "comments/comment, comment: comment" %>
  </div>


Render collection
不過要記得，這樣會效能很差，再改成這樣
  <div class="m-10">
    <% comment.comments.each do |comment.comments| %>
        <%= render partial “comments/comment”,  collection: comment.comments, as: comment.comments%>
  </div>

最後我們可以這樣寫
<%= render  comment.comments %>
```

這樣我們可以持續的把子留言印出來，而且都會帶著 comments/comment 這個頁面有寫的所有資訊！


(8) 用create.js.erb 來寫新增留言 - 先暫時不要用

(9) 刪除留言

先把軟刪除加上去
```shell
$ Rials g migration add_deleted_at_to_comment
```


把 migration 檔案加上去
```rb
class AddDeleteAtToComment < ActiveRecord::Migration[7.0]
  def change
    add_column :comments, :deleted_at, :datetime
    add_index :comments, :deleted_at
  end
end
```



把這一段加進comment的model
```rb
acts_as_paranoid
```

house.controller加上這個action
```rb

  def destroy
    # render html: params
    @comment.destroy
    redirect_to house_path(@comment.house_id), alert: "成功刪除留言"
  end
```



在_comment.html.erb這個留言頁面，把刪除按鈕加上去，這邊我有補上兩個條件，如果今天是自己寫的留言、該留言沒有子留言的話，才能刪除留言
```html
  <% if comment.user_id == current_user.id && comment.comments.size == 0 %>
    <small class="mr-4">    
      <%= link_to '刪除留言', comment_path(comment), data: {turbo_method: "delete", confirm: "確定嗎？"}, class: "text-red-500" %>
    </small>
  <% end %>
```


(10) 把回覆留言一開始先隱藏，後來再展開
記得先創建一個新的stimulus，把它掛在整個留言的最上層

```html
<article data-controller="comment-reply">
<article>

這裡我們一開始先把form隱藏
  <div class="hidden" data-comment-reply-target="form">
    <%= render partial: 'comments/form', locals: { house: comment.house, parent: comment } %>          
  </div>

```


comment_reply_controller.js，用js來控制狀態轉變

```js
export default class extends Controller {
  static targets = [ 'form' ]

  connect() {
    this.replyState = false
  }

  reply() {
    if (this.replyState){
      this.formTarget.classList.add("hidden")
    } else {
      this.formTarget.classList.remove("hidden")
    }
    this.replyState = !this.replyState
  }
}

```










巢狀留言軟刪除
------

如果今天已經幫留言設定好了軟刪除，但是想要把被刪除的留言顯示出來，並且顯示的是特定文字(留言已被使用者刪除)，這樣要怎麼做呢？



看過官方文件後，只要這樣輸入，就可以把不管有沒有被刪除過的東西都抓出來

```m
> YourModel.readonly.with_deleted
> 這樣可以把指定model的物件全部抓出來，不管有沒有被軟刪除過
```


讓我們來看看實際樣子：
```rb
# controllers/houses_controller.rb

class HousesController < ApplicationController
  before_action :find_house, only: [:show, :like, :dislike]
  

  def show
    @comment = @house.comments.new

    # 注意這段
    @comments = @house.comments.readonly.with_deleted.where("parent_id IS NULL OR parent_id = 0")
  end

  private
  def find_house
    @house = House.find(params[:id])
  end
end
```

可以看到 @comments 這邊，他把留言全部抓出來後，再補上後面那一段where，那一段的意思是，找到 `root_houses`，因為我們做的是巢狀留言，所以這邊先把沒有子留言的抓出來

接著我們把留言顯示出來
```html
<!-- views/houses/show.html.erb -->

<section class="mt-36 mx-36">
  <div class="mb-6 bg-white rounded-lg shadow">
    
    <%# 渲染房子上方區塊 %>
    <%= render "card", house: @house %>
  
    <div class="w-full p-4 text-gray-600 focus-within:text-gray-400">            
      <%# 下面這邊是渲染留言表單出來，順便會把兩個實體變數傳過去，一個是房子實體變數傳過去變區域變數，一個是把nil當作parent區域變數傳過去 <%>
      <%= render partial: 'comments/form', locals: { house: @house, parent: nil, url: comments_path(@house) } %>                
    </div>

    <div class="w-full p-10 mb-8 border-gray-100 border-y">
      <%= render @comments %>                            
    </div>
    <br>
     
  </div>

</section>
```

可以看到這一行：
```html
<div class="w-full p-10 mb-8 border-gray-100 border-y">
  <%= render @comments %>                            
</div>
```


這一段可以渲染出這個畫面：
```html
<!-- views/comments/_comments -->
<div class="my-2 text-sm text-gray-600">
  <span><%= comment.deleted_at ? "這則留言被該用戶刪除" : simple_format(comment.content) %></span>
</div>
```

這邊可以看到，我們先判斷留言的deleted_at是否有資料，如果今天有資料，就代表這則留言已經被刪除，如果沒有，就代表該留言還存在，因此我們寫一個判斷式，就是上面呈現的那樣



### 子留言的軟刪除

我們想要把子留言留在父留言的下方，因此我們這樣做：

```html
<!-- views/comments/_comments -->
<div class="my-2 text-sm text-gray-600">
  <span><%= comment.deleted_at ? "這則留言被該用戶刪除" : simple_format(comment.content) %></span>
</div>

<div class="mt-10">
  <%= render comment.comments.readonly.with_deleted %>
</div>
```

這樣我們就可以把父留言、子留言已經被刪除的留言都顯示出來




### 實際例子

我們實際把東西印出來看看，我們先把正常的樣子印出來，一間房屋會有很多留言

```md
> h = House.first
> h.comments
> 
> ------
> [#<Comment:0x00000001137940a0                                        
>   id: 71,                                                            
>   content: "fdsfdsf",                                                
>   user_id: 1,                                                        
>   house_id: 9,                                                       
>   created_at: Tue, 02 May 2023 15:29:16.264570000 CST +08:00,        
>   updated_at: Tue, 02 May 2023 15:29:16.264570000 CST +08:00,        
>   parent_id: 68,                                                     
>   deleted_at: nil>,                                                  
>  #<Comment:0x000000011378ff00                                        
>   id: 66,                                                            
>   content: "讚讚好棒",                                               
>   user_id: 2,                                                        
>   house_id: 9,
>   created_at: Thu, 27 Apr 2023 17:34:24.227720000 CST +08:00,
>   updated_at: Thu, 27 Apr 2023 17:34:24.227720000 CST +08:00,
>   parent_id: nil,
>   deleted_at: nil>,
>  #<Comment:0x000000011378fd98
>   id: 67,
>   content: "我好爛",
>   user_id: 2,
>   house_id: 9,
>   created_at: Thu, 27 Apr 2023 17:34:29.893655000 CST +08:00,
>   updated_at: Thu, 27 Apr 2023 17:34:29.893655000 CST +08:00,
>   parent_id: 66,
>   deleted_at: nil>,
>  #<Comment:0x000000011378fc30
>   id: 68,
>   content: "我\n好\n棒",
>   user_id: 2,
>   house_id: 9,
>   created_at: Fri, 28 Apr 2023 16:47:42.343017000 CST +08:00,
>   updated_at: Fri, 28 Apr 2023 16:47:42.343017000 CST +08:00,
>   parent_id: nil,
>   deleted_at: nil>] 
```



不過上面那樣輸入指令，只會把沒被軟刪除的留言列出來，我們來看看用 `readonly.with_deleted` 這個指令：

```md

> h = House.first
> h.comments.readonly.with_deleted

------
> [#<Comment:0x0000000114d670f8                                                                             
>   id: 71,                                                                                                 
>   content: "fdsfdsf",                                                                                     
>   user_id: 1,                                                                                             
>   house_id: 9,                                                                                            
>   created_at: Tue, 02 May 2023 15:29:16.264570000 CST +08:00,                                             
>   updated_at: Tue, 02 May 2023 15:29:16.264570000 CST +08:00,                                             
>   parent_id: 68,                                                                                          
>   deleted_at: nil>,                                                                                       
>  #<Comment:0x0000000114d66f90                                                                             
>   id: 65,                                                                                                 
>   content: "讚讚讚",                                                                                      
>   user_id: 2,                                                                                             
>   house_id: 9,
>   created_at: Thu, 27 Apr 2023 15:01:26.975122000 CST +08:00,
>   updated_at: Thu, 27 Apr 2023 15:03:00.691882000 CST +08:00,
>   parent_id: 64,
>   deleted_at: Thu, 27 Apr 2023 15:03:00.691850000 CST +08:00>,
>  #<Comment:0x0000000114d66e28
>   id: 64,
>   content: "哇哇哇",
>   user_id: 2,
>   house_id: 9,
>   created_at: Thu, 27 Apr 2023 15:01:21.818236000 CST +08:00,
>   updated_at: Thu, 27 Apr 2023 15:03:03.701301000 CST +08:00,
>   parent_id: nil,
>   deleted_at: Thu, 27 Apr 2023 15:03:03.701293000 CST +08:00>,
>  #<Comment:0x0000000114d66cc0
>   id: 66,
>   content: "讚讚好棒",
>   user_id: 2,
>   house_id: 9,
>   created_at: Thu, 27 Apr 2023 17:34:24.227720000 CST +08:00,
>   updated_at: Thu, 27 Apr 2023 17:34:24.227720000 CST +08:00,
>   parent_id: nil,
>   deleted_at: nil>,
>  #<Comment:0x0000000114d66b58
>   id: 67,
>   content: "我好爛",
>   user_id: 2,
>   house_id: 9,
>   created_at: Thu, 27 Apr 2023 17:34:29.893655000 CST +08:00,
>   updated_at: Thu, 27 Apr 2023 17:34:29.893655000 CST +08:00,
>   parent_id: 66,
>   deleted_at: nil>,
>  #<Comment:0x0000000114d669f0
>   id: 68,
>   content: "我\n好\n棒",
>   user_id: 2,
>   house_id: 9,
>   created_at: Fri, 28 Apr 2023 16:47:42.343017000 CST +08:00,
>   updated_at: Fri, 28 Apr 2023 16:47:42.343017000 CST +08:00,
>   parent_id: nil,
>   deleted_at: nil>,
>  #<Comment:0x0000000114d66888
>   id: 70,
>   content: "你好棒",
>   user_id: 1,
>   house_id: 9,
>   created_at: Tue, 02 May 2023 15:27:43.352346000 CST +08:00,
>   updated_at: Tue, 02 May 2023 15:28:23.829312000 CST +08:00,
>   parent_id: 68,
>   deleted_at: Tue, 02 May 2023 15:28:23.829200000 CST +08:00>]
```



上面輸入後，可以看到我們把所有留言都抓到了， `deleted_at` 有數據的留言，就是代表該留言在該時間點被刪掉。














