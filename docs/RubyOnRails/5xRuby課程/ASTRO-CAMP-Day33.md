---
title: ROR/Ruby
sidebar_label: "DAY33. [ASTROCamp] Ruby-11"
description: Ruby
last_update:
  date: 2022-11-25
keywords:
  - 5xRuby
  - Ruby
sidebar_position: 34
---



:::tip
**前情提要**  
最後再來處理checkout的畫面了，新增一個checkout.erb.html  
我們準備要把刷卡機放在這個畫面    
:::



1、Paypal金流練習
------

[braintree官網](https://developer.paypal.com/braintree/docs/start/overview)  
先到上面網址註冊一個帳號，記得要註冊sandbox的，他是給你測試用的，註冊好後去信箱收信，從連結登入就可以   


:::tip
**SDK是什麼 - 軟體開發套件(不單指一個)**  
串金流需要用到的是客戶端和server端的SDK  
Client SDKs - for Android, iOS, and JavaScript  
Server SDKs - for Java, .NET, Node.js, PHP, Python, and Ruby  
:::



### 1-1、BrainTree刷卡的流程圖


![流程圖](./img/payment.png)

1. 一開始，用戶使用手機或電腦，打開你的網站的時候，他們會給我們一個request，我們會給他一個response(以此次做的網頁為例，他們看到的網頁連結就是"/orders/訂單編號/checkout")  
2. 在用戶瀏覽此網頁的時候，Server經過驗算，會偷偷塞一個token給用戶(不是rails給的喔)  
3. 當使用者輸入信用卡，按下送出之後，他不會馬上送回我們的Server，會先送到BrainServer，再來BrainServer會給使用者一個Nonce  
4. 用戶收到Nonce(隨機數)後，會把Nonce送到我們的Server這邊  
5. 最後我們拿著使用者給我的Nonce，去跟BrainServer做交易  


:::tip **nonce是什麼？**
他就是只能用一次的隨機數字   
英文縮寫是number only use once  
:::

:::danger
正常來說網站不會存使用者的完整信用卡卡號  
如果在台灣如果你要在你的server存取信用卡卡號，要符合pci compliance的規定  
:::



### 1-2、Set Up Your Client

來處理JS前端的部分，首先增加Drop-in到你的頁面
```md
> <div id="dropin-container"></div>
```

設好Drop-in後，並且用promises的方式來抓他
```md
> braintree.dropin.create({
>   container: document.getElementById('dropin-container'),   # container放上面那個div區塊
>  }).then((dropinInstance) => {                              # 猜測這個是上面拋下來的東西
>  }).catch((error) => {});
```



### 1-3、Get a client token

JavaScript SDK需要一個由您的BrainTree服務器所產生的Client Token，一旦你產生了客戶端Token，請你放到範例中authorization的位置  
```md
braintree.dropin.create({

  authorization: CLIENT_TOKEN_FROM_SERVER,          # 這個就是客戶端token
  container: '#dropin-container'                    # 這個是剛剛的dropin-container
}).then((dropinInstance) => {

}).catch((error) => {});
```




該換我們操作了，在checkout建立一個dropin，等等要來操作他
```md
> checkout.html.erb
> 
> <h2>結帳</h2>
>   
> <div id="dropin" data-controller="paypal-dropin"></div>       # 增加dropin
```


並建立新的paypal_dropin_controller，這個controller可以操作剛剛那個div，首先把基本的建立好，並隨便印出一個數讓他可以動  
```md
> import { Controller } from "stimulus"
> export default class extends Controller {
>   static targets = [  ]
> 
>   initialize() {  }
>   connect() {
>     console.log("123");
>   }
> }
```


### 1-4、BrainTree的npm/yarn
我們來使用npm的方法來抓金流，先不要用cdn的方法
[BrainTree的npm](https://www.npmjs.com/package/braintree-web-drop-in)
```md
> yarn add BrainTree-web-drop-in
```

輸入完指令後，先出找到module檔案裡，BrainTree的名字，接著import先隨便試，因為不知道對方怎麼export的，所以那邊名字隨便打都可以，如果這樣不行的話，再加上大括號試試(這種引入猜測很常用到喔！！)

```md
> paypal_dropin.controller

> import { Controller } from "stimulus"
> import dropin from "braintree-web-drop-in"     # braintree是隨便打的，猜測他們有default export
> 
> 
> export default class extends Controller {
>   static targets = [  ]
> 
>   initialize() {  }
>   connect() {
>     console.log("123");
>     console.log(dropin);               # 這樣印就可以印出braintree那一包
>   }                                      仔細看會發現裡面會有剛剛看到的create
> 
> }
```

確定有掛上controller後，可以開始寫dropin了，這邊只寫connect()，上下都省略  
```md
> paypal_dropin.controller

> connect() {
>  drop.create({
>    container: this.element,           # 剛剛文件是是用getElement，我們直接用this.element就抓到了
>    authorization: "123"               # 這個先隨便給，目前一定會錯，之後再填上
>  })   
>  .then((instance) => {
>    console.log(instance)   
>  })
>  .catch((err) => {
>    console.log(err)  
>  })
> }
```


### 1-5、產生authorization token

要產出這出這個token，需要三個金鑰，這個金鑰金流商會給你，下面是示範用，所以我直接列出來，等等我們就可以用三個key轉出一大串的亂碼
```md
> 下面這一串是金流商直接提供給我們的商店key，等等就可以用這個產出token

> gateway = Braintree::Gateway.new(
>   :environment => :sandbox,
>   :merchant_id => 'qd67XXXXXXXx63',
>   :public_key => '8d6XXXXXXXX32t',
>   :private_key => '8fd0051cXXXXXXXXXX64339cb',
> )
```



### 1-6、Send payment method nonce to server

BrainTree透過收集客戶端傳來的信用卡資訊，把這些東西轉成nonce(只能用一次的隨機數)，接下來下面是一個form表單，為啥突然變表單呢？剛剛不是只是一個div嗎？原因就是轉成nonce後，要再送到自己的網站，之後再從你的網站傳給金流商做比對     
  
**所以接下來的form，就是在做轉成nonce後要再送到自己的網站**  
```md
> 文件資訊

> <form id="payment-form" action="/route/on/your/server" method="post">     # 送出後轉nonce再傳給自己
>   <!-- Putting the empty container you plan to pass to
>     `braintree.dropin.create` inside a form will make layout and flow
>     easier to manage -->
>   <div id="dropin-container"></div>       # 剛剛的controller在這
>   <input type="submit" />
>   <input type="hidden" id="nonce" name="payment_method_nonce"/>
> </form>
```

文件資訊，範例建好表單，再操作controller，這邊的行為在流程圖就是3、4的那幾條線   
```md
> const form = document.getElementById('payment-form');   # 抓到上面的form
> 
> braintree.dropin.create({
>   authorization: 'CLIENT_AUTHORIZATION',
>   container: '#dropin-container'
> }).then((dropinInstance) => {
>   form.addEventListener('submit', (event) => {          # 幫表單加監聽器
>     event.preventDefault();                             # 把表單預設行為停下來
> 
>     dropinInstance.requestPaymentMethod().then((payload) => {    # 抓一包東西回來(nonce)
>       document.getElementById('nonce').value = payload.nonce;    # 把剛剛拿到的東西，塞進上面隱藏的input
>       form.submit();                                             # 表單送出
>     }).catch((error) => { throw error; });
>   });
> }).catch((error) => {
>   // handle errors
> });
```
Ps. 上面有一段requestPaymentMethod，看不懂沒關係，他就是把nonce傳給我們  

:::danger
剛剛要把表單預設行為停下來的原因，就是因為我們要先得到nonce，再手動提交，要不然拿不到nonce  
:::

### 1-7、Set Up Your Server


接下來我們要來處理Server端了，首先要處理的就是前面說的(2)，要把token塞給使用者，不過實作前，我們先載入braintree提供的gem   
```md
> gem "braintree", "~> 4.9.0"
```

重新bundle
```md
> bundle
```

安裝好後開始實作拉，我們到結帳頁面(/orders/訂單編號/checkout)，還記得剛剛說的嗎？客戶到這個頁面的時候，我們要偷偷塞一個token給他，這個行為要在哪裡做呢？就是在order controller做
```md
> orders_controller.rb

> private
> def gateway                                     # 把系統商給的方法，貼到private這邊
>   gateway = Braintree::Gateway.new(               並把自己商店的key填上
>     :environment => :sandbox,                     等等呼叫gateway會回傳下面這些東西
>     :merchant_id => 'qd67XXXXXXXx63',
>     :public_key => '8d6XXXXXXXX32t',
>     :private_key => '8fd0051cXXXXXXXXXX64339cb',
>   )
> end
```


### 1-8、Generate a client token
  
文件說明：照著他寫，就可以產生一段token  
(1) gateway是剛剛寫的私有方法  
(2) client_token、generate是他提供的   
(3) customer_id是客戶的id，不過你不帶也沒關係   

```md
> @client_token = gateway.client_token.generate(      # 產生token
>   :customer_id => a_customer_id                     # 可寫可不寫
> )
```

看完文件來寫我們的產生token方法     
```md
> orders_controller.rb

> def checkout
>   @token = gateway.client_token.generate          # 這個都是照個手冊寫的
> end

> private
> def gateway                                     # 把系統商給的方法，貼到private這邊
>   gateway = Braintree::Gateway.new(               並把自己商店的key填上
>     :environment => :sandbox,                     等等呼叫gateway會回傳下面這些東西
>     :merchant_id => 'qd67XXXXXXXx63',
>     :public_key => '8d6XXXXXXXX32t',
>     :private_key => '8fd0051cXXXXXXXXXX64339cb',
>   )
> end
```


如果這時候你把這個token印出來，會發現超級長的一串亂碼
```md
> checkout.html.erb
> 
> <%= @token %>                 # 超長一段亂碼
> <h2>結帳</h2>
>   
> <div id="dropin" data-controller="paypal-dropin"></div>       # 增加dropin
```


確定token有產生出來後，我們要來把token傳去JS那邊了，為啥要傳去那邊呢？還記得剛剛有一段authorization嗎？忘了沒關係，等等會看到，不過要怎麼帶過去呢？這時候又要使用data屬性了！使用data屬性就可以把ruby傳給JS
```md
> checkout.html.erb
> 
> <h2>結帳</h2>
>   
> <div id="dropin" data-controller="paypal-dropin" data-token="<%= @token %>"></div>       # 增加dropin
```


這邊設定好就可以到controller那邊抓了，把token塞進剛剛說的authorization後，刷卡機就會跑出來了！！！
```md
> paypal_dropin.controller

> connect() {
>   const token = this.element.dataset.token          # 抓到那個token
>   
>   drop.create({
>     container: this.element,           
>     authorization: token               # 把token塞進去
>   })   
>   .then((instance) => {
>     console.log(instance)   
>   })
>   .catch((err) => {
>     console.log(err)  
>   })
> }
```


:::tip
**測試號碼**  
visa給的測試卡號 第一個是4、後面全都是1(15個1)  
日期可以隨便填  
:::


### 1-9、建立form傳送token，並得到nonce
再來要做一個form給刷卡機，我們要做一個路徑接收剛剛傳過來的東西
```md
> 先想一下路徑： POST  /order/訂單編號/pay

> 因此到routes把路徑加上
>  resources :orders, only: [:create] do 
>    member do
>      get :checkout
>      post :pay                            # 這一條！！！
>    end
>  end
```

路徑做好來做表單拉～  
表單注意事項   
1. id記得要給我們轉成亂數過後的訂單邊到喔  
2. 這個表單不用model   
3. @order是在order controller的checkout那邊傳過來的  
```md
> @order = Order.find_by!(serial: params[:id])
```

完整表單
```md
> checkout.html.erb
> 
> <h2>結帳</h2>
> <%= form_with url: pay_order_path(id: @order.serial) do |end| %>  
>    <div id="dropin" data-controller="paypal-dropin" data-token="<%= @token %>"></div>   
> <% end %>
```

:::tip
再來我們要來停下表單的預設事件了，不過還記得為什麼要停下表單預設事件嗎？原因就是收到nonce的流程是這樣  
(1) 這個form表單會傳出token給點擊送出表單的使用者  
(2) 此時要先停止預設事件，要先到金流商那邊取得nonce  
(3) 取得nonce後，最後再手動submit，這樣我們就可以從使用者(金流商)那邊取得nonce了  
:::



不過目前我們的表單設計，因為我們的controller是掛在input上，所以無法抓到表單，所以為了不用再多創一個controller，我們直接把input的controller拿掉，移到form上面，順便把token帶過去  
```md
> checkout.html.erb
> 
> <h2>結帳</h2>
> <%= form_with url: pay_order_path(id: @order.serial), data: {controller: "paypal-dropin",
>                                                              token: <%= @token %>, } 
>                                                              do |end| %>  
>
> <% end %>
```

不過這樣寫會遇到另外一個問題，因為這樣會噴錯誤說，controller must reference an empty DOM node，所以我們要塞一個DOM給他，就直接塞一個簡單的div就好了  

```md
> paypal_dropin.controller

> connect() {
>   const token = this.element.dataset.token          
>   const div = document.createElement("div")       # 創一個div的區塊
>   this.element.appendChild(div)                   # 把他用appendChild塞進form裡面
>
>   drop.create({
>     container: div,                    # container記得要改成div           
>     authorization: token               # 把token塞進去
    })   
    .then((instance) => {
      console.log(instance)   
    })
    .catch((err) => {
      console.log(err)  
    })
  }
```

照這樣改完之後，就可以讓刷卡機跑出來了！！不過這樣還沒完成，還記得剛剛說的暫停預設行為嗎，現在來寫這一塊

```md
> paypal_dropin.controller

  connect() {
    const token = this.element.dataset.token          
    const div = document.createElement("div")       
    this.element.appendChild(div)                   
 
    drop.create({
      container: div,                    
      authorization: token               
     })   
>    .then((instance) => {                                  # 這一段開始都是跟著文件做的
>      this.element.addEventListener("submit", (e) => { 
>        e.preventDefault()  
>        instance.requestPaymentMethod()
>        .then((preload) => {
>           console.log(preload)                # 印出preload，會發現有一包信用卡相關訊息，nonce就在裡面
>        }).catch((error) => { error })
>      })
>
>    })
>    .catch((err) => {
>      console.log(err)  
>    })
>  }
```


知道nonce在哪之後，就可以把它從preload那一包解構出來了
```md
> paypal_dropin.controller


    drop.create({
      container: div,                    
      authorization: token               
     })   
     .then((instance) => {                                  
       this.element.addEventListener("submit", (e) => { 
         e.preventDefault()  
         instance.requestPaymentMethod()
>        .then(({nonce}) => {
>           this.setNonce(nonce)                                 # 這邊把nonce值帶進去 
>           this.element.submit()                                # 手動把表單提交
>        }).catch((error) => { error })
>      })
 
     })
     .catch((err) => {
       console.log(err)  
     })   
>
>  setNonce(value) {                             # 這裡多做一個fc是因為，我們要多創一個input欄位
>    const el = document.createElement("input")    等等想把nonce塞進來
>    el.setAttribute("type", "hidden")           # 下面三行是在設定input的欄位屬性
>    el.setAttribute("name", "nonce")
>    el.setAttribute("value", value)             # 這一行就是把nonce值塞進來的地方
>    this.element.appendChild(el)                  把輸入欄位塞進form欄位
>  }
```



這樣就可以確定收到nonce了，接下來我們把東西印出來看看！不過還記得我們剛剛form提交後，會去到哪嗎～要記得我們剛剛有多設定pay的路徑喔，所以要到order的controller新設定一個action   

```md
> order_controller.rb

> def pay
>   render html: params         # 我們把form傳來的那一包表單印出來
> end                             會發現裡面有一堆東西，包括nonce、訂單編號
```



### 1-10、Receive a payment method nonce from your client

確定有nonce後，我們看一下文件，他怎麼處理的。他先宣告一個區域變數 = 抓取剛剛放置nonce的input欄位(記得這個input剛剛name是你自己取的喔，不要亂抄文件)
```md
> post "/checkout" do
>   nonce_from_the_client = params[:payment_method_nonce]
> end
```

### 1-11、Create a transaction

再來的文件是，把取得的東西，變成一張訂單  
(1) gateway是剛剛建立的方法  
(2) amount是order的欄位  
(3) payment_method_nonce這個是剛剛拿到的nonce  
(4) 後面兩個都不填沒關係，device那個主要是判斷客戶的裝置  
```md
> result = gateway.transaction.sale(      
>   :amount => "10.00",                               
>   :payment_method_nonce => nonce_from_the_client,
>   :device_data => device_data_from_the_client,
>   :options => {
>     :submit_for_settlement => true
>   }
> )
```

### 1-12、Successful result

確認是否有成功完成訂單，使用這個方法會回傳true or false值
```md
> result.success?
> #=> true
```


### 1-13、正式寫pay action
確認文件好，可以開始寫自己的訂單了
```md
> order_controller.rb

> def pay
>   @order = Order.find_by!(serial: params[:id])        # 用剛剛的訂單編號來抓訂單id
>
>   result = gateway.transaction.sale(      
>     amount = @order.amount,                           # 訂單的價錢    
>     payment_method_nonce = params[:nance],            # 抓到拿到的nonce 
>   )
>
>  if result.success?                                   # 這個方法是官方提供的
>    // 改變訂單狀態
>    redirect_to root_path, notice: "付款成功"
>  else
>    redirect_to root_path, alert: "付款發生問題"
>  end
>
> end
```


最後最後，要記得把金鑰用環境變數藏起來，首先把原本放key的地方都用環境變數包起來
```md
> order_controller.rb

> def gateway
>   gateway = Braintree::Gateway.new(
>     :environment => :sandbox,
>     :merchant_id => ENV["BRAIN_MERCHANT_ID"],
>     :public_key => ENV["BRAIN_PUBLIC_KEY"],
>     :private_key => ENV["BRAIN_PRIVATE_KEY"],
>   )   
> end
```

### 1-14、新增環境變數
再來到.env_development的檔案，把key放上去
```md
> .env_development

> BRAIN_MERCHANT_ID=qd67XXXXXXXx63
> BRAIN_PUBLIC_KEY=8d6yqXXXXXXXX2t
> BRAIN_PRIVATE_KEY=8fd0051cXXXXXXXX4339cb
```

:::tip
設定好環境變數，記得要重新開server一次喔～  
:::


記得推上去前，要建一個.env.sample的檔案，裡面放的是之後給別人改成.env的資料
```md
> .env.sample

> BRAIN_MERCHANT_ID=
> BRAIN_PUBLIC_KEY=
> BRAIN_PRIVATE_KEY=
```

:::tip
也要記得把.env.development放進gitignore喔，先前有放進去過了，不過還是再提醒一次  
:::



2、拉下GitHub更改環境變數示範
------
龍哥試一次抓github的專案
```md
> (1) git clone下來
> (2) bundle
> (3) yarn install
> (4) 把.env.example改成.env
> (5) 把自己商家的key填上env檔案
```



3、改變訂單狀態
------

有限狀態機是什麼！？狀態機的概念是說，打開狀態只能關閉 關閉狀態只打開  

為什麼要引入狀態機？有了狀態機，就可以引入很多狀態，以訂單來說，我們就可以幫她設定pending、cancelled之類的   

:::tip
狀態機的意思就是會引入很多狀態，每個狀態間都會有一個動詞  
狀態機就是一堆的if/else  
:::


### 3-1、狀態機推薦使用AASM

安裝AASM
[AASM GitHub](https://github.com/aasm/aasm)  
```md
> gem 'aasm'
> bundle
```


### 3-2、狀態流程圖
把訂單所有狀態的傳遞都畫出來
```md
>                                     refund(動詞)
>                      refunded  <--------
>                         |               |
>                         | cancel        |
>        cancel(動詞)      V  (動詞)       |
>          ------- >  cancelled           |
>  pending                                |
>          ------- >  paid    ------------ 
>          pay(動詞)
```

上面狀態圖的意思是   
1. pending(待付款)是一個狀態，經由兩個動詞，一個是pay，pay完後會到paid(已付款)的這個狀態  
2. paid這個狀態接後面的動詞為refund，後面的狀態為refunded(退款完成)  
3. refunded(退款完成)後面的動詞為cancel，也就是可以對退款進行取消這個動詞，最後是cancelled(取消)狀態   
4. 回到剛剛第一條，pending也可以經由cancel這個動詞，到cancelled這個狀態  
  

畫完圖後，就來實際寫一次吧！首先到Order的Model，由於原本model還有其他東西，這次只顯示狀態機的書寫   
   
  
### 3-3、狀態機設定
這邊有幾件事情要注意  
1. aasm column: "state"，這句的意思是，你指定Order的state欄位，設定為狀態機，這樣就可以啟用了  
2. 加上no_direct_assignment的意思是，讓訂單的狀態不能被強制更改，比如現在是pending狀態，你直接進資料庫改他狀態  
3. initial的意思是，你要讓整個訂單的初始狀態是什麼  
4. 記得要引入AASM才能用  

```md
> class Order < ApplicationRecord
> 
>   include AASM              # 首先引入AASM
> 
>   aasm column: "state", no_direct_assignment: true do        
>     state :pending, initial: true
>     state :paid, :cancelled, :refunded
> 
>     # 付款動作，是從待付款，到付款
>     event :pay do
>       transitions from: :pending, to: :paid
>     end
>     # 取消動作，是從待付款、退款，到取消付款
>     event :cancel do
>       transitions from: [:pending, :refunded], to: :cancelled
>     end
>     # 退款動作，是從待付款，到退款
>     event :refund do
>       transitions from: :paid, to: :refunded
>     end
>   end
> end
```



照上面這樣設定好後，就可以到console做一些有趣的事情
```md
> rails c --sandbox         # 進沙盒環境試試

> o1 = Order.first          # 先抓第一筆訂單
```

### 3-4、來測試AASM送給你的一些方法
```md
> o1.pending?               # true -> 確認現在是否為"某狀態"
> o1.paid?                  # false

> o1.may_pay?               # true  -> 確定現在狀態下接下來可以做某些事情嗎(做動作)
> o1.may_refund?            # false -> 因為pending狀態下，沒有接到refund動作

> o1.pay!                   # 這樣會直接把pending的狀態改成pay
> o1.refund!                # 報一大堆錯，因為pending沒有接到refund的動作
```

:::danger
如過今天你已經是paid的狀態  
再執行o1.pay!，會直接報錯誤給你喔！！  
:::


### 3-5、完成pay action
這樣設定好後，我們就可以來把剛剛pay的改變訂單狀態寫完了
```md
> order_controller.rb

> def pay
>   order = Order.find_by!(serial: params[:id])     
>
>   if order.may_pay?                                   # 先確定能不能付款   
>      result = gateway.transaction.sale(      
>        amount = @order.amount,                       
>        payment_method_nonce = params[:nance],        
>      )
>      
>     if result.success?                               
>       // 改變訂單狀態
>       order.pay!                                      # 能付款的話，直接把pending改成paid
>       redirect_to root_path, notice: "付款成功"
>     else
>       redirect_to root_path, alert: "付款發生問題"
>     end
>   else
>     redirect_to root_path, alert: "無效訂單"           # 假設不能把狀態改成pay，回傳無效訂單
>   end
> end
```


### 3-6、最後整理一下狀態機

fsm有一個重點，就是“有限“狀態機，不是無限的，無限是啥意思呢！？無限狀態就是指你可以亂設定，如果今天你沒有設定有限狀態機，就像你開手排車，可以直接從1檔打到8檔！  

我們是經由操作他的動作，去改變它的狀態，而不是直接改變它的狀態  
ex. 我是用關門這個動作，來讓打開的門關閉，而不是啥都不做原本打開的門，門就關起來了  

:::tip
有限狀態機不只可以用在model身上，其他情況也是可以用的，只是這裡剛好很適合用到在model上    
:::



4、Eslint
------

這個可以幫助整理你JS的檔案，比如說縮排不好，或是函式太多行之類的，他都會提醒你
```md
> npm install eslint --global
```

接下來就可以執行eslint這個程式
```md
> eslint 123.js             # 後面是檔案名稱
```

不過應該會要你執行eslint的初始設定，他會問你一些問題
```md
> npm init                     # 先生一個package.json檔案出來
> npm init @eslint/config       # 再執行初始化
```

照他的規矩都寫好後，就可以再執行一次
```md
> eslint 123.js             # 後面是檔案名稱
```



5、rubocop
------

:::tip
A Ruby static code analyzer and formatter, based on the community Ruby style guide.  
可以整理你ruby的檔案，並依照你的情況幫你做修改、限制  
:::



### 5-1、安裝rubocop-rails
A RuboCop extension focused on enforcing Rails best practices and coding conventions.  
專門整理rails檔案的插件


接下來我們就實際來安裝看看，把下面這一段放到gemfile  
require: false的意思是，預設先關閉，我們呼叫他再出現就好  
Ps. 放在gemfile的測試區域   
```md
> gemfile

> gem 'rubocop-rails', require: false       
> bundle
```

### 5-2、啟動rubocop
安裝好後可以來啟動他了    
接著它會噴一大串錯誤  
```md
> rubocop           
```

### 5-3、強制修改
可以直接修改一大堆檔案
```md
> rubocop -A            # 強制修改可以改的檔案
```


### 5-4、新增規則
在做專案的時候，先把規矩都訂好，那要把這些規格放哪？    
可以新增一個.rubocop.yml檔案    
[可以觀看這個網頁，有一些規格設定的方式](https://github.com/rubocop/rubocop/blob/master/config/default.yml)  

在檔案裡面新增這個，把預設的style/Document關掉
```md
> Style/Documentation:
>   Enabled: false
```

想忽略某些檔案怎麼辦，這邊直接把schema檔案忽略、bin相關檔案忽略
```md
> Style/Documentation:
>   Enabled: false
> AllCops:
>   Exclude:
>     - '/db/schema.rb'
>     - 'bin/*'

> Metrics/MethodLength:         把最大行數加到20
>   max: 20
```


:::tip
Assignment Branch Condition 這個意思是，你的code寫太複雜了    
ex. 你if判斷裡面又包了一層if  
:::


還可以在yml檔案裡面加上一段，可以把程式給的建議關掉
```md
> AllCops:
>   SuggestExtensions: false
```

最後把新警察加上去
Ps.  把新警察打開，會有其他錯誤跑出來，不過只要把它給解決掉，這個專案的coding style檢查就完成了
```md
> AllCops:
>   NewCops: enable
```




6、RSpec跑測試
------

:::tip
bundle init可以建出一個空的gemfile  
:::

Ruby兩個測試工具比較    
(1) mini-test - ruby內建，比較快    
(2) rspec - 另外安裝，比較慢一點 -> 我們用這個寫    


### 6-1、安裝rspec
首先下這個指令，就可以開始啟用RSpec 
```md
> bundle add rspec
```

測試就是在建立規格，先把規格訂好，再來寫code
建立新的檔案，並開始跑
```md
> bank_spec.rb

> rspec bank_spec.rb    # 這樣就可以跑測試了，不過我們現在啥都沒寫
```


### 6-2、先開好規格

#### 6-2-1、存錢功能
1. 可以存錢     
2. 不可以存0元或是小於0元的金額(越存錢越少)     

#### 6-2-2、領錢功能   
1. 可以領錢     
2. 不能領0元或是小於0元的金額   
3. 不能領超過本身的餘額     


測試規格
```md
> RSpec.describe House do
>   describe "存錢功能" do
>     it "存錢" do
>       # 先把銀號帳號產生出來
>       account = House.new(10)
>       # 我把錢存進去
>       account.deposit(4)
>       # 預期得到14元
>       expect(account.balance).to be 14    
>     end
> 
>     it "不可以存0元或是小於0元的金額" do
>       account = House.new(10)
>       account.deposit(-5)
>       expect(account.balance).to be 10
>     end
> 
>   end
> 
> 
>   describe "領錢功能" do
>     it "領錢" do
>       account = House.new(10)
>       money = account.withdraw(4)
>       expect(money).to be 4
>       expect(account.balance).to be 6
>     end
> 
>     it "不能領0元或是小於0元的金額" do
>       account = House.new(10)
>       money = account.withdraw(-2)
>       expect(money).to be 0
>       expect(account.balance).to be 10      
>     end
> 
>     it "不能領超過本身的餘額" do
>       account = House.new(10)
>       money = account.withdraw(12)
>       expect(money).to be 0
>       expect(account.balance).to be 10
>     end
>   end
> end
```

正式寫
```md
> class House
>   def initialize(money)
>     @money = money
>   end
> 
>   def deposit(money)
>     @money = @money + money if money > 0
>   end
> 
>   def withdraw(money)
>     
>     return 0 if money <= 0 || balance < money    
>     @money = @money - money
>     money
> 
>   end
> 
>   def balance
>     @money
>   end
> end
```


