---
title: ROR/Rails 顯示被軟刪除的留言
sidebar_label: "16. [ROR] Rails 顯示被軟刪除的留言"
description: Rails 顯示被軟刪除的留言
last_update:
  date: 2023-11-07
keywords:
  - ROR
  - 巢狀留言功能
sidebar_position: 16
---


:::tip
**前情提要：**         
如果今天已經幫留言設定好了軟刪除，但是想要把被刪除的留言顯示出來，並且顯示的是特定文字(留言已被使用者刪除)，這樣要怎麼做呢？
:::


巢狀留言軟刪除
------

看過官方文件後，只要這樣輸入，就可以把不管有沒有被刪除過的東西都抓出來

```md
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














