---
title: ROR/Ruby
sidebar_label: "DAY27. [ASTROCamp] Ruby-5"
description: Ruby
last_update:
  date: 2022-11-15
keywords:
  - 5xRuby
  - Ruby
sidebar_position: 28
---



1、WishList製作功能 - CR
-----

### 1-1、第零步驟：路徑設定routes

來做許願頁面，先給出許願頁面的首頁、new、create功能
```md
> get "/make_a_wish", to: "wish_list#card"              # 許願首頁
> get "/new_wish", to: "wish_list#new_wish"             # 許願新增頁
> post "/wish_card_list", to: "wish_list#create_wish"   # 許願新增頁的表單提交地方

> Verb "網址", to: "controller#action"                   # 此為routes格式 
```



### 1-2、第一步驟：新增表單new form

到new頁面的把form功能做出來
```md
> <form action="/wish_card_list" method="post">
>     <input type="text" name="title">
>     <textarea name="description" id="" cols="30" rows="10"></textarea>
>     <button>送出</button>
> </form>
```

#### 1-2-1、form action
form 的 action 是指表單送出後，要把資料送去的地方  

#### 1-2-2、form method
form 的 method 的 預設是get

:::tip **get和post的差異**  
用get的方式送，欄位的name名稱會轉成queryString，並傳遞給下一個頁面   
用post的方式送，input的name參數，不會顯示在queryString上  
如果今天做登入功能，不能用get方法傳，圖片之類的也是，要要用post傳  
Ps.post只是沒有顯示input資料在網址上，如果今天用console抓，還是抓的到    
:::

#### 1-2-3、input name
:::tip
input會被抓到的是name值很重要，後端抓的是這個值     
:::  


### 1-3、第二步驟：authenticity_token介紹

接著到wish_list的controller，我們把表單input送過來的東西印出來
```md
> def create_wish
>  render html: params              # 這裡直接 params 就可以抓到那一包
> end
```

不過這時候會發生一個錯誤，authenticity_token，這個是什麼呢？？  
其實這個就是rails的安全系統，假設你今天表單送出的token，是正常管道送進去  
並且有遵守自己檔案內key值產生的原則，系統就會讓你送進資料庫  

:::info
ActionController::InvalidAuthenticityToken  
這個是預設打開的，如果表單有送出token，並且是正常管道送出去的，rails就讓你通過  
:::

:::danger
authenticity_token -> config檔案裡面有一個master.key檔案，裡面的key非常重要，key掉了，就爆了  
:::



### 1-4、第三步驟：解決authenticity_token

在form表單，加上一個input，就可以讓資料表做token驗證(如果屬性不給hidden，他會顯示在畫面上)
```md
> <input type="hidden" name="authenticity_token">
```

我們實際把表單送出的東西，印出之後就是這一大串，可以發現就是一大串亂碼
```md
> render html: params           # 把表單送過來的資料印出來

> {"authenticity_token"=>"S57jwgxx7R4vr0g0iBRHMd7dIbcc9Ept980Rr9dmgRKMMkhHAv8JJS9YaVbOuVrT0hVJWJulQLwZ2RRCi_hXRw", "title"=>"sdf", "content"=>"qwe", "controller"=>"wish_list", "action"=>"create_wish"}
```




### 1-5、第四步驟：表單input name用中括號包起來

把input欄位的值，前面加上一個字串，wish_list[title]這樣可以用wish_list把中括弧裡面的東西變成hash格式
```md
  <form action="/wish_card_list" method="post">
      <input type="hidden" name="authenticity_token">

>     <input type="text" name="wish_list[title]">                                   -> 原本是title
>     <textarea name="wish_list[description]" id="" cols="30" rows="10"></textarea> -> 原本是description

      <button>送出</button>
  </form>
```


上面兩個name值更改後，用檢查工具查看網頁html，會發現表單的input欄位會變這樣    
```md
> title、content被wish_list包成一個hash了 
> "wish_list"=>{"title"=>"sdfdsf", "content"=>"eqwe"}
```


如果實際印出來
```md
> render html: params[:wish_list]           # 把[:wish_list]印出來

> {"title"=>"sdfdsf", "content"=>"eqwe"}
```



### 1-6、第五步驟：ForbiddenAttributesError介紹

因為剛剛改過form的input後，我們可以一次抓取input的東西，把這一包塞進表單中
```md
> wish_list = WishList.new(params[:wish_list])

> params[:wish_list]剛剛有解釋了，就是因為在表單的name值那邊有動手腳
```


但!!!這樣寫之後，跑出了另外一個error => ForbiddenAttributesError  
這個東西是什麼呢？  
假設今天有奇怪的欄位(容易被猜中亂搞的欄位)，惡意攻擊者可以直接在html那邊寫入資料，這樣就可以搞亂你的資料庫了  
又因為(params[:wish_list] -> 這個是一大包的)，所以假如今天有人偷偷把資料塞進這一包，是塞得進來的  
所以rails做了一個安全機制，什麼安全機制呢？就是你要匯進去的欄位，要精確的指定  



### 1-7、第六步驟：ForbiddenAttributesError解決方法
要怎麼解決這個問題呢？？rails 有一個再一次驗證的寫法  
permit後面要接的，就是允許匯進資料庫的欄位  
```rb
clean_params = params.require(:wish_list).permit(:title, :description)
```

可以把clean_params印出來看看
```md
> render html: clean_params

> "title"=>"sdfdsf", "content"=>"eqwe" => 他只會印出上面permit裡包含的欄位
```


:::tip **params.require Vs. params[:wish_list]**  
用 params.require 的原因是，require會做額外的檢查  
要不然用params[:wish_list]也可以  
clean_params 這個專有名詞叫做 Strong Parameters、也就是一個白名單的概念  
:::



### 1-8、第七步驟：資料成功送進資料庫

把前面的安全機制都設定好後，就可以成功把資料送進資料庫了
```rb
# create action 

clean_params = params.require(:wish_list).permit(:title, :description)
wish_list = WishList.new(clean_params)

if wish_list.save
  redirect_to make_a_wish_path
else
  render html: "fail"
end
```


### 1-9、第八步驟：flash[:notice]、flash[:alert]設定

接下來我們來設定，當成功送出、失敗時，要顯示的訊息   
首先到application.html，公版的位置，把flash的顯示條件設定好  
```md
> <% if flash.any? %>       
>     <div style = "background-color: green; color: white">
>         <%= flash[:notice] %>
>     </div>
>     <div style = "background-color: red; color: white">
>       <%= flash[:alert] %>
>     </div>
> <% end %>
```

之後到controller把flash功能加上去
```md
> if wish_list.save
>     redirect_to make_a_wish_path, notice: "success new wish_list" -> 這個notice是簡寫，和redirect一起寫
> else
```



### 1-10、第九步驟：輸入欄位驗證

可以在想要限制model某個欄位的值，是否一定要輸入才能送出，還有很多類型的驗證方式，之後有機會再補充
```md
> validates :title, presence: true              # 這個意思是，title欄位一定要填寫
> validates :description, presence: true        # 這個意思是，description欄位一定要填寫
```



### 1-11、第十步驟：表單改寫 form_helper
前面表單產生太囉唆了，要產生token，又要把input name中的東西包起來，因此我們來用form_helper來寫  
```md
> 1. form_tag => form_tag(url)    > 做功能，不需要用到model，就用form_tag。ex. 做BMI計算際就不用model  
> 2. form_for => form_for(@model) > 做功能，不需要用到model，就用form_for。ex. 普通有做有model功能的表單，就是用此方法  
> 3. form_with => form_hash(HASH) > 融合上面兩種寫法  
```

#### 1-11-1、for_for使用注意事項
:::tip
(1) form_for 後面接的是一個model實體    
(2) 如果今天url沒有填，rails會猜，你的路徑是跟前面給的WishList很像，他會猜wish_list_path，但是因為今天路徑是我們自己客製的，所以這裡的url要自己寫   
(3) form_for後面要接的實體，不要在這邊new物件，先在controller寫完，並創一個實體變數，再傳給view用就好   
(4) 今天這樣寫@wish_list = WishList.new(title: 123)，title那個欄位一開始會有預設值123，可以傳過去的關係就是help幫你的   
:::

#### 1-11-2、controller更改
controller更改，model.new實體在controller寫好，再傳給view  
```rb
def new_wish
  @wish_list = WishList.new
end
```

#### 1-11-3、view更改
view頁面更改，記得form_for後面第一個參數是model.new，第二個參數是url  
```md
> <%= form_for @wish_list, url: "/wish_card_list" do |form| %>
>   <%= form.label :title %>
>   <%= form.text_field :title %>
> 
>   <%= form.label :description %>
>   <%= form.text_area :description %>
> 
>   <%= form.submit %>
> <% end %>
```



#### 1-11-4、form_with 使用注意事項

1. 一定要寫model: @wishlist  
2. 在表單實體那邊，form_for 給nil就會噴錯，form_with給nil只會啥都不會印給你  
3. form_for在html那邊，多了一串id="new_wish_list"(id=new的路徑)

```md
> <%= form_with (model: @wishlist) do |form| %> .  => 表單實體就是@wishlist
>   <%= form.label :title %>
>   <%= form.text_field :title %>
> 
>   <%= form.label :description %>
>   <%= form.text_area :description %>
> <% end %>
```




### 1-12、第十一步驟：field_with_errors 欄位驗證 
這個欄位是當你有設定欄位驗證時，假如你沒有符合驗證，就會跳出這個欄位   
此時我們來改寫，如果create無法寫進資料庫(有寫validate)，此時失敗的話，我們要重新渲染新增表單頁面   
  
#### 1-12-1、render redirect原地重導
```md
> if @wish_list.save
>     redirect_to make_a_wish_path, notice: "success new wish_list"
> else
>     render :new_wish                      -> 如果存入資料庫失敗，就重新渲染網頁
> end
```



#### 1-12-2、更改css
欄位錯誤的話，HTML會有多一個欄位叫做field_with_errors，針對錯誤欄位增加CSS
```css
.field_with_errors {
    display: inline;    

    label[for="wish_list_title"] {
        color: red;
    }

    label[for="wish_list_description"] {
        color: red;
    }
    textarea[id="wish_list_description"] {
        border: 1px solid red;
    }

    input[id="wish_list_title"] {
        border: 1px solid red;
    }
}
```





### 1-13、第十二步驟：許願單的Read

接下來我們把model中的東西讀取出來  
controller的部分，用實體變數接住model所有的數據  
```rb
def card
  @wish_lists = WishList.all
end
```

view的部分，用迴圈把model中的所有數據印出來
```md
> <% @wish_lists.each do |wish_list| %>
>   <li>
>     <%= link_to wish_list.title, wish_card_info_path(wish_list.id) %>
>   </li>
> <% end %>
```




### 1-14、第十三步驟：許願單的show
再來把各別筆數的所有資料印出來   

controller的部分，用抓取個別連結的id   
```rb
def show_wish
    @wish_list = WishList.find_by(id: params[:num])
end
```


這邊是 `params[:num]` 的關係，是因為我在 `routes` 設定跟 `resources` 不太一樣，注意get後面第一個字串  
我是使用 `/:num`，因此最後 `params` 抓到就是 `{id: params[:num]}`
```rb
get "/wish_card_info/:num", to: "wish_list#show_wish", as: "wish_card_info"
```

:::tip
這邊會在routes用到as的關係是因為，這次我們是都用客製的，所以path很亂，基本上不太會用到   
如果以後有機會用到，其實他就是網址，仔細看他和get後面那一串很像，不過因為這一次我們亂打，  
所以rails他在抓取的時候，會判斷錯誤，所以我們要再加寫path    
:::


  
上面那樣可能有點模糊，我們實際把params id抓到的東西印出來
```md
> 網址連結 http://[::1]:3000/wish_card_info/9
```
  

到erb使用params把東西印出來，要抓的東西就是"num"=>"9"，這個hash中的值(9)
```md
> <%= params %>

> 印出這一串 params資料 {"controller"=>"wish_list", "action"=>"show_wish", "num"=>"9"}
```
  
  
解釋完為啥是params[:num]後，就可以到view頁面把資料印出來
```md
> <table>
>     <tr>
>         <td>wish_list ID</td>
>         <td>wish_list 姓名</td>
>         <td>wish_list 描述</td>
>     </tr>
>     <tr>
>         <td><%= @wish_list.id %></td>
>         <td><%= @wish_list.title %></td>
>         <td><%= @wish_list.description %></td>
>     </tr>
> </table>
```



2、rescue、rescue_from 
------
11/17 - 11.05


如果今天顧客直接更改網址，找到超出資料庫的資料，會讓網頁爆炸
```md
> 網址 : http://localhost:3000/wish_card_info/3464645757
```


### 2-1、使用rescue方法
**第一種發生錯誤解法**   


因此我們要加一點東西，讓網頁不會爆炸 
```rb
def create
    begin 
        @wish_list = WishList.find(params[:num])
        render html: @wish_list.title
    rescue
        render html: "找不到"
    end
end
```


### 2-2、使用rescue_from
**第二種發生錯誤解法**  
不過因為報錯很常見，我們把這個報錯的寫法，拉高層級，寫到controller那邊，並且新增一個方法  
這樣寫只要以後都有發生RecordNotFound錯誤，都會報錯  
  
寫在整個controller最上面  
```rb
rescue_from ActiveRecord::RecordNotFound, with: :not_found 
```
  
並把方法寫在private區塊
Ps. 如果錯誤，印出找不到，但是這樣不太對，應該要連到404頁面才對
```rb
private
def not_found
  render html: "找不到", status: 404
end
```


### 2-3、修改404頁面
  
從這個路徑，public > 404.html，就可以找到404.html，接著可以根據你想要的畫面，去修改404頁面




### 2-4、public資料夾說明
:::tip
Q. 為什麼404、422、500會放在public資料夾，不像其他routes一樣，在那邊寫路徑就好了？？    
A. 因為如果今天網站壞掉的話，public裡的檔案會先查看，不用進routes查看路徑，就可以解決了   
:::

放在public的東西，基本就是不需要要進routes的檔案    
ex. favicon, robots.text, 404, 500(robots那個是給爬蟲看的)    

最後利用rails特有語法Rails.root(rails內的檔案)，讓我們可以連結到404網頁，並且記得帶404狀態給他  
不過這時候如果開html看，會發現layout會和404頁面都存在，所以記得在這邊設定layout: false  
```rb
def record_not_found
    render file: "#{Rails.root}/public/404.html", status: 404, layout: false
end
```




3、show 頁面的路徑演化
------
非常重要，以下看完可以知道，原本用a-link連到不同id的內頁，後面是怎麼變成由.format方式的  
教學影音在ruby 11/15 最後一個影片的20:00 筆記要記得做完，了解這個流程，對以後queryString很有幫助      


接著到view頁面，想要點下連結可以跳轉到每個實體裡面的內容時


### 3-1、首先我們先設定路徑
目前路徑還是錯的，後面會轉成正確的
```rb
get "/wish_list_info", to: "wishlist#show"
```

接著我們把路徑連結放的view頁面，並用a-link連過去
```md
> <a href="/wish_list_info"><%= @wishlist.title %></a>
```

不過這樣設定會有一個問題，每個實體點了這個連結，導過去都是同一個連結(我想要達成的是，連結後方要帶著各自的id)  
因此我們帶著id給他，這邊以前用a-link，有兩種寫法  

#### 3-1-1、第一種設定法  
第一種?=id(會有點難看懂，簡單就是說，用字串組成的方式，把id組合在?=後面)
```md
> <a href="/wish_list_info?id=<%= wishlist.id %>"><%= @wishlist.title %></a>
 
> HTML印出下面這樣(假設我們今天按的連結是model的第一個實體，id會顯示為1)
> <a href="/wish_list_info?id=1"></a>
```

#### 3-1-2、第二種設定法
第二種/id(這種事比較常見的)
```md
> <a href="/wish_list_info/<%= wishlist.id %>"><%= @wishlist.title %></a>

> HTML印出下面這樣(假設我們今天按的連結是model的第一個實體，id會顯示為1)
> <a href="/wish_list_info/1"></a>
```



假設今天用a-link，也可以在後面帶其他字串給他(練習queryString)
```md
> 我在id後面又加了?a=1&b=2這一串
> <a href="/wish_list_info/<%= wishlist.id %>?a=1&b=2"><%= @wishlist.title %></a>

> HTML印出下面這樣(假設我們今天按的連結是model的第一個實體，id會顯示為1)
> <a href="/wish_list_info/1?a=1&b=2"></a>
```



假設我們今天在controller，用params把剛剛a-link東西抓下來看看
```md
> def show
>   render html: params
> end
```

可以發現剛剛顯示在網址上的東西，都被params抓成hash了
```md
> 網址：localhost:3000/wish_list_info/1?a=1&b=2

> params下來的數據是下面，如果想要抓到特定的key，就用params[:a]就好
> {"id"=>"1", "a"=>"1", "b"=>"2", "controller"=>"wishlist", "action"=>"show"}
```


### 3-2、修改路徑
經過剛剛這麼長的研究，可以知道要查看每一筆詳細資料的話，因為要抓每一筆的id，所以路徑那邊要給一個變數id

```md
> 原本路徑
> get "/wish_list_info", to: "wishlist#show"

> 帶入變數id的路徑
> get "/wish_list_info:id", to: "wishlist#show"
```



### 3-3、link_to寫法
不過其實前面用a-link寫很醜，rails有支援link_to的寫法，可以直接用簡短的程式碼達成a-link在做的事
```md
> <%= link_to @wishlist.title, wish_card_info_path(wishlist.id)  %>
```



4、SSR Vs. SPA
------

### 4-1、SSR = Server Side Rendering  
wiki：一個web頁面的數據渲染完全由客戶端或者瀏覽器端來完成  
常見於 rails  

### 4-2、SPA = Single Page Application
wiki：是一種網路應用程式或網站的模型，它透過動態重寫當前頁面來與使用者互動，而非傳統的從伺服器重新載入整個新頁面。  
常見於前端，React、vue  



:::tip **DoubleRenderError**  
遇到這個錯誤，代表你的render or redirect 在一個action 裡面，重複觸發兩次  
只有把其中一個刪掉就可以繼續運行  
:::




5、終端機BUG問題 control+z、fg
------
Q. 如果今天進到rails + s -> 並使用control + z -> 接著又打rails s，然後整個畫面就卡住了  
A. 解決方法為，使用fg(直接輸入fg，他就會把縮到背景的程式抓回來)  
  
Ps1. control + z 會把目前執行的東西，縮到電腦背景(他其實還在跑)  
Ps2. fg 意思是 fore ground => 可以把在背景執行的工作撿回來  






:::info **面試題**  
Q. SQL injection(注入)是什麼  
A. 也稱SQL隱碼或SQL注碼，是發生於應用程式與資料庫層的安全漏洞。  
簡而言之，是在輸入的字串之中夾帶SQL指令，在設計不良的程式當中忽略了字元檢查，  
那麼這些夾帶進去的惡意指令就會被資料庫伺服器誤認為是正常的SQL指令而執行，因此遭到破壞或是入侵。  
:::



6、script在html顯示為亂碼
------
Q. 為什麼rails中script的JS網址是一串亂碼？？  
  
**A. 舊方法**  
因為瀏覽器很常會遇到靜態頁面暫存問題，但是你不能跟消費者說清一下暫存  
因此早期要解決css/JS檔案暫存問題，使用queryString的方式，讓瀏覽者不會暫存此css檔案  
queryString解決方法，就是幫每個檔案灌一個亂數，消費者每次開一次都會換一個數字  
style.css?t=312312312  
style.css?t=312312313  
style.css?t=312312314  
但是這個會造成，每次使用者進到網頁都會使用到網頁的暫存(多一串queryString)  

**A. 新方法**  
所以後來更改成，只要JS內部檔案只要改過，程式就會幫你算出一串雜湊值  

:::tip
如果今天用rails寫JS不會動，或是不如你預期，是為什麼，要記得是因為turbolinks的關係  
:::