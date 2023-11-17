---
title: ROR/Ruby
sidebar_label: "DAY36. [ASTROCamp] Ruby-13"
description: Ruby
last_update:
  date: 2022-12-01
keywords:
  - 5xRuby
  - Ruby
sidebar_position: 37
---



1、專案Table建議
------

1. 藍table的欄位中，type是保留字，不要使用  
2. phone都是integer，因為電話還有這種類型的 - 0800-137-000  
3. password是string 
4. admin的意思是上帝視角的後台，所以店長的話建議用別的名稱  
5. 用role來控制user的身份，ex. user表單有   
6. sti, mti 與多型關聯，如果今天有很多類似的表格，用繼承的概念，把共通的table開在同一個表格，用繼承的概念去管理 

8. 幫店家新增一個軟刪除，假設我把店家刪掉，其他店家的相關資料還會存在，每個表格都建議加上軟刪除 

11. 商店和位子的關聯設定建議，商店和位子的類型是一對多關聯，不是多對多  

12. 店家的分類要改成tag的方式(多對多)，如果真的要做分類的model用category    

14. Order的表單名字取好(我們的是訂位系統)，並且用關聯表來表達有哪個欄位。   

15. 不一定要用serial，可以用slug配上friendly_ID，把原本serial轉成一段指定的文字  

16. 商店的id也可以使用friendly_id來轉成有意義的slug     
17. acts_as_taggable套件，可以用來處理標籤(acts_as類型的都很好用)   

18. 如果今天對多對多取名沒想法，多對多表單的取名有個慣例，可以用兩張表的英文字母排序合併後來取名    

19. 如果今天有庫存系統，產品中有一個stock欄位，不能直接去改動這個欄位，要不然後面的消費者看到的產品數量都是0(都被買完了)，所以真的要做的話，可以在另外一個表統計該產品數量，比如再開一個表單，統計產品數量的現存狀態    

20. carrierwave是處理圖片存放的的套件，可以再Product的table裡面有一個avatar:string的欄位(放圖片路徑)，他會指向另外一個表單，裡面有所有資料的路徑(別組案例)  

Ps. 有一個更好用的官方插件是Active Storage，他可以幫你處理圖片存放的問題，不用開任何欄位，只要照著他的操作，就可以把圖片上傳，如果不會用到上傳圖片，試試把Active Storage關掉，他的概念是，開一個table，幫你存放一堆圖片  







2、rails 7 特色
-------

> ---     
> action cable是做聊天室的套件  
>    
> ---    

在application.rb裡面，原本都是require "all"，現在已經可以自己取消(用不到就把它拔掉)  
```md
> application.rb

> require "rails/all"
```

### 2-1、tailwind

rails 7 安裝tailwind變得非常簡單，只要在一開始new專案的時候，順便加上後面一段指令就可以順便安裝了
```md
> rails new BookStore --css=tailwind
```

> ---  
> **預設ruby版本**
> rvm use 3.1.1 --default  
>    
> ---    


如果剛剛有裝tailwind，我們可以執行這個指令
rails 7 預設使用foreman s，只要執行他，就可以一次啟動多個server
```md
> bin/dev
```


### 2-2、importmaps & node_modules

rails 7 預設把node_modules關掉，那要怎麼使用原本node_modules的功能呢？importmaps解決了這件事情

還記得我們原先怎麼使用node_modules嗎？
就像下面這樣
```md
> yarn add dayjs                # 先把yarn add 某個套件
> package.json <-- dayjs        # 會把剛剛的套件塞一份到package裡面
> node_modules/ <-- day.js      # 也會裝一份在node_modules裡面
```

引入之後還要import來用
```md
> import moment form "moment"
```

### 2-3、importmaps 實際操作

不過用了importmaps，我們就可以不用把安裝那一包node_modules了，而且實際上import就像下面那樣
類似CDN的概念，不用把要import的檔案放在node_module，只要直接去雲端空間找就好
```md
> import moment form "/node_modules/sdf/sfdsf"
```

不過在某個套件誕生之前，importmaps在所有瀏覽器上的支援率只有70幾趴，非常的少，所以很少人敢用，  
不過自從es-module-shims，有了這個東西，直接讓importmaps的支援度衝到95趴以上，因此讓很多人   
開始使用了  


:::tip
rails 7 開始使用importmap的原因，是因為有shims的存在  
:::



輸入下面這一行後，會自動在config->importmap裡面生成一段code
```md
> bin/importmap pin dayjs
```

下面這一段會自動生成在config/importmap裡面
```md
> pin "dayjs", to: "https://ga.jspm.io/npm:dayjs@1.11.6/dayjs.min.js"
```

這樣放上去後，就可以正常在使用dayjs了！！
```md
> hello controller

> import xyz from "dayjs"

> connect() {
>   console.log(xyz)    
> }
```


:::tip  
#### HTTP/2   
不過這邊有一個問題，假設今天Pin了很多插件，那不就會讓header那邊會有一大堆request嗎    
不過這因為有HTTP/2出來，所以解決了這個問題  
   
在HTTP1.1的時代，的確是建議大家減少request的數量，可以的話就把一大堆的JS檔案，打包成一個JS檔案，  
這樣除了可以減少網站的JS檔案數量，也可以減少request數量   
  
不過到了HTTP的時代，開始可以多管線平行同時下載，如果今天importmap有一大堆，這樣雖然有很多的request出   
現在HTML的script那邊，但是對於同一個網域，他們只會做一個handshake(以前的時代，一個request就要  
handshake一次)，在HTTP2的時代，一個網域只要做一次handshake    
:::





### 2-4、rails7 CRUD差異

Book
 - title
 - content:text


把按鈕都統一格式後，在命名一個css，並用@apply把所有屬性塞進去，這樣就可以用用簡短的名字就可以達成
```md
> .btn-red {
>     @apply px-2 py-3 xxxxxxxxxxx;
> }
```

    
12/01 15:20 
### rails 7 的 turbo-links發展

turbo分成三個東西
- turbo drive = Turbolinks - 負責前進、後退、移動、改網址之類的  
- turbo stream = js.erb - 前端和後端交換資訊用的     
- turbo frame => 這個是rails 7 新加進來的，他可以把一個標籤抽換到另外一個地方  

:::tip
以前做網站有一個東西叫做 `<iframe>` 的概念，他可以抽換裡面的東西
:::

### turbo-frame 範例

(1) 先創造一個comments/_form.html.erb，這個表單是等等new.html.erb、edit.html.erb要渲染的檔案    
```md
> <section>    
>   <%= form_with model: @comment, url: url do |f| %>
> 
>     <%# 預設情況下，parent是nil，如果今天parent是有數值的，就多下面這個欄位，這個欄位是要用來新增子留言的  %>
>     <% if !parent.nil? %>    
>       <input type="hidden" name="parent_id" value="<%= parent.id %>">
>     <% end %>
>     <div class="my-4">
>       <%= f.label :content, '留言' %>      
>       <%= f.text_area :content %>
>     </div>
> 
>     
>     <div class="my-4">      
>       <%= f.submit class: 'btn-primary' %>
>     </div>
>     
>   <% end %>
> </section>
```

(2) 接著設定edit.html.erb，我們要把表單渲染出來
```md
> <h2>留言編輯</h2>
> <%= render "form", house: @house, comment: @comment, parent: @comment, url: comment_path(@comment) %>
```

(3) 幫渲染表單的地方，最外面加上一層turbo_frame_tag，記得要加上dom_id，這樣才可以幫每一則留言附上不同的id
```md
> <h2>留言編輯</h2>
> 
> <%= turbo_frame_tag dom_id(@comment) do %>
>   <%= render "form", house: @house, comment: @comment, parent: @comment, url: comment_path(@comment) %>
> <% end %>
```

(4) 幫想要使用到turbo_frame的地方，加上turbo_frame_tag，這樣寫的話，就可以幫這一段全部加上turbo_frame_tag，所以當我們按下"編輯"時，就會直接把edit.html.erb的編輯欄位抽換過來。
```md
> <%= turbo_frame_tag dom_id(comment) do %>
>  <article data-controller="comment-reply" class="">
>    
>    <p>
>      <span><%= comment.content %></span>
>      <span class="ml-4 text-xs text-slate-400"><%= comment.created_at %></span>
>    </p>
>    <small class="mr-4">by <%= comment.user.name %></small>
>    <small class="mr-4">
>      <%= link_to '編輯', edit_comment_path(comment) %>
>    </small>
>    <small class="mr-4 cursor-pointer" data-action="click->comment-reply#reply">回覆</small>
>    <% if comment.user_id == current_user.id && comment.comments.size == 0 %>
>      <small class="mr-4">    
>        <%= link_to '刪除留言', comment_path(comment), data: {turbo_method: "delete", confirm: "確定嗎？"}, class: "text-red-500" %>
>      </small>
>    <% end %>
>  
>    <div class="hidden" data-comment-reply-target="form">
>      <%= render partial: 'comments/form', locals: { house: comment.house, parent: comment, url: comments_path(comment.house_id) } %>          
>    </div>
>    
>    <hr>>  
>    <div class="m-10">
>      <%= render comment.comments %>
>    </div>
>    
>  </article>
> <% end %>
```



15:45 
rails 7的刪除寫法改了
```html
<%= link_to "刪除", book_path(@book), data: {turbo-confirm: "sure?", turbo-method: "delete"} %>
```
而且表單也被turbo抓走了，所以不想要被抓走的話，可以把這個頁面的turbo關掉




16:00
hotwire
HTML OVER THE WIRE


turbo-stream

destroy

原本的destroy.js.erb
變成destroy.turbo_stream.erb


有一個turbo-stream訊號過來，就會變成一行
```html
<%= turbo_stream.remove dom_id(@book) %>
```


這一行這樣寫，就可以跟之前做JS刪除的方法一樣了




```html
<!-- create -->
<%= turbo_stream.append "book_list", @book %>   -> 在最後面塞一個
<%= turbo_stream.prepend "book_list", @book %>  -> 在最前面塞一個
```




```html
<!-- edit -->
<%= turbo_stream.replace %>
```
