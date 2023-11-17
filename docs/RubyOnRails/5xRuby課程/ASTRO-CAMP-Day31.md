---
title: ROR/Ruby
sidebar_label: "DAY31. [ASTROCamp] Ruby-9"
description: Ruby
last_update:
  date: 2022-11-22
keywords:
  - 5xRuby
  - Ruby
sidebar_position: 32
---



1、按讚功能
------

:::tip
做按讚功能會比較麻煩，因為要處理 `前後端媒合`  
為什麼會說要做前後端媒合呢  
   
因為當我們點下按鈕時，我們首先要控制按鈕的狀態切換(點下去後DOM要做用，並讓按鈕轉換型態)  
這樣改只是前端，接下來我們要把按鈕被點擊過的狀態傳到後端(已經被點擊的狀態傳到資料庫)  
所以要處理的就是html、JS、ruby的互動過程    
:::


### 1-1、製作流程
那要怎麼處理呢？通常我們在寫rails的時候，html就是我們交換JS、ruby資料的好地方 

(1)、點讚的icon放上去，並用JS寫出可以變換狀態的按鈕   
(2)、我們要用fetch去接住ruby傳來的狀態，因此先來做路徑(這個路徑是點讚後會觸發的路徑)   
(3)、處理fetch，使用HTML傳Ruby值到JS(這個值就是網址上面的id，目前許願卡的id)   
(4)、做一個多對多的model(因為要連結使用者、許願卡)   
(5)、用Ruby傳到fetch的資料，判斷按鈕狀態(如果已經在約診簿裡，代表已按過讚)   
(6)、設定剛進網站按鈕的狀態(如果已經按過讚，要是黑色的)，這個要用HTML來做雙向溝通  
(Ruby傳是否已經在資料庫給HTML，HTML設定data，利用stimulus傳ruby給JS，最後用JS來做按鈕初始狀態)   
(7)、整理、優化CODE  

### 1-2、步驟一：點讚icon

#### 1-2-1、fontawesome教學(可以增加icon)

[fontawesome首頁](https://fontawesome.com/icons)  
[fontawesome - npm指令](https://fontawesome.com/docs/apis/javascript/get-started)  


首先我們來用指令下載fontawesome  
雖然官網是用npm，不過我們來用yarn也可以成功引入  
Ps. 記得yarn的指令是yarn add  

因此我們來輸入這段指令   
```shell
$ yarn add @fortawesome/fontawesome-svg-core
```

另外還有其他的指令要輸入，因為他有很多種不同的字型，有solid、regular.....等等，要引入不同類型，就要先下載一次
```md
> yarn add @fortawesome/free-solid-svg-icons      -> 可以引入solid
> yarn add @fortawesome/free-regular-svg-icons      -> 可以引入regular
```

:::tip
上面輸入的指令，都會下載一份檔案到modules   
並且會更新package.json、yarn.lock   
:::


引入後正式來寫icon，這裡有幾個重點  
(1) 先看這一份文件，https://fontawesome.com/docs/web/dig-deeper/svg-core   
(2) 要使用dom.watch才能引入(新的寫法)  
(3) 第二行裡面的faThumbsUp，是依據你要引入的icon來改寫   
(4) 假設今天要引入兩種同名的icon，但是不同樣式，就要用as幫他另外取名   
(5) 初始化的地方放library.add，裡面要帶入上面icon的名稱  

```md
> import { library, icon, dom} from '@fortawesome/fontawesome-svg-core'               -> 這裡要加上dom喔(新版的寫法)
> import { faThumbsUp as thumbsSolid } from '@fortawesome/free-solid-svg-icons'       -> 這裏引入的東西，要看你引入的icon類型
> import { faThumbsUp sd thumbsRegular } from '@fortawesome/free-regular-svg-icons'   -> 這裏引入的東西，要看你引入的icon類型
> 
> initialize() {
>   library.add(thumbsSolid, thumbsRegular)           # 這個放在初始化的地方，並且記得icon引入的名稱放在這
> }
> connect() {
>   dom.watch()                                       # 正式啟用
> }
```

show頁面的寫法，我們要把icon放在這，選好想要的icon，直接copy官網code就好
```md
> <i class="fa-solid fa-thumbs-up"></i>         -> 這邊fa-solid是前面寫yarn add引入的，
```


幫按鈕加上target，加上target的目的是，等等我們要來操作這個icon
```md
> <i class="fa-solid fa-thumbs-up" data-love-btn-target="icon"></i>
```


完整code，這樣就可以切換按鈕的兩種狀態
```md
> import { Controller } from "stimulus"
> import { library, icon, dom} from '@fortawesome/fontawesome-svg-core'
> import { faThumbsUp as thumbsSolid } from '@fortawesome/free-solid-svg-icons'
> import { faThumbsUp as thumbsRegular } from '@fortawesome/free-regular-svg-icons'
> 
> export default class extends Controller {
>   static targets = [ "text", "icon" ]
> 
>   initialize() {
>     this.btnState = false
>     library.add(thumbsSolid, thumbsRegular)
> 
>   }
>   
>   // 這邊可以讓icon啟用(實際讓他做的事，就是把 i tag，改成svg tag)
>   connect() {
>     dom.watch()    
>   }
>   
>   // 使用classList，幫target切換class(list可以把css塞進陣列裡面)
>   toggle() {
>     if (this.btnState) {
>         this.textTarget.textContent = "NO"
>         this.iconTarget.classList.remove("fa-regular")
>         this.iconTarget.classList.add("fa-solid")
>     } else {
>         this.textTarget.textContent = "YES"
>         this.iconTarget.classList.add("fa-regular")
>         this.iconTarget.classList.remove("fa-solid")
>     }
> 
>     this.btnState = !this.btnState
>   }
> }
```



### 1-3、步驟二：點讚後會傳的路徑

:::tip
不過這樣只做了前端，接下來要做寫到後端(讓後端知道這個按鈕有被按到)      
按一下，會有一個request出去，送出去之後，那我們要怎麼接呢！？  
:::

#### 1-3-1、用fetch接住request
  
不過這邊會有一點點問題，就是我們要用fetch，要先知道fetch的路徑是什麼  

預想中的fetch的路徑
```md
> fetch(...)
> patch /wish_lists/:id/like          -> 按讚送過的網址
> delete /wish_lists/:id/unlike       -> 取消讚送過的網址(但是我們今天只需要上面那一條就可以)
```

多加一條路徑
```md
>  resources :wish_lists do
> 
>    member do
>      patch :like              # 用member包在resources裡面就好
>    end
>
>    resources :comments, shallow: true, only: [:create, :destroy]
>  end
```


有了路徑後，可以用fetch來接收了，method記得要改方法   
Ps. 先把路徑隨便寫，這個預期會失敗，不過先把基本fetch寫好，預期是要跑出錯誤喔！  

```md
> fetch("/wish_lists/100/like", {method: "PATCH"})      # 路徑先隨便寫，要先確定有收到fetch
>     .then(() => {
>     console.log("ok");
>     })
> 
>     .catch((err) => {
>     console.log(err);
>     })
```



### 1-4、步驟三、處理fetch，使用HTML傳Ruby值到JS

接下來處理fetch的路徑，用HTML當作資料交換的地方(JS、ruby)   
在HTML任一元素上塞data，並且用dataset去抓他    
   
先在show.html.erb那邊設定data-id，這邊可以寫ruby，因此把許願卡的id塞過去   
```md
> data-id="<%= @wish_list.id %>"            # 這個東西可以送到JS那邊
```


接著到JS那邊，把用dataset抓住，然後塞正確路徑到fetch裡面    
不過你可能會有疑問，dataset是幹嘛的？？我們把它印出來看看  
```md
> 前情提要，這個this.element，當初我們controller是掛在button上，所以單印this.element會印出按鈕的所有元素
> console.log(this.element);            # <button style=""...........>

> 那我們來印dataset
> console.log(this.element.dataset)     # action: "click->love-btn#toggle"
>                                         controller: "love-btn"
>                                         id: "2"                                        

> 會發現dataset把整個在HTML上的data屬性，全部包起來了，所以假設我要拿剛剛Ruby傳的id，只要.id，就拿得到了
```


把dataset抓到的ID，塞進路徑裡面
```md
> const id = this.element.dataset.id                      # 先拿id
> fetch(`/wish_lists/${id}/like`, {method: "PATCH"})      # 把路徑設定好
>     .then((resp) => {
>       return resp.json()                                
>     })
> 
>     .then((data) =>                                     # 先把路徑傳來的東西，印出來看看
>       console.log(data)
>     )
> 
>     .catch((err) => {
>     console.log(err);
>     })
```


再來到許願卡的controller寫action
```md
> def like
>   render json: {status: "ok"}                          # 我們先傳簡單的Json過來，不能傳HTML喔，會壞掉
> end
```

:::tip
不過這樣會卡住，原因是什麼呢？？原因是authenticate-token錯誤(可以開f12的Network中的preview)   
簡單説就是沒有驗證就想進去資料庫，這樣會被rails擋住   
:::

#### rails提供的meta-token
rails 有提供給我們一個會變動的meta-token，只要我們抓到這個token，之後再塞給fetch，這樣就可以
```md
> const token = document.querySelector("meta[name='csrf-token']")
```

最後完整的fetch，fetch上半部的寫法要看文件，才能知道如何寫
```md
>  fetch(`/wish_lists/${id}/like`, {
>      method: "PATCH",                         -> 他一定要大寫，要記得
>      headers: {
>          "X-CSRF-Token": token                -> 用這個方法把token帶進去
>      },
>  })
>    .then((resp) => {
>      return resp.json()
>    })
>   
>  .then((data) => {
>      console.log(data);
>    })
>  
>    .catch((err) => {
>      console.log(err);
>    })
```


### 1-5、步驟四：model的多對多 - User & WishList

:::tip
多對多會有額外一個約診簿  
假設今天有很多個病人，也有很多位醫生  
  
一位醫生有很多個病人  
一位病人也會看多醫生(假設這個病人身體比較差)  
  
那今天要做到這個關聯，我們要怎麼做呢  
那就需要有額外的記錄簿，來記錄醫生、病人的狀況  

為什麼要提這個呢？？因為我們等等要做一個多對多的表單，也就是一個許願卡可以被很使用者喜歡，而且使用者也很喜歡很多許願卡    
這個第三張表的取名 - 這個表單的取名很重要，如果以後還有另外多對多的表，如果沒取好名，痛苦的是你！！   
:::


第三張表跟使用者和許願卡的關聯
```md
> belongs_to :user_id
> belongs_to :wish_list_id
```
|欄位名稱|資料型態|說明|
|:-:|:-:|:-:|
|user_id|integer|跟User表單做關聯|
|wish_list_id|integer|跟WishList表單做關聯|


取好名，並且把欄位都考慮好，就可以來建立新 model  
```shell
$ rails g model LikeWishList user:belongs_to wish_list:belongs_to
$ rails db:migrate
```


到model寫關聯
```md
> has_many :like_wish_lists
> has_many :wish_lists, through: :like_wish_lists
```


#### 1-5-1、多對多關聯的命名 - 如果跟一對多的關聯相撞如何解決

不過遇到問題了，還記得前面我們有寫過，has_many :wish_lists的關聯嗎！？  
新增加的多對多的關聯命名跟他撞到了，那要怎麼解決呢？我們幫新的表改名就可以了  

```md
> wish_list model

> has_many :wish_lists   -> 這個是前面寫過的連結
> has_many :comments
> 
> has_many :like_wish_lists
> has_many :liked_wish_lists, through: :like_wish_lists, source: :wish_list  -> 幫關聯改名，不改名的話，會跟前面相撞
```
:::info
Ps. source是單數的原因，是因為文件上這樣寫   
:::


順便把has_many :users改掉的原因，因為我們之後寫方法時，會跟user搞混，如果會搞混，那我們就改寫users的名稱，方便我們寫
```md
> user model

> has_many :like_wish_lists
> has_many :liked_users, through: :like_wish_lists, source :user
```


#### 1-5-2、user改方法名
我來舉個例子，首先我們來看wish_list的model有一些關聯
```md
> belongs_to :user              -> 許願卡跟使用者的關聯(這張許願卡是誰寫的)
> has_many :comments
> 
> 
> has_many :like_wish_lists     -> 許願卡跟使用者多對多的關聯(有哪些使用者喜歡這張許願卡)
> has_many :liked_users, through: :like_wish_lists, source: :user
```

用console來實際舉例子
```md
> 先來找出第一張許願卡

> w1 = WishList.first
```

接下來我們先來測試belongs_to :user
```md
> w1.user               # 這一行的意思是，我要找出這張卡是誰寫的
``

接下來我們先來測試has_many :users
Ps. 上面那個是我們換過名字，我們現在先用沒改名過的
```md
> w1.users              # 這一行的意思是，我要找出誰喜歡這張許願卡
```

那問題出來了，如果用users原名，有沒有覺得這樣很容易跟.uesr搞混  
就是這樣我們才要來改名  
 


### 1-6、like action：點讚結果塞進第三張表

like action 的細節 - 第三張表的使用方式、查找方法  
接下來寫like的action，我們要來寫，假設今天有人對許願卡點讚，我們要把這張許願卡寫進第三張表(約診簿)  
那要怎麼做呢？有以下幾點步驟  

(1) 找到許願卡  
(2) 用include判斷，目前許願卡有沒有被按讚  
(3) 從許願卡角度or使用者角度，查找第三張表(約診簿)  
(4) 如果沒有在第三張表，代表沒辦按讚過，我們要把被按讚的許願卡塞進約診簿  
(5) 如果已經存在，我們就把許願卡刪掉(讓讚可以被使用者收回)  


首先我們要找到許願卡 
```md
> 這一段的意思是，喜歡這張許願卡的人，有沒有包括登入者自己
> def find_wish_list
>   @wish_list = current_user.wish_lists.find(params[:id])
> end
```

再來用兩種不同的角度，利用include判斷許願卡是否有被按讚
```md
> 許願卡的角度 -> 喜歡這張許願卡的人，有沒有包括登入者自己
> if @wish_list.liked_user.include?(current_user)

> 使用者的角度 -> 每個使用者(登入者)都有很多喜歡的清單，那清單裡面有沒有包括@wish_list
> if current_user.liked_wish_lists.include?(@wish_list)
```


最後做新增和移除，假設已經在裡面就刪除，反之新增
```md
> if current_user.liked_wish_lists.include?(@wish_list)

>   // 刪除 -> 如果有的話，我們就把它移除
>   current_user.liked_wish_lists.delete(@wish_list)
>   render json: {status: "unliked", id: @wish_list.id}
> else

>   // 新增 -> 如果沒有的話，我們就去新增
>   current_user.liked_wish_lists << @wish_list
>   render json: {status: "liked", id: @wish_list.id}
> end
```

:::tip
**Line斷網傳簡訊的方法**  
如果今天像Line那種斷線狀態，訊息送出正常來說是送不出去的  
但是可以用JS把文字放到訊息框上面，用灰色的字表示  
我們用JS演戲給使用者看，以為有送出去，但是實際上是沒送出去，只是因為我們用JS假裝有送出去   
:::



### 1-7、步驟五：用Ruby傳到JS的資料，判斷按鈕狀態

上面判斷好之後，就可以根據資料庫的狀態，來做fetch了，並且可以依據有無寫進資料庫
來做按鈕的狀態判斷，並依照狀態給按鈕顯示特定特色
```md
>  const token = document.querySelector("meta[name='csrf-token']").content
>  const id = this.element.dataset.id
>  fetch(`/wish_lists/${id}/like`, {
>      method: "PATCH", 
>      headers: {
>          "X-CSRF-Token": token
>      },
>  })
>    .then((resp) => {
>      return resp.json()
>    })
>
>    .then(({status}) => {                          -> 我們依照按鈕資料庫的狀態，來判斷按鈕的狀態
>      if (status === "liked") {
>          this.textTarget.textContent = "NO"
>          this.iconTarget.classList.remove("fa-regular")
>          this.iconTarget.classList.add("fa-solid")
>      } else {
>          this.textTarget.textContent = "YES"
>          this.iconTarget.classList.add("fa-regular")
>          this.iconTarget.classList.remove("fa-solid")
>      }
>    })
>  
>    .catch((err) => {
>      console.log(err);
>    })
> 
>  this.btnState = !this.btnState
```




### 1-8、步驟六：設定剛進網站按鈕的狀態

:::tip
剛剛那樣就可以做到，依照是否有寫進資料庫，來做按讚按鈕的狀態了   
不過這樣有另外一個問題，如果我們今天重新整理畫面，liked的狀態就消失了  
要怎麼讓按鈕的狀態是保持的呢？我們可以使用用HTML來判斷是否有寫進資料庫   
:::

設定剛進網站按鈕的狀態(如果已經按過讚，要是黑色的)，這個要用HTML來做雙向溝通  
(Ruby傳是否已經在資料庫給HTML，HTML設定data，利用stimulus傳ruby給JS，最後用JS來做按鈕初始狀態)  


:::tip
Ruby是沒辦法把狀態塞到JS的    
因此我們要透過HTML -> stimulus  
HTML是JS、ruby做資料交換的地方  
:::

所以首先，第一步就要想到stimulus JS，這個可以在rails上寫JS的小幫手   
(1) 到剛剛把按鈕、圖案掛上去的頁面 - show頁面   
(2) 幫這個按鈕掛上新的data-liked(這樣取名可以幫助我們後面使用)  
(3) 到stimulus controller加上判斷式(當HTML這邊的傳的是"true"，就顯示XX，否則XX)  
  

先來處理show頁面，可以看到下面data-liked那一行，就是我們等等要傳給JS處理的資料  
因為這裡是erb，所以可以寫ruby，這裡寫的ruby等等就可以用JS接收到並處理  

```md
> <button style= "padding: 10px 20px", data-controller="love-btn"
>                                      data-action="click->love-btn#toggle"
>                                      data-id="<%= @wish_list.id %>"
>                                      data-liked="<%= current_user.liked_wish_lists.include?(@wish_list) %>"
>     <span data-love-btn-target="text">no</span>
>     <i class="fa-regular fa-thumbs-up" data-love-btn-target="icon"></i>
>     <%# <i class="fa-regular fa-thumbs-up"></i> %>
> </button>
```

ps. data-liked這一段目前有點醜，他的意思是，假設我今天有登入，並且對該張許願卡點贊，那就會回傳true值
```md
> <%= current_user.liked_wish_lists.include?(@wish_list)
```


再來到stimulus controller處理剛剛HTML傳過來的資料庫狀態(是否已經被點擊過)  

把從HTML傳過的的資料，在connect做判斷  
假設剛剛傳來的資料是true，(true代表這個讚被按過了，因此初始狀態要顯示黑色按鈕，反之顯示白色)  
```md
> connect() {
>   dom.watch()
>   if (this.element.dataset.liked === "true") {
>       this.iconTarget.classList.add("fa-solid");
>       this.iconTarget.classList.remove("fa-regular");
>   } else {
>       this.iconTarget.classList.add("fa-regular");
>       this.iconTarget.classList.remove("fa-solid");
>   }
> }
```


### 1-9、步驟七：優化CODE

#### 1-9-1、JS抽象化

以下是完整的段落，如果不清楚再去翻檔案來看，註解都還沒刪掉  
這邊是用fc濃縮過後的版本  
```md
> export default class extends Controller {
>  static targets = [ "text", "icon" ]
>
>  initialize() {
>    library.add(thumbsSolid, thumbsRegular)
>  }
>
>  connect() {
>    dom.watch()
>    if (this.element.dataset.liked === "true") {
>        this.setLikedState(true)
>    } else {
>        this.setLikedState(false)
>    }
>  }
>
>  toggle() {
>
>    const token = document.querySelector("meta[name='csrf-token']").content
>    const id = this.element.dataset.id
>    fetch(`/wish_lists/${id}/like`, {
>        method: "PATCH", 
>        headers: {
>            "X-CSRF-Token": token
>        },
>    })
>      .then((resp) => {
>        return resp.json()
>      })
>      .then(({status}) => {
>        
>        if(status === "liked") {
>            this.setLikedState(true)
>        } else {
>            this.setLikedState(false)
>        }
>      })
>      .catch((err) => {
>        console.log(err);
>      })
>   
>  }
>  
>  setLikedState(state) {
>    if (state) {
>      this.iconTarget.classList.add("fa-solid");
>      this.iconTarget.classList.remove("fa-regular");
>    } else {
>      this.iconTarget.classList.add("fa-regular");
>      this.iconTarget.classList.remove("fa-solid");
>    }
> }
```




#### 1-9-2、新增liked_by的方法
最後優化，把剛剛data-liked抓到的東西，寫一個新方法做簡化  
這一段有點醜，我們縮短一點  
```md
> data-liked="<%= current_user.liked_wish_lists.include?(@wish_list) %>"
```

我想要改成這樣，不過like_by這個方法還沒弄出來
```md
> data-liked="<%= current_user.liked_by?(@wish_list) %>"
```

因此我們到wish_list的model，把like_by寫上去
這一句話的意思是，喜歡這張許願卡的使用者，有沒有包括自己(因為我們目前只有自己可以點贊)
```md
> def like_by?(user)                # 這個user是方法的參數，等等要傳引數進來的
>   liked_users.include?(user)
> end
```


再來到許願卡controller的like那邊更改剛剛一長串的東西  
原本的寫法  
```md
> def like
>   if current_user.liked_wish_lists.include?(@wish_list)           # 這一行(舊)
>     current_user.liked_wish_lists.delete(@wish_list)
>     render json: {status: "unliked", id: @wish_list.id}
>   else
>     current_user.liked_wish_lists << @wish_list
>     render json: {status: "liked", id: @wish_list.id}
>   end
> end
```


用like_by?(user)的寫法 
```md
> def like
>   if if @wish_list.liked_by?(current_user)                        # 這一行(新)
>     current_user.liked_wish_lists.delete(@wish_list)
>     render json: {status: "unliked", id: @wish_list.id}
>   else
>     current_user.liked_wish_lists << @wish_list
>     render json: {status: "liked", id: @wish_list.id}
>   end
> end
```



2、套用markdown語法
------

:::tip
讓rails可以使用markdown的顯示  
首先找很多人使用的套件，老師上課示範的是這個   
[mark down套件](https://www.npmjs.com/package/marked)  
:::


接下來就來安裝拉，我們使用yarn
```md
> yarn add marked
```

裝好後就可以來把文字轉成markdown了   
1. 抓出該文字區塊，該文字區塊是article  
2. 把該區塊加上 - data-controller="marked"  
3. 加好後，就可以在JS那邊多開一個markdown_controller.js的檔案(這裡寫JS的)  
4. 在該資料夾引入marked套件  
5. 把原本的文字，套用markdown語法   
  
找到該區塊，就可以加上data-controller，這一行的目的是，讓我們可以使用JS操作HTML
```md
> <article data-controller="markdown">
```

新增markdown.controller，並且在裡面寫這一份
```md
> import { Controller } from "stimulus"
> import marked from "marked"                   # 引入marked
> 
> export default class extends Controller {
>   connect() {
>     this.element.innerHTML = marked.parse(this.element.textContent)  # 依照官方手冊，把文字套用markdown
>   }
> }
```

:::tip
老師引入marked的流程，是先猜測對對方有export default一個變數，所以可以不用大括弧     
`import marked` -> 這個 marked 可以自己亂取名字   
:::




3、namespace + routes
------
為啥要特別用namespace加上路徑呢？像下面這樣寫  
  

我想要的路徑 
```md
> /api/wish_lists/2/like
```

用namespace來做
```md
> namespace :api do
>   resources :wish_lists, only: [] do
>     patch :like
>   end
> end
```

用namespace寫的原因是因為，今天的情境下，我們存粹是要在路徑前面加上/api，  
就不要使用使用resources，用resources產生一堆方法，不過我們又用不到   
而且用namespace好處，可以在controller那邊多一個資料夾，分開會很清楚也很好寫   

:::tip
後台系統用namespace寫  
/admin/users  
:::



如果今天網站改版，路徑中多了v2，就繼續用namespace再包一層就好
```md
> /api/v2/wish_lists/2/like
```

路徑設定
```md
> namespace :api do
>   namespace :v2 do
>     resources :wish_lists, only: [] do
>         patch :like
>     end
>   end
> end
```

這樣寫完路徑後，可以值接寫指令，產生出api的controller
```md
> rails g Controller api/like
```

:::danger  
！！！！設計api路徑，要考慮到升級的情況  
:::
