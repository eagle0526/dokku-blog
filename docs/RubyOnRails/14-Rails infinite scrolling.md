---
title: ROR/Rails infinite scrolling
sidebar_label: "14. [ROR] Rails infinite scrolling"
description: Rails infinite scrolling
last_update:
  date: 2023-11-07
keywords:
  - ROR
sidebar_position: 14
---


:::tip
**前情提要：**         
幾乎每個網站都遇到，如果今天你有一千條訊息，你不會想一次載入他，你會使用添加分頁，又或是滾動自動加載的方式載入更多的資訊，今天我們就來用Turbo 和 Stimulus來達成這件事！
:::



先安裝Pagy插件
------



```shell
$ bundle add pagy
```



### pagy 設定

將 Pagy 的後端添加到app/controllers/application_controller.rb：
```rb
class ApplicationController < ActionController::Base
  include Pagy::Backend
end
```


將 Pagy 的前端助手添加到app/helpers/application_helper.rb：

```rb
module ApplicationHelper
  include Pagy::Frontend
end
```



### controller 設定

```rb
class HousesController < ApplicationController
  def index
    @pagy, @houses = pagy(House.all, items: 3)
  end
end
```

這邊我們把@pagy、@houses印出來看看分別是什麼東西

#### @pagy 

這個是物件套用pagy後的一些詳細參數
```md
> #<Pagy:0x00000001112b7610 @vars={:page=>1, :items=>3, :outset=>0, :size=>[1, 4, 4, 1], :page_param=>:page, :params=>{}, :fragment=>"", :link_extra=>"", 
> :i18n_key=>"pagy.item_name", :cycle=>false, :request_path=>"", :count=>6}, @count=6, @page=1, @outset=0, @items=3, @last=2, @pages=2, @offset=0, @params={}, @from=1, 
> @to=3, @in=3, @prev=nil, @next=2>
```

#### @houses

因為我再items那邊填上三個，所以一開始只會印出3個房子(物件)
```md
> #<ActiveRecord::Relation [#<House id: 9, title: "另外一個人新增的房屋", description: "珊瑚屋", tel: "", address: "", owner: "", user_id: 2, created_at: "2023-04-02 08:26:28.872131000 +0000", updated_at: "2023-04-26 07:14:46.085853000 +0000", deleted_at: nil, slug: "83ab3996-6ad6-49c1-b4ea-2dbc4fd33c55", houses_count: 67>, 

> #<House id: 12, title: "第三個人的新房屋", description: "哇", tel: "", address: "", owner: "", user_id: 3, created_at: "2023-04-03 06:28:21.059926000 +0000", updated_at: "2023-04-04 09:41:36.361837000 +0000", deleted_at: nil, slug: "第三個人的新房屋-哇", houses_count: 204>, 

> #<House id: 14, title: "超進化房屋", description: nil, tel: "091011111222", address: "中壢市", owner: "惡房東", user_id: 2, created_at: "2023-04-26 07:17:20.089667000 +0000", updated_at: "2023-04-26 07:17:20.099772000 +0000", deleted_at: nil, slug: "超進化房屋", houses_count: 10>]>
```


Ps. 如果今天是要渲染house的物件，就記得要在house_controller，要不然會讀取不到喔，因為插件預設的關係，像是如果今天想要選染house的分頁效果，但是如果是寫在home_controller，會做不出分頁效果喔




第一種 - 使用上一頁和下一頁按鈕進行分頁
------


### view 設定

接下來我們把上一頁、下一頁的按鈕加上去，這邊有個重要的事情要提醒一下


(1) 渲染出房子內容

```html
<!-- views/houses/index.html.erb -->
<div id="houses" class="min-w-full">
  <%= render @houses %>
</div>
```

額外提醒：
```md
> 下面這一行是縮寫
> <%= render @houses %> 

> --------

> 最原始型態
> <% @houses.each do |house| %>
>   <% render "house", house: house %>
> <% end %>


> 不過因為上面那樣寫會造成效率問題，所以改成這樣(用collection寫法)
> <%= render partial: "houses/house", collection: @houses, as: :house %>

> 最後可以縮寫成
> <%= render @houses %> 
```


(2) 增加上一頁和下一頁內容

```html
<!-- views/houses/index.html.erb -->

<div id="pager" class="min-w-full my-8 flex justify-between">
  <div>
    <% if @pagy.prev %>
      <%= link_to "< Previous page", houses_path(page: @pagy.prev), class: "rounded py-3 px-5 bg-gray-600 text-white block hover:bg-gray-800" %>
    <% end %>
  </div>
  <div class="text-right">
    <% if @pagy.next %>
      <%= link_to "Next page >", houses_path(page: @pagy.next), class: "rounded py-3 px-5 bg-gray-600 text-white block hover:bg-gray-800" %>
    <% end %>
  </div>
</div>
```


完成的程式碼
```html
<!-- views/houses/index.html.erb -->

<div class="w-full">

  <div class="flex justify-between items-center">
    <h1 class="font-bold text-4xl">Houses</h1>
    <%= link_to 'New houses', new_houses_path, class: "rounded-lg py-3 px-5 bg-blue-600 text-white block font-medium" %>
  </div>

  <div id="houses" class="min-w-full">
    <%= render @houses %>
  </div>
  
  <div id="pager" class="min-w-full my-8 flex justify-between">
    <div>
      <% if @pagy.prev %>
        <%= link_to "< Previous page", houses_path(page: @pagy.prev), class: "rounded py-3 px-5 bg-gray-600 text-white block hover:bg-gray-800" %>
      <% end %>
    </div>
    <div class="text-right">
      <% if @pagy.next %>
        <%= link_to "Next page >", houses_path(page: @pagy.next), class: "rounded py-3 px-5 bg-gray-600 text-white block hover:bg-gray-800" %>
      <% end %>
    </div>
  </div>
</div>
```

這樣我們就完成用上一頁和下一頁的按鈕來達成換頁功能，接下來我們要用Turbo來瀏覽頁面

### 使用 Turbo 來瀏覽頁面

接著我們用 `turbo_frame_tag`，把渲染的房子、pager頁面給包起來，這樣我們可以利用turbo的特性，可以只抽換特定區塊，不用整個頁面重整
Ps. 這樣速度很快

```html
<!-- views/houses/index.html.erb -->

<div class="w-full">
  <% if notice.present? %>
    <p class="py-2 px-3 bg-green-50 mb-5 text-green-500 font-medium rounded-lg inline-block" id="notice"><%= notice %></p>
  <% end %>

  <div class="flex justify-between items-center">
    <h1 class="font-bold text-4xl">houses</h1>
    <%= link_to 'New widget', new_house_path, class: "rounded-lg py-3 px-5 bg-blue-600 text-white block font-medium" %>
  </div>

  <%= turbo_frame_tag "houses", class: "min-w-full" do %>
    <%= render @houses %>
    <%= render "pager", pagy: @pagy %>
  <% end %>
</div>
```

再來因為我們剛剛有渲染 `_pager` 頁面，所以我們現在把這個頁面補完，這個頁面主要放的是上一頁和下一頁的按鈕，這邊要特別注意到兩個按鈕跟前面最大的差異，就是兩個按鈕都掛上的 `turbo-action`。
```html
<!-- views/houses/_pager.html.erb -->

<div id="pager" class="min-w-full my-8 flex justify-between">
  <div>
    <% if pagy.prev %>
      <%= link_to(
        "< Previous page",
        houses_path(page: pagy.prev),
        class: "rounded py-3 px-5 bg-gray-600 text-white block hover:bg-gray-800",
        data: {
          turbo_action: "advance"
        }
      ) %>
    <% end %>
  </div>
  <div class="text-right">
    <% if pagy.next %>
      <%= link_to(
        "Next page >",
        houses_path(page: pagy.next),
        class: "rounded py-3 px-5 bg-gray-600 text-white block hover:bg-gray-800",
        data: {
          turbo_action: "advance"
        }
      ) %>
    <% end %>
  </div>
</div>
```

這樣改完之後，現在到瀏覽器操作，會發現當你按下按鈕時，網頁不會重整，而是原地抽換房子區塊。

但是這邊會有一個小問題，如果今天房子內容很多，你想要讓使用者按下按鈕時，重新捲回頁面最上方，你可以加上這一段：
```html
<!-- views/houses/index.html.erb -->

<div class="w-full">
  <% if notice.present? %>
    <p class="py-2 px-3 bg-green-50 mb-5 text-green-500 font-medium rounded-lg inline-block" id="notice"><%= notice %></p>
  <% end %>

  <div class="flex justify-between items-center">
    <h1 class="font-bold text-4xl">houses</h1>
    <%= link_to 'New widget', new_house_path, class: "rounded-lg py-3 px-5 bg-blue-600 text-white block font-medium" %>
  </div>

  <%= turbo_frame_tag "houses", class: "min-w-full", autoscroll: "true" do %>
    <%= render @houses %>
    <%= render "pager", pagy: @pagy %>
  <% end %>
</div>
```


第二種 - 帶有加載更多的按鈕
------

接下來我們要使用 `Turbo Streams`，因為 `Turbo Frame` 是直接把 `Frame` 裡面的內容 `整個` 替換，而 `Turbo Streams` 可以根據需要做替換、刪除、附加、前置內容(Turbo Streams can replace, remove, append, and prepend content as desired)



### 建置 Turbo Frame

首先我們先建置一個空的 `Turbo Frame`

```html
<!-- views/houses/_index.html.erb -->

<div class="w-full">
  <% if notice.present? %>
    <p class="py-2 px-3 bg-green-50 mb-5 text-green-500 font-medium rounded-lg inline-block" id="notice"><%= notice %></p>
  <% end %>

  <div class="flex justify-between items-center">
    <h1 class="font-bold text-4xl">Widgets</h1>
    <%= link_to 'New widget', new_widget_path, class: "rounded-lg py-3 px-5 bg-blue-600 text-white block font-medium" %>
  </div>


  <%= turbo_frame_tag "page_handler" %>

  <div id="houses" class="min-w-full">
    <%= render @houses %>
  </div>
  <%= render "pager", pagy: @pagy %>
</div>
```

在這裡，我們新增了一個沒有內容的 `page_handler` Turbo Frame，並刪除了houses。這個空的 `page_handler 框架` 是從服務器偷偷進入我們的 Turbo Stream 內容的信使，不需要修改標題。   

接著我們來查看實際效果，請更新 `pager部分` 以刪除舊的分頁並將它們替換為單個加載更多鏈接：  
```html
<!-- views/houses/_pager.html.erb -->

<div id="pager" class="min-w-full my-8 flex justify-center">
  <div>
    <% if pagy.next %>
      <%= link_to(
        "Load more houses",
        houses_path(page: pagy.next),
        class: "rounded py-3 px-5 bg-gray-600 text-white block hover:bg-gray-800",
        data: {
          turbo_frame: "page_handler"
        }
      ) %>
    <% end %>
  </div>
</div>
```

請注意，加載更多連結針對的是 `page_handler` Turbo Frame，去通知 `Turbo單擊` 該連結 `替換page_handler框架` 的內容，而不是重整整個頁面。
因為加載更多連結沒有嵌套在框架內page_handler，所以我們需要這個屬性來定位那個框架。



現在我們有了針對 `空 Turbo Frame 的分頁控制`，但是單擊連結只會重新渲染 `app/views/widgets/index.html.erb` 一個空page_handler框架。那不是很有用。
為了使這項工作正常進行，我們需要更新我們的 `controller` 以啟用 `turbo_frame variants`，這讓我們可以 回應 `Turbo Frame 的請求`，在 `index action` 中渲染不同的內容。  


```rb
# app/controllers/application_controller.rb

class ApplicationController < ActionController::Base
  include Pagy::Backend

  before_action :turbo_frame_request_variant
  
  private

  def turbo_frame_request_variant
    request.variant = :turbo_frame if turbo_frame_request?
  end
end
```

在這裡，我們使用 turbo-rails 方法 `turbo_frame_request?` 來識別入站 Turbo Frame 請求。    
當入站請求是 Turbo Frame 時，我們告訴我們的控制器以 `turbo_frame variant` 而不是正常的 html.erb 內容來響應。   
   

要查看實際效果，請為該動作創建新的 `Turbo Frame variant`，我們來新增一個頁面：

```shell
$ touch app/views/widgets/index.html+turbo_frame.erb
```

然後在這個頁面寫上這一段：
```html
<%= turbo_frame_tag "page_handler" do %>
  <%= turbo_stream_action_tag(
    "append",
    target: "houses",
    template: %(#{render @houses}) 
  ) %>
  <%= turbo_stream_action_tag(
    "replace",
    target: "pager",
    template: %(#{render "pager", pagy: @pagy})
    ) %>
<% end %>
```


在這裡，我們使用 `page_handler` Turbo Frame 進行回應，因為 Turbo 期望我們在單擊 `加載更多` 連結時呈現內容。  

Turbo Frame 的內部是魔法發生的地方。我們首先渲染一個 `Turbo Stream`，它將附加@houses到現有的小部件列表（使用houses id）。  
然後我們渲染另一個 `Turbo Stream` 以用更新版本的分頁器替換 `pager` div 的內容。  


Turbo 在客戶端看到回應，用兩個 `turbo-stream` 元素 `替換` page_handler Turbo Frame 標記的內容，然後處理在這些 turbo-streams 中定義的操作。   
最終結果是一組新的小部件附加到列表，並更新加載更多按鈕以獲取下一頁結果。   

通過前往小部件索引頁面並單擊加載更多按鈕來查看實際效果。 如果一切順利，每次單擊加載更多按鈕都會將更多小部件添加到列表中並每次增加頁碼。  




第三種 - 自動無限滾動
------

參考資料：[https://www.colby.so/posts/pagination-and-infinite-scrolling-with-hotwire](https://www.colby.so/posts/pagination-and-infinite-scrolling-with-hotwire)