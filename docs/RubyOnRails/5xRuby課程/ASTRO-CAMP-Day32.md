---
title: ROR/Ruby
sidebar_label: "DAY32. [ASTROCamp] Ruby-10"
description: Ruby
last_update:
  date: 2022-11-24
keywords:
  - 5xRuby
  - Ruby
sidebar_position: 33
---



1、金流串接
------

### 1-1、綠界手冊

串金流前，要先準備訂單，因為在傳API過去金流商的時候，要傳訂單編號給金流商

#### 1-1-1、什麼叫做幕前
在刷卡的過程中，綠界把你導到綠界的頁面，然後你填完信用卡資訊，照理來說你用導回原本的頁面  
這個過程你是看得到的，這個叫做幕前，因為瀏覽器是開著的，整個過程你都看得到  
**又可以稱作同步付款**  

#### 1-1-2、什麼叫做幕後
當你在使用ATM付款、銀行匯款之類的，你是在線下網站看不到的地方匯款，這種就稱作幕後  
**又稱做非同步付款**  

:::tip
**幕後流程解析**  
你會得到一個臨時付款的資訊，然後去超商繳費，等到你繳費完成的時候，綠界就會發一個POST  
POST到另外一個API，所以你要準備另外一個API是可以讓他打回來的  
  
這個API會接的是訂單編號、金額之類的，你就可以params他打回來的這包資訊，看訂單是哪一筆、狀態  
最後你再去資料庫把此筆資料改成已付款  
:::

:::tip
小知識  
國外大部分都是幕前就處理(信用卡)  
台灣則是兩種都常碰到  
:::

:::danger
金流商會丟訂單資訊回來  
記得params得時候，不要擋他的token，直接skip_before_action :authenticate_user!  
:::



#### 1-1-3、NotifyURL

假設今天綠界post過來網址是
```md
> Post /api/v2/cash/notify
```

金流的路徑要給絕對路徑，NotifyURL那邊的值要給完整的網址
```md
> <form method="post" action="藍新api">
>   <input type="hidden" name="NotifyURL" value="http://ddnews.com/api/v2/notify">
> </form>
```
在測試金流的時候，value要給絕對路徑，要不然金流商那邊會發生錯誤


### 1-2、ngrok 線上測試金流

ngrok 做為一個轉發的伺服器，他可以把外界的請求轉發到你指定的Port，使用的背景原理是連接到ngrok雲端伺服器，將你本機指定的地址公開，再將由ngrok一串公開的網址來存取內容。  
他的優點是快速而且還提供了https的服務讓你使用上更安全，甚至你還可以設置密碼保護  

:::tip
懶人包：ngrok這個插件可以線上測試金流  
:::
  
#### 1-2-1、安裝ngrok
使用homebrew安裝，執行以下指令 
```md
> brew cask install ngrok
```

#### 1-2-2、執行ngrok
並執行以下指令進來操作(終端機會有連結給你，暫時的短網址會指向local:3000)
```md
> ngrok http 3000
```

#### 1-2-3、暫時網址使用
可以建立一個暫時網址https://9bdd-61-220-182-115.jp.ngrok.io/，這個就可以塞進input中的value    
這樣就可以即時測試金流是否有成功  
```md
> <form method="post" action="藍新api">
>   <input type="hidden" name="NotifyURL" value="https://9bdd-61-220-182-115.jp.ngrok.io/api/v2/notify">
> </form>
```



之後上線之後，一定要記得改回原網址喔，要不然會導致別人付款成功，但是你會收不到客戶的付款狀態(你最後要自己一筆一筆對帳)，不過我們有方法可以防止上線後忘記改連結，這個方法就是設定環境變數，直接讓測試環境、上線環境使用不同的值   

:::tip
串接第三方服務的時候，都會用到ngrok  
ex. 串接google登入、金流  
Ps. 如果demo當天，雲端server爆炸了，可以用ngrok臨時補救，把localhost轉成一個暫時網址  
:::


2、環境變數
------
為了避免之後實際上線忘記改ngrok，我們可以來設定環境變數，就可以絕對的避免網站上線卻沒改到東西的情況   

:::tip
兩個設定環境變數的套件，這兩個都是在專門處理環境變數  
figaro  
[dotenv](https://github.com/bkeepers/dotenv)  
:::
  

### 2-1、dotenv是什麼

#### 2-1-1、安裝dotenv

輸入下面這一行，這樣寫會直接放在gemfile的最尾端  
這個寫法的好處是會產生版號  
```md
> bundle add dotenv-rails
```

#### 2-1-2、設定dotenv
我們要把產出的gem放在development、test裡面
```md
> groups :development, :test do
>   gem "dotenv-rails", "~> 2.8"
> end
```


!!這邊有一點要特別注意，如果你想儘早觸發環境變數，你可加上這一段
```md
> #config/application.rb
> 
> // Load dotenv only in development or test environment
> if ['development', 'test'].include? ENV['RAILS_ENV']
>   Dotenv::Railtie.load
> end
```




#### 2-1-3、在正式環境可以用dotenv嗎
:::tip
dotenv原本主要給測試環境的環境變數使用，在正式環境有更好的環境變數設置使用  
像是Puppet or Chef管理的/etc/environment  
不過真的要用也是可以用，不過記得把gem "dotenv-rails", "~> 2.8"搬到外面    
:::


#### 2-2、staging是什麼  
staging簡單說就是雲端平台上，測試環境、正式上線環境，中間再有一個環境給公司內部人於測試網站的地方   
staging的特色是，他的環境跟正式環境基本上是一模樣，所以如果在staging可以正常運作，正式版本也可以正常運作     

Ps. 開發的過程中會有一台雲端伺服器，來測試本機寫的code跑起來的狀況  

```md
>             工程師看的環境
> 本機(dev) -> 雲端平台dev ---------------------> 雲端平台Prod
>                           staging在這
>                           這個環境跟正式環境一模一樣
>                           然後給公司內部其他人測試
```





### 2-3、為什麼要放環境變數
 
#### 2-3-1、好處一：不同變數
因為環境變數不同，一樣的變數，會因為不同的環境造成不同的結果   
用剛剛的notifyURL舉例，一個是測試環境檔案，一個是上線環境檔案  

首先寫一個寫死的路徑
```md
> <%= link_to "rr", "http://localhost:3000/aa/bb" %>      -> 寫死的路徑
```

用環境變數改寫剛剛寫死的路徑   
   
這個是測試環境檔案 
```md
> .env.development檔案  

> hostURL=http://localhost:3000
> <%= link_to "rr", "<%= ENV['hostURL'] %>/aa/bb" =%>

> 最後整條路徑是http://localhost:3000/aa/bb
```


上線環境檔案
```md
> .env.production檔案

> hostURL=http://ddnews.com
> <%= link_to "rr", "<%= ENV['hostURL'] >" =%>

> 最後整條路徑是http://ddnews.com/aa/bb
```



如果今天打這個指令，可以進到測試環境，進到測試環境，他就會去讀取config > environment中的不同資料夾(情境)的變數
```shell
$ RAILS_ENV=test rails s            # server是測試環境(Environment: test)
```



#### 2-3-2、好處二：商業機密
我們不應該把金鑰寫在程式碼裡面(商店代碼的key、雲端伺服器的部署key之類的)，我們應該把這些放在環境變數裡面



### 2-4、為什麼不要推上.git

:::tip
剛剛有提到，因為商業機密的關係，會把商業機密放在.env裡面，如果把機密推上去不就讓大家都知道了   
因此我們會改變一下資料放的位置，實務上，會多放做一個.env.sample檔案，並把DEV、TEXT裡面的資料進到sample裡面   
:::



### 2-5、實際操作 - 把測試環境的東西印出來

首先新增三個資料夾
```md
.env                        # 其他    - 看情況進版控
.env.development            # 開發環境 - 不要進版控
.env.test                   # 測試環境 - 不要進版控
```


```md
> .env

> SITE_NAME=HelloWorld(ENV)
> SecretKEY=123123123
```

```md
> env.development

> SITE_NAME=HelloWorld(DEV)
> SecretKEY=123
```

```md
> .env.test

> SITE_NAME=HelloWorld(TEST)
> SecretKEY=123
```

接著開啟console
```shell
$ rails s           # 這個是開發環境
```

接著輸入ENV
```shell
$ ENV               # 輸入後會跑出一大串的環境變數，剛剛開發環境中的SITE_NAME也在裡面
```

把環境變數拿出來
```shell
$ ENV["SITE_NAME"]  # "HelloWorld(DEV)"
```



#### 2-5-1、這三個檔案要不要進版控呢？  
由於我剛剛三個檔案都有放重要的資訊(KEY)，所以我都不想給他們上   
如果不想上版控，可以把檔案放在.gitignore  
```md
> .gitignore檔案

> .env
> .env.development*          # 星號的意思是，後面有相關名稱的全部包含 
> .env.test*                   ex. .env.development.123
```


不過問題來了！！如果我們不推上去，上線的時候不就壞掉了嗎？因此我們要留一點線索拉你專案的人    
多做一個.env.sample檔案，這檔案裡面放沒有進版控的資料，然後別人拉進來的時候，再把自己裡面專案的值改掉，最後再把.env.sample改成.env，就可以使用了   
```md
> .env.sample檔案

> SITE_NAME=your-site-name
> SecretKEY=your-secret-key
```

改好.sample後，最後記得到到README寫上詳細資訊
```md
> copy and rename `.env.sample` to `.env` and set values for your projects
```

:::tip
你的專案區要env檔案，但是git上又不會有，因此我們給一個樣本檔案   
別人拉你專案的時候，只要把sample檔案改成env，就可以用了  
:::

:::tip
有了環境變數後，我們稍候就可以去申請API金鑰，就可以放進這裡面  
:::

3、許願卡購買流程
------

### 3-1、第一步驟：許願卡購買路徑

期望長這樣"/wish_lists/2/buy" (我要買id為2的許願卡)
用post的原因是要把資料傳給金流商，不過這邊用get也可以
```md
> resources :wish_lists do
> 
>   member do
>     patch :like
>     post :buy             # 路徑是buy，動詞是post
>   end
>   
>   resources :comments, shallow: true, only: [:create, :destroy]
> end
```

有了路徑後，就可以把連結寫出來了
```md
> <%= link_to "購買", buy_wish_list_path(wish_list), method: "post" %>
```

接下來可以來寫buy的action，可以先把購買連結送過來的東西印出來
```md
> def buy
>   render html: params
> end
```


### 3-2、第二步驟：訂單的Model

訂單要有個Model，並且有幾個要注意的事情  
1、要有訂單編號，這個編號是一定要的(之後給金流商就是這個)  
2、訂單不會有destroy(不太需要讓訂單可以被刪除)   
3、一定要給一個價格欄位(這邊是amount)，功用是，假設後續更改了商品價格，總價就變了，不過你這筆訂單還是會記錄下先前的總價  


#### 3-2-1、`order <-- Has one --> WishList` 
 - serial: string -> 訂單編號 -> require  
 - amount: decimal-> 固定最後單價結果(下定成交的金額寫下來，以防有人亂改)  
 - quantity: integer, default: 1  
 - state: string (訂單狀態), (default: pending) - 三種狀態(pending, paid, cancelled)    
 - wish_list_id - belongs_to  
 - user_id - belongs_to  
 - paid_at - 付款時間 (如果今天非同步匯款 - ATM(早上下單、下午拿到錢)才需要用) - 不一定要  




|欄位名稱|資料型態|說明|
|:-:|:-:|:-:|
|serial|string|訂單編號|
|amount|decimal|價格|
|quantity|integer|數量, default: 1|
|state|string|訂單狀態, default: "pending"|
|wish_list_id|integer|一筆許願卡有一張訂單|
|user_id|integer|一個使用者有很多張訂單|



:::tip
由於先前忘記幫許願卡新增價錢欄位，所以這邊幫wishlist新增amount欄位    
rails g migration add_amount_to_wish_list  
:::


剛剛整理好Order的欄位，可以來開設model了
```shell
$ rails g model Order serial amount:decimal quantity:integer state wish_list:belongs_to user:belongs_to
```



### 3-3、第三步驟：model關聯填寫

接下來寫一下關聯

```md
> Wishlist Model

> has_one :order 
```

```md
> Order Model

> belongs_to :wish_list
> belongs_to :user
```

```md
> User Model

> has_many :orders
```


```md
> order的schema

> class CreateOrders < ActiveRecord::Migration[6.1]
>   def change
>     create_table :orders do |t|
>       t.string :serial
>       t.decimal :amount
>       t.integer :quantity, default: 1                           -> 預設值為1
>       t.string :state, default: "pending"                       -> 預設值為pending
>       t.belongs_to :wish_list, null: false, foreign_key: true   -> wish_list_id
>       t.belongs_to :user, null: false, foreign_key: true        -> user_id
> 
>       t.timestamps
>     end
>   end
> end
```
***


### 3-4、第四步驟：購物流程

流程是先選要購買哪個，然後再挑選想要購買的數量，因此把method從post改成get  
Ps. 為什麼是get呢？因為post通常是用在表單傳送，這邊用連結把資料傳給下一個網頁就可以  
     
:::tip  
購買流程為：先選要買哪張卡片，把資料用get方式傳給下一個頁面，下一個頁面要選擇購買的數量   
:::  

```md
> 路徑設定

> resources :wish_lists do
> 
>   member do
>     patch :like
>     get :buy             # 改為get
>   end
>   
>   resources :comments, shallow: true, only: [:create, :destroy]
> end
```

購買按鈕的連結也要記得修改喔！剛剛我們有多一行method: "post"
```md
> <%= link_to "購買", buy_wish_list_path(wish_list) %>      # 把post拿掉
```

這時後我們看一下，購買按鈕點下去，用params抓會拿到哪些東西
```md
> def buy
>   render html: params
> end

> 印出 {"controller"=>"wish_list", "action"=>"buy", "id"=>"1"}  => 這個id就是許願卡的id
```




### 3-5、第五步驟：增/減許願卡數量頁面建立

再來我們來在/wish_list:id/buy，這個頁面，增加可以增加數量、減少數量的輸入欄位(簡易的購物車概念)  

前情提要，@wish_list是什麼，就是許願卡的id
Ps. 記得在before action那邊先掛上去才能用喔！
```md
> wish_lists_controller.rb

> @wish_list = current_user.wish_lists.find(params[:id])
```


因此到buy.html.erb，把許願卡資料填上去
```md
> buy.html.erb

> <h2><%= @wish_list.title %></h2>                    # 許願卡名稱
> 價格：<%= @wish_list.amount %>                       # 許願卡價錢

> <%= form_with url: "#" do |form| %>                 # 表單不用model，只要給url就好
>   <button>-</button>
>   <input type="number" name="quantity" value="1" min="1" max="10">
>   <button>+</button>
>   
>   <button>結帳</button>
> <% end %>
```

:::tip
button被form_with包住的時候，就已經被預設會post出去了  
所以在使用的時候，記得把預設行為關掉喔！  
也就是e.preventDefault()  
:::



#### 3-5-1、掛stimulus controller
基本架構寫好之後，我們要來寫stimulus了，首先新增order_quantity_controller.js的資料夾，因為檔案是剛剛複製前面的，要記得復原到最初的狀態，然後connect那邊先隨便打一個log，等等要來看有沒有掛成功  
```md
> order_quantity_controller.js

> import { Controller } from "stimulus"
> 
> export default class extends Controller {
>   static targets = [ ]
> 
>   initialize() {
>   }
> 
>   connect() {
>     console.log("123");
>   }
> }
```


寫好controller，我們來掛在form身上，這邊可以使用data:{}方法喔！還記得我們之前在link_to的刪除連結上做的事情嗎？之前刪除是加data-confirm上面   
```md
> buy.html.erb

> ...上方省略
> 
> <%= form_with(url: "#", data: {controller: "order-quantity"}) do |form|  %>   # 掛controller上去
>   <button>-</button>
>   <input type="number" name="quantity" value="1" min="1" max="10">
>   <button>+</button>
> 
>   <button>結帳</button>
> <% end %>
```

:::tip
stimulus 掛上去的時候，就會觸發initialize   
controller 掛上去，會觸發connect    
:::



form掛好controller後，接下來處裡action，按下增加或減少按鈕，就會變動target的值，所以幫button掛上data-action  
```md
> buy.html.erb

> ...上方省略
> 
> <%= form_with(url: "#", data: {controller: "order-quantity"}) do |form|  %>   # 掛controller上去
>   <button data-action="click->order-quantity#decrement">-</button>
>   <input type="number" name="quantity" value="1" min="1" max="10" data-order-quantity-target="quantity">
>   <button data-action="click->order-quantity#increment">+</button>
> 
>   <button>結帳</button>
> <% end %>
```

:::tip **描述一次stimulus寫的邏輯**  
首先先想之前用querySelector怎麼抓物件，抓完後用addEventListener、click去做事件  
現在直接用data-action、target的組合技就可以寫完之前那一大串的JS寫法  
   
第一、對想要click後，做動作的元素，對他下data-action，後面的名稱的方式為，"click->該controller#函式"     
後面那個函式就等於寫在click裡面的函式邏輯，簡單說就是點完此元素，想要做的事  
     
第二、對指定的目標下target，就等於之前你點擊某個按鈕後，想要改變他的文字之類的事情，而他的命名邏輯  
data-controller-target="自己取名"，data、target都是固定寫法  
:::



#### 3-5-2、停止預設行為
再來測試是否有掛成功並掛上停止預設行為
```md
> order_quantity_controller.js

> import { Controller } from "stimulus"
> 
> export default class extends Controller {
>   static targets = [ "quantity" ]             # 掛上target
> 
>   initialize() {
>   }
> 
>   connect() {
>     console.log("123");
>   }
>
>   decrement(e) {
>     e.preventDefault()
>     console.log("-")              # 按下按鈕會印出 -
>   }
>
>   increment(e) {
>     e.preventDefault()
>     console.log("+")              # 按下按鈕會印出 +
>   }> 
> }
```



#### 3-5-3、產品數量控制
確認做到基本的數量控制，用this抓取目標target，取得input欄位的值後，就可以用剛剛設定的button來修改值  
下面寫上基本的增加數量、減少數量的邏輯判斷  
```md
> import { Controller } from "stimulus"
> 
> export default class extends Controller {
>   static targets = [ "quantity" ]
> 
>   initialize() {  }
> 
>   connect() {
>   }
> 
>   decrement(e) {
>     e.preventDefault()
> 
>     const q = +this.quantityTarget.value        # q = 輸入欄位的值
>     if (q > 1) {                                # 假設 q > 1，才能抓到欄位數量-1
>       this.quantityTarget.value = q - 1
>     }
> 
>   }
> 
>   increment(e) {
>     e.preventDefault()
>     const q = +this.quantityTarget.value        # q = 輸入欄位的值
>     this.quantityTarget.value = q + 1 
> 
>    }
> }
```




#### 3-5-4、用MAX的值來做數量限制

剛剛那樣寫有點死，如果今天想要改數量限制，那不就HTML那邊要改，JS這邊也要改，所以我們可以利用this.quantityTarget的特性，這樣輸入可以把input整個抓出來，抓出來後我們在取用他的min、max屬性，不就可以達到JS、HTML同步了嗎！！！  
   
因此我們來修改一下，把剛剛寫的1，換成target的寫法
```md
> order_quantity_controller.js

> import { Controller } from "stimulus"
> 
> export default class extends Controller {
>   static targets = [ "quantity" ]
> 
>   initialize() {  }
> 
>   connect() {
>     this.min = +this.quantityTarget.min        # 我們用this帶實體變數，並且帶HTML的min值給他
>     this.max = +this.quantityTarget.max        # 我們用this帶實體變數，並且帶HTML的max值給他
>       
>   }
> 
>   decrement(e) {
>     e.preventDefault()
> 
>     const q = +this.quantityTarget.value       
>     if (q > this.min) {                     # 這個改成min的屬性              
>       this.quantityTarget.value = q - 1
>     }
> 
>   }
> 
>   increment(e) {
>     e.preventDefault()
>     const q = +this.quantityTarget.value     
>     if (q < this.max) {                        # 這個改成max的屬性
>       this.quantityTarget.value = q + 1 
>     }
> 
>    }
> }
```

:::tip
這裡可能有人會對connect()那邊的this.min有疑問  
這邊再提一下，stimulus的特性是，可以用this攜帶狀態，來改變某個物件的狀態  
白話文就是，如果今天想要改變某個物件的true/false，就用this就對了  
而this後面那個變數不重要，你隨便寫名稱都可以  
:::



#### 3-5-5、用資料庫的數值，來取代HTML值

剛剛我們min、max都是直接寫在input上的，那我們不就可以把實際資料庫裡面的存量，寫在這邊嗎！？因為HTML可以接收ruby，然後又可以把值傳給JS，這樣我們不就可以完美的結合資料庫的數量、前端的實際限制數量嗎！！  

Ps. 不過因為我們現在沒有開這個欄位喔～這個只是小提醒可以這樣寫   
```md
> buy.html.erb

> 省略...

>   <input type="number" name="quantity" value="1" min="1" max="<%= @wish_list.剩餘數量 %>" data-order-quantity-target="quantity">

> 省略...
```





#### 3-5-6、金額的小計(total) 

我希望達成的效果是，今天有許願卡的單價，並且我可以調整數量，並且單價 x 數量，並把結果顯示在小計這個地方
```md
> buy.html.erb

> <h2><%= @wish_list.title %></h2>
> 
> <div>
>   單價：<%= @wish_list.amount %>
>   小計：<span>10</span>               多寫一個小計區塊
> </div>
> 
> ...省略
```



又到寫stimulus的流程了，首先要想的是，我想操作什麼！我們來拆解流程，現在我要把先抓到order-quantity數量欄位的結果，並乘上單價欄位的數字，最後顯示在小計的區塊，剛剛數量已經寫過controller了，先想單價怎麼抓  
  
1. 多掛一個controller在單價、小計的div身上，這樣我們等等就可以操作單價、小計區塊  
2. 用data-price的方式，取得許願卡的單價(為什麼這樣可以取得單價呢？下面會提到)  
3. 接收到order_quantity_controller的值，並跟單價的controller相乘(做互動)  
3. 把結果顯示在小計上  

Ps. 記得我們一開始是把order_quantity掛載form身上喔，不能直接在超出他階層的地方掛其他target    


剛剛寫了四件事情，最難的應該就是第三點，要怎麼做到跨controller互動！？不過那等等在想，我們要先新增一個controller在div身上：取名為order_amount_controller.js  

因此這邊先新增一個controller.js
```md
> order_amount_controller.js

> controller初始化，跟前面controller初始化一樣，就不寫了
```

再來幫小計區塊掛上controller
```md
> buy.html.erb

> 價格：<%= @wish_list.amount %>
> <div data-controller="order-amount">
>   小計：<%= @wish_list.amount %>
> </div>
```



利用data-price的寫法，可以傳遞目前許願卡的單價，這邊再說明一次為啥這樣就可以取得單價，還記得JS那邊有一個特殊用法是this.element.dataset嗎？忘記的可以到JS那邊印出來試試，簡單說他可以印出你掛controller的元素，像我現在掛在div上，他就會印出div裡面所有的元素  
如果再加上dataset的用法，他可以印出data裡面所有的屬性，這樣我們就可以取用到許願卡價錢了  
```md
> <div data-controller="order-amount" data-price="<%= @wish_list.amount %>">
>   單價：<span data-order-amount-target="price"><%= @wish_list.amount %></span>
>   小計：<span data-order-amount-target="total"></span>     # 這時候順便掛上target
> </div>
```

Ps. 我上面順便幫單價、小計掛上target了，我猜有人又忘了target可以幹嘛，簡單說他可以印出該target的值，以這個為例
```md
> console.log(this.priceTarget)           # 印出<span data-order-amount-target="price">300</span>

> 還有剛剛input為例
> console.log(this.quantityTarget.max)    # 10
```



不過我們這邊縮減一下，我只想寫一個target，因為變動的只有total，price一直都是固定的，所以我掛一個target在小計上面就好
```md
> <div data-controller="order-amount" data-price="<%= @wish_list.amount %>">
>   單價：<%= @wish_list.amount %>
>   小計：<span data-order-amount-target="amount"></span>     # 只給這個target
> </div>
```
   
再來我們在JS那邊處理小計顯示問題，我們剛剛用data-price抓到單價的值，因此直接把小計的值先填上去(跟單價一樣)   
```md
> order-amount-controller.js

> import { Controller } from "stimulus"
> 
> export default class extends Controller {
>   static targets = [ "amount" ]
> 
>   initialize() { }
> 
>   connect() {
>     this.amountTarget.textContent = this.element.dataset.price    # 給小計區塊價錢
>   }
> }
```


### 3-6、第六步驟：dispatchEvent 兩個controller互相溝通

:::tip
我們現在有兩個controller，一個是價錢的controller，一個是數量的controller   
接下來要把數量的值傳給價錢的controller，接著單價X數量，最後就是小計，因此這邊重點就是兩個controller的溝通。  
:::

:::tip
dispatchEvent文件寫法：  
一個controller發送事件  
一個controller接收事件  
:::
  

#### 3-6-1、發送事件的controller  
1、CustomEvent為JS內建    
2、update-map隨便你寫  
3、window層級，並把event發送出去(發送update-map)  

```md  
> const event = new CustomEvent("update-map")
> window.dispatchEvent(event)
```

#### 3-6-2、接收事件的controller  
1、原本的click，行為變成了update-map  
2、層級是window  
3、map是controller  
4、update一樣是action  

```md
> data-action="update-map@window->map#update"
```


#### 3-6-3、把數量傳出去
看官方文件如果還是不太懂，我們自己來實際寫一次好了，直接用剛剛的小計來做測試，首先我們要先把quantity的值傳出去，因此先到order_quantity，由於範圍太大，我只先寫increment的部分
```md
> order_quantity_controller.js

> increment(e) {
>   e.preventDefault()
>   省略...   
>
>   const evt = new CustomEvent("sendValue")        # 小括號的值可以亂取
>   window.dispatchEvent(evt)                       # window層級把sendValue傳出去
> }
```


#### 3-6-4、價錢controller收到剛剛傳來的數量
傳出去後要來接收了，所以我們要到HTML頁面多設一個接收的data-action的屬性  
1、sendValue是剛剛傳播出來的  
2、@window是剛剛有設定window層級  
3、後面兩個就是傳給的controller、等等要執行的action  

```md
> <div data-controller="order-amount" 
>      data-price="<%= @wish_list.amount %>"
>      data-action="sendValue@window->order-amount#ddd">      # 多加這一行
>   單價：<%= @wish_list.amount %>
>   小計：<span data-order-amount-target="amount"></span>   
> </div>
```



寫好後來實驗一下，有沒有接到傳過來的值
```md
> order-amount-controller.js

>   省略...
>   connect() {
>     this.amountTarget.textContent = this.element.dataset.price    # 給小計區塊價錢
>   }
>   
>   ddd(e) {
>     console.log("234")            # 有傳成功會收到"234"
>     console.log(e)                # 如果你印e，會印出你剛剛傳過來的那個CustomEvent
>   }
> }
```



既然接收到了，就下來我們要來傳order-quantity的數量到order-amount了，這邊要提到另外一個東西，就是CustomEvent的detail，他是固定寫法，可以，在CustomEvent裡面包一個{}，並在裡面設定一些值，這樣就可以透過廣播一起傳過去  

**如果看不太懂，舉個例子，先看事件發送**  
```md
> order-quantity

> const evt = new CustomEvent("ccc", {
>   detail: {name: 123, age: 18}                # 我傳送的時候，順便帶了這兩個東西過去
> })
> window.dispatchEvent(evt)
```


**事件接收 - order-amount**
```md
> order-amount

>   gggg({detail}) {
>     console.log(detail);                    # {name: 123, age: 18}
>   }                                           這樣我接收的時候就會順便帶值進來了
```



11/24 15:30 筆記還沒做，之後記得回來做 
解釋為啥同一層級的controller不能聽到





15:45
實際操作兩個controller - 抽象化
並用a的controller去改變b controller的值







15:53
addEventListener、patchListener是一對的

當瀏覽器按下去的時候，瀏覽器會dispatch那個click事件出來
addEventListener的運作流程就是這樣來的






dispatch更簡化的寫法 - 先別記，可以先參考
[dispatch教學]
就好

16:00
原寫法
```md
    this.quantityTarget.value = n
    const evt = new CustomEvent("ccc", {
      detail: {quantity: n}
    })
    window.dispatchEvent(evt)
```






### 3-7、第七步驟：點下結帳後，要傳送的資訊
數量選擇好後，要送哪些資料出去，終於要來處理form_with的路徑了

:::tip
要傳數量、卡片的id！其他都資料都不重要，因為別人都可以用f12更改  
:::

所以我們要來決定網址了，用get的原因，是因為用post送的話，重新整理會遇到問你要不要再送一次
```md
> 網址：GET .wish_list/2/checkout/?q=2
```

新增路徑
```md
> resources :wish_lists do
> 
>   member do
>     patch :like
>     get :buy
>     get :checkout                 # 新增這一條
>   end
> 
>   resources :comments, shallow: true, only: [:create, :destroy]
> end
```

#### 3-7-1、form 表單路徑
新增好路徑就可以把表單的路徑補上了，增加form_with的url，記得要改成get喔，不過這邊會有一個很特別的地方，因為下面那些欄位，只有input有name，所以送出去後，仔細看網址，點下送出後，網址會變成這個
```md
> /wish_lists/3/checkout?quantity=3(代表我input那邊選擇數量3)  
```

```md
> buy.html.erb

> <%= form_with(url: checkout_wish_list_path(@wish_list), data: {controller: "order-quantity"}, method: "get") do |form|  %>
>   <button data-action="click->order-quantity#decrement">-</button>
>   <input data-order-quantity-target="quantity" type="number" name="quantity" value="1" min="1" max="10">
>   <button data-action="click->order-quantity#increment">+</button>

>   <button >結帳</button>
> <% end %>
```

這樣寫的話，就可以達到拿取許願卡id、數量為2的目的了，而且還不會有post重整頁面的問題！！送出去後就可以來寫checkout了，先來看一下收到了什麼  
```md
> wish_list.controller

> def checkout
>   render html: params  # {"quantity"=>"2", "controller"=>"wish_list", "action"=>"checkout", "id"=>"1"}
> end
```

### 3-8、建立訂單
初步確定抓到的東西後，就可以來寫完整的checkout(建立訂單)  
1、使用current_user建立訂單，這樣就可以把user_id帶進去  
2、amount => 許願卡單價 x 剛剛抓到的quantity  
3、quantity => params有抓到  
4、wish_list => 許願卡id => 能抓到這個是因為我們用表單那邊塞在隱藏欄位的喔！！沒有塞你會根本抓不到他，要記得

```md
> wish_list.controller

> def checkout
>   // 建立訂單
>
>   quantity = params[:quantity].to_i           # 許願卡數量
>   amount = @wish_list.amount * quantity       # 總價
>   order = current_user.orders.new(amount: amount , quantity: quantity, wish_list: @wish_list)
>   
>   if order.save
>     render html: "ok"
>   else
>     render html: "gg"
>   end
> end
```




### 3-9、老師建議order新流程
雖然上面那樣就建立定沒問題，不過老師建議多加一個流程，如果用原本的流程，buy完直接到checkout，一進來會建立第一張訂單，重新整理會再建立一張訂單(因為post的特性)

所以新的流程是這樣   
```md
>                    在這邊建立訂單                       這邊會有訂單編號，因為前面建好了
>                    建立完馬上就走                       這邊會有刷卡單
> buy頁面 -------->  order(跟create一樣沒有頁面) --------> checkout頁面
>          post                                 get
```




#### 3-9-1、order新流程的路徑設定

```md
> /order/訂單編號/checkout

> //Post /orders
> resources :orders, only: [:create] do
>
>   member do
>
>     // GET/orders/訂單編號/checkout
>     get :checkout
>
>   end
>
> end
```

:::tip
訂單頁面的建立用post比較適合  
:::




#### 3-9-2、form_with表單路徑改寫
路徑建好後，我們來改寫一下表單的路徑，這邊有一個特別的地方，就是我們這樣改後，表單就變成create訂單了，建立訂單的動詞為post，url的路徑要改為剛剛新改的order-path    

不過這邊有一個很重要的東西，就是沒有許願卡的id，那要怎麼新增呢？就跟剛剛的quantity一樣，我們用input的name、value帶給他，所以像下面這樣設定就可以
```md
> <%= form_with(url: orders_path, data: {controller: "order-quantity"}, method: "POST") do |form|  %>

>   <input type="hidden" name="wish_list_id", value="<%= @wish_list.id %>">     -> 這邊帶值進去

>   <button data-action="click->order-quantity#decrement">-</button>
>   <input data-order-quantity-target="quantity" type="number" name="quantity" value="1" min="1" max="10">
>   <button data-action="click->order-quantity#increment">+</button>
> 
>   <button >結帳</button>
> <% end %>
```





#### 3-9-3、orders controller
接下來把剛剛新創路徑的controller補完，預期要有create的action
寫好我們先把東西印出來看看
```md
> class OrdersController < ApplicationController
>   before_action :authenticate_user!      
> 
>   def create
>     render html: params       # 會發現裡面有一大包東西，包含了wish_list_id=>1、quantity=>1
>   end                           有了這兩個我們就可以來拿卡片、計算訂單了
> 
> end
```


確認抓到的東西後，可以把create補完了，這邊的流程跟剛剛建立訂單要的東西一樣，這邊要特別提的就是，如果訂單建立成功，就會直接導到checkout那邊(剛開始規劃的路徑就是，建立完直接踢走)，失敗的話就回到購買許願卡的地方
```md
> class OrdersController < ApplicationController
>   before_action :authenticate_user!
>   before_action :find_wish_list, only: [:create]
> 
>   def create
>     # render html: params
>     quantity = params[:quantity].to_i
>     amount = @wish_list.amount * quantity
>     order = current_user.orders.new(amount: amount , quantity: quantity, wish_list: @wish_list)
> 
>     if order.save 
>       redirect_to checkout_order_path(order)
>     else
>       redirect_to buy_wish_list_path(@wish_list), alert: "訂單成立失敗"
>     end
> 
>     
>   end
> 
> 
>   def checkout
>     render html: params
>   end
> 
>   private
>   def find_wish_list
>     @wish_list = WishList.find(params[:wish_list_id])
>   end
> end
```


#### 3-9-4、訂單的serial
不過這樣寫完會失敗，原因就是因為我們的訂單編號是必填欄位，一定要帶給他，所以這邊我們直接在model寫
一個Order新方法，讓訂單可以產生編號
```md
> class Order < ApplicationRecord
>   belongs_to :wish_list
>   belongs_to :user
> 
>   =======下面是新增的=======
>
>   validates :serial, presence: true, uniqueness: true   #訂單要是唯一值
> 
>   before_validation :generate_serial                    #一定要用before_validation，因為callback的關係
> 
>   def generate_serial                                   #給Order一個新方法
>     self.serial = SecureRandom.alphanumeric(12)         #隨機產生一堆數值
>   end
> end
```



產生好後還有一個東西要改，就是目前我們的連結，有帶訂單的編號，這樣被人看到了有點醜，我們直接用剛剛的新方法帶進去
```md
>  def create

>    quantity = params[:quantity].to_i
>    amount = @wish_list.amount * quantity
>    order = current_user.orders.new(amount: amount , quantity: quantity, wish_list: @wish_list)
>
>    if order.save 
>      redirect_to checkout_order_path(id: order.serial)
>    else
>      redirect_to buy_wish_list_path(@wish_list), alert: "訂單成立失敗"
>    end
>
>    
>  end
```



#### 3-9-5、新增完成訂單頁面
最後再來處理checkout的畫面了，新增一個checkout.erb.html



[https://www.refactoredtelegram.net/2021/03/communication-among-stimulus-controllers-part-2/]
