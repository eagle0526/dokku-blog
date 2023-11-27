---
title: ROR/Rails render collection 介紹
sidebar_label: "17. [ROR] Rails render collection 介紹"
description: Rails render collection 介紹
last_update:
  date: 2023-11-07
keywords:
  - ROR
sidebar_position: 17
---



:::tip 
**前情提要：**         
在迴圈內渲染其他頁面時候，rails有推薦使用collection這個方法，使用此方法就可以讓效能變更好，因此我們來看詳細的數據，確認使否有讓效能變好
:::



基本設定
------

### controller

以下是我們的controller設定，現在只有專注在 `show action` 就好，其他都不必在意
```rb
class PostsController < ApplicationController
  before_action :authenticate_user!
  before_action :find_post, only: [:show, :edit, :update, :destroy]
    
  # ...省略

  def show      
    @comment = Comment.new

    # 下面這一行是我們要用到的
    @comments = @post.comments.where(parent_id: nil).order(id: :desc)
  end
  
  private
  def find_post
    @post = Post.find_by(id: params[:id])        
  end
end
```

PS. 順便說一下為啥後面要有where那一段，因為我們有做子留言系統，一個留言今天如果有人要對此留言做回覆，該留言就會產生一個parent_id，所以如果今天留言沒有parent_id，就代表該留言是父留言


### view

下面這樣留言方式，就可以渲染出這個頁面 `comments/comment` 的東西，因此我們要再去該頁面做設定
```html
<!-- views/posts/show.html.erb -->

<h4>我是大家的留言</h4>

<ul class="ul">
    
  <% @comments.each do |comment| %>
    <%= render "comments/comment", comment: comment %>
  <% end %>
  
</ul>
```

照下面的設定，我們想要把留言的內容寫出來，這樣寫後，就可以把所有留言都顯示出來了
```html
<!-- views/comments/_comment.html.erb -->

<p><%= comment.content %></p>
```

### render collection 改寫


但是當我們在迴圈裡面，渲染另外一個頁面的時候，效能不太好，因此rails也建議使用一個方法，也就是下面這樣：
```html
<!-- views/posts/show.html.erb -->

<h4>我是大家的留言</h4>

<ul class="ul">
  
  <!-- 這個是原始版本，partial 就是我們想要渲染的頁面，collection就是我們想要跑的迴圈，as是我們想要傳進去的參數 -->
  <%#= render partial: "comments/comment", collection: @comments, as: :comment %>

  <!-- 如果今天格式都像上面那樣符合格式，我們可以直接像這樣縮寫 -->
  <%= render @comments %>
  
</ul>
```

不過這個效能真的有比較好嗎？我們來把所有終端機的數據全部都列出來看



效能比較
-----


一篇文章會有很多的留言，我先把所有留言列出來
```shell
# console

p = Post.first

p.comments
Comment Load (0.4ms)  SELECT "comments".* FROM "comments" WHERE "comments"."deleted_at" IS NULL AND "comments"."post_id" = ?  [["post_id", 12]]
------  
 =>                        
[#<Comment:0x00000001096ff158
  id: 76,
  content: "first comment",
  user_id: 6,
  post_id: 12,
  deleted_at: nil,
  created_at: Mon, 08 May 2023 17:51:17.167900000 UTC +00:00,
  updated_at: Mon, 08 May 2023 17:51:17.167900000 UTC +00:00,
  parent_id: nil>,
 #<Comment:0x0000000108e88b00
  id: 77,
  content: "second comment",
  user_id: 6,
  post_id: 12,
  deleted_at: nil,
  created_at: Mon, 08 May 2023 17:52:19.571892000 UTC +00:00,
  updated_at: Mon, 08 May 2023 17:52:19.571892000 UTC +00:00,
  parent_id: nil>,
 #<Comment:0x0000000108e88998
  id: 78,
  content: "third comment",
  user_id: 6,
  post_id: 12,
  deleted_at: nil,
  created_at: Mon, 08 May 2023 18:01:41.998257000 UTC +00:00,
  updated_at: Mon, 08 May 2023 18:01:41.998257000 UTC +00:00,
  parent_id: nil>] 
```

可以看到目前這一篇文章有三則留言，接下來我們分別用有使用collection和沒有使用的效能列出來

### 沒用collection的寫法


先寫出HTML
```html
<h4>我是大家的留言</h4>

<ul class="ul">
    
  <% @comments.each do |comment| %>
    <%= render "comments/comment", comment: comment %>
  <% end %>
  
</ul>
```



完整數據，但是這邊數據太雜了，我把重要的部分欻出來
```md
>   Rendering posts/show.html.erb within layouts/application
>   Comment Load (0.4ms)  SELECT "comments".* FROM "comments" WHERE "comments"."deleted_at" IS NULL AND "comments"."post_id" = ?  [["post_id", 12]]
>   ↳ app/views/posts/show.html.erb:61
>   Comment Load (0.2ms)  SELECT "comments".* FROM "comments" WHERE "comments"."deleted_at" IS NULL AND "comments"."parent_id" = ?  [["parent_id", 76]]
>   ↳ app/views/comments/_comment.html.erb:17
>   Rendered comments/_comment.html.erb (Duration: 3.2ms | Allocations: 1177)
>   Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."deleted_at" IS NULL AND "comments"."parent_id" = ?  [["parent_id", 77]]
>   ↳ app/views/comments/_comment.html.erb:17
>   Rendered comments/_comment.html.erb (Duration: 3.1ms | Allocations: 1139)
>   Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."deleted_at" IS NULL AND "comments"."parent_id" = ?  [["parent_id", 78]]
>   ↳ app/views/comments/_comment.html.erb:17
>   Rendered comments/_comment.html.erb (Duration: 1.9ms | Allocations: 1139)
>   Rendered posts/show.html.erb within layouts/application (Duration: 16.5ms | Allocations: 5253)
> [Webpacker] Everything's up-to-date. Nothing to do
>   Rendered shared/_navbar.html.erb (Duration: 0.6ms | Allocations: 296)
>   Rendered shared/_flash.html.erb (Duration: 0.1ms | Allocations: 57)
>   Rendered layout layouts/application.html.erb (Duration: 50.0ms | Allocations: 12362)
> Completed 200 OK in 70ms (Views: 50.6ms | ActiveRecord: 4.4ms | Allocations: 15167)
```

重點擷取：該篇文章目前有三則留言，因此這邊有印出三次留言的數據
```md
>   Comment Load (0.4ms)  SELECT "comments".* FROM "comments" WHERE "comments"."deleted_at" IS NULL AND "comments"."post_id" = ?  [["post_id", 12]]
>   ↳ app/views/posts/show.html.erb:61
>   Comment Load (0.2ms)  SELECT "comments".* FROM "comments" WHERE "comments"."deleted_at" IS NULL AND "comments"."parent_id" = ?  [["parent_id", 76]]
>   ↳ app/views/comments/_comment.html.erb:17
>   Rendered comments/_comment.html.erb (Duration: 3.2ms | Allocations: 1177)
>   Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."deleted_at" IS NULL AND "comments"."parent_id" = ?  [["parent_id", 77]]
>   ↳ app/views/comments/_comment.html.erb:17
>   Rendered comments/_comment.html.erb (Duration: 3.1ms | Allocations: 1139)
>   Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."deleted_at" IS NULL AND "comments"."parent_id" = ?  [["parent_id", 78]]
>   ↳ app/views/comments/_comment.html.erb:17
>   Rendered comments/_comment.html.erb (Duration: 1.9ms | Allocations: 1139)
>   Rendered posts/show.html.erb within layouts/application (Duration: 16.5ms | Allocations: 5253)

<!-- 此頁面渲染時間 -->
> Rendered posts/show.html.erb within layouts/application (Duration: 16.5ms | Allocations: 5253)

<!-- 最終時間 -->
> Completed 200 OK in 70ms (Views: 50.6ms | ActiveRecord: 4.4ms | Allocations: 15167)
```

可以看到，大約渲染出一個留言，要花 `1177`，此頁面選染時間 `5253`，最終渲染該頁面的時間大約是 `15167`。


### 用collection的寫法

```html
<h4>我是大家的留言</h4>

<ul class="ul">
  
  <%= render @comments %>

</ul>

```


完整數據，但是這邊數據太雜了，我把重要的部分欻出來
```md
>   Rendering posts/show.html.erb within layouts/application
>   Comment Load (0.6ms)  SELECT "comments".* FROM "comments" WHERE "comments"."deleted_at" IS NULL AND "comments"."post_id" = ?  [["post_id", 12]]
>   ↳ app/views/posts/show.html.erb:59
>   Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."deleted_at" IS NULL AND "comments"."parent_id" = ?  [["parent_id", 76]]
>   ↳ app/views/comments/_comment.html.erb:17
>   Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."deleted_at" IS NULL AND "comments"."parent_id" = ?  [["parent_id", 77]]
>   ↳ app/views/comments/_comment.html.erb:17
>   Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."deleted_at" IS NULL AND "comments"."parent_id" = ?  [["parent_id", 78]]
>   ↳ app/views/comments/_comment.html.erb:17
>   Rendered collection of comments/_comment.html.erb [3 times] (Duration: 9.1ms | Allocations: 3856)
>   Rendered posts/show.html.erb within layouts/application (Duration: 65.1ms | Allocations: 6496)
> [Webpacker] Everything's up-to-date. Nothing to do
>   Rendered shared/_navbar.html.erb (Duration: 1.1ms | Allocations: 794)
>   Rendered shared/_flash.html.erb (Duration: 0.6ms | Allocations: 222)
>   Rendered layout layouts/application.html.erb (Duration: 78.6ms | Allocations: 14941)
> Completed 200 OK in 94ms (Views: 80.4ms | ActiveRecord: 1.3ms | Allocations: 17792)
```


重點擷取：該篇文章目前有三則留言，跟前面沒用collection相比，這邊多了一個很有趣的東西 `[3 times]`
```md
>   Comment Load (0.6ms)  SELECT "comments".* FROM "comments" WHERE "comments"."deleted_at" IS NULL AND "comments"."post_id" = ?  [["post_id", 12]]
>   ↳ app/views/posts/show.html.erb:59
>   Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."deleted_at" IS NULL AND "comments"."parent_id" = ?  [["parent_id", 76]]
>   ↳ app/views/comments/_comment.html.erb:17
>   Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."deleted_at" IS NULL AND "comments"."parent_id" = ?  [["parent_id", 77]]
>   ↳ app/views/comments/_comment.html.erb:17
>   Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."deleted_at" IS NULL AND "comments"."parent_id" = ?  [["parent_id", 78]]
>   ↳ app/views/comments/_comment.html.erb:17
>   Rendered collection of comments/_comment.html.erb [3 times] (Duration: 9.1ms | Allocations: 3856)
>   Rendered posts/show.html.erb within layouts/application (Duration: 65.1ms | Allocations: 6496)


> <!-- 此頁面渲染時間  -->
> Rendered posts/show.html.erb within layouts/application (Duration: 65.1ms | Allocations: 6496)

<!-- 最終時間 -->
> Completed 200 OK in 94ms (Views: 80.4ms | ActiveRecord: 1.3ms | Allocations: 17792)
```

可以看到有一行有 `3 times`，那個東西代表的是你渲染東西的次數，因為今天這一篇文章有3則留言，也就是迴圈裡面有3個物件，因此會顯示 `3 times`，因此一個留言渲染出來的時間為 `3856 / 3 = 1285`，此頁面渲染時間 `6496` ，最終渲染該頁面的時間大約是 `17792`。  
   
  