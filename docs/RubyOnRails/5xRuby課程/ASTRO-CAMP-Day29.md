---
title: ROR/Ruby
sidebar_label: "DAY29. [ASTROCamp] Ruby-7"
description: Ruby
last_update:
  date: 2022-11-18
keywords:
  - 5xRuby
  - Ruby
sidebar_position: 30
---


1、會員系統 User model
------

WishList第二個MODEL => user會員系統

新增會員會有的路徑
```md
> /users/new       => 會員註冊又等於新增user
> /users/create    
> /users/show      => show出會員資料
> /users/edit      => 修改會員資料
> /users/update     
> /users/destroy   => 刪除會員資料
```


user路徑創造，我們這邊使用單數的resource，這樣show不用帶id
```md
> resource :user
```

:::tip
**resource使用慣例**  
單數 :單數  
複數 :複數  
:::

:::tip
**resource + only**  
路徑也可以這樣寫，這樣可以限制只產生指定action的路徑  
resource :user, only:[:index, :show]  
:::



要記得把筆記寫完，應該是11/18 - 第二個影片
### resource :user Vs. resources :users


**把這兩者的路徑全部印出來**
resource :user

沒有format :id


**把這兩者的路徑全部印出來**
resources :users

有format :id







:::info
做前台，不應該讓網址帶有id，像是會員系統的id、購物車的id，都不用讓它帶有id  
:::





2、User會員系統的CRUD
------

### 2-1、第一步驟：navbar

因為登入連結通常會做在navbar上，所以我們做一個navbar出來  
不過因為是全站都看得到，所以寫在layout那邊  
```md
> <nav id="nav">
> 
>     <%= link_to "註冊", new_users_path %>
>     <%= link_to "登入", "#" %>
>     <%= link_to "登出", "#" %>
> 
> </nav>
```

但是還記得之前我們做的shared資料夾嗎？我們可以把nav放在這邊，並用render渲染過去
```md
> <body>
>     <%= render "shared/navbar" %>           # 這一行
>     <%= render "shared/flash" %>
>     <%= yield %>
> </body>
```




### 2-2、第二步驟：form_with

new頁面用form_with做表單，要注意事項
```md
> <%= form_with(model: @user) do |form| %>
>     <div>
>       <%= form.label :nickname %>
>       <%= form.text_field :nickname %>
>     </div>
>     <div>
>       <%= form.label :email %>
>       <%= form.email_field :email %>
>     </div>
>     <div>
>       <%= form.label :password %>
>       <%= form.password_field :password %>
>     </div>
> 
>     <%= form.submit "註冊" %>
> <% end %>
```



but!!!，會發現form_with，會找不到users_path，因此要來更改一下路徑  
```md
> resource :user, as: "users"  
```
這個可以把path變成都是負數  
不過要記得在路徑後面+as 動的是 prefix，實際上網址還是單數  






### 2-3、第三步驟：密碼再次輸入

#### 2-3-1、confirmation
密碼再次確認的欄位 password_confirmation  
model的confirmation方法，用這個東西可以不用再幫model開欄位，rails會幫你做雙重確認   
  
```rb
# user的model
class User < ApplicationRecord
    validates :email, presence: true
    validates :password, confirmation: true             # 加上密碼確認
end
```

user的view，增加這個欄位
```html
<div>
    <%= form.label :password_confirmation %>
    <%= form.password_field :password_confirmation %>
</div>
```

#### 2-3-2、length

再多加一個條件，密碼最小長度不能小於6個字 -> `length: {minimum: 6}`

```rb
# user的model

class User < ApplicationRecord
  validates :email, presence: true
  validates :password, confirmation: true, length: {minimum: 6}  # 最小長度
end
```




### 2-4、第四步驟：user_params

做user的create，create的流程，記得先做資料驗證的方法(password_confirmation要記得給進去喔)  
然後可以先把它params出來看看，會發現

```rb
def create
    render html: user_params
end

private
def user_params
    params.require(:user).permit(:nickname, :email, :password, :password_confirmation)
end
```


### 2-5、第五步驟：create user

把create寫進資料庫

```rb
def create
    @user = User.new(user_params)

    if @user.save
        redirect_to root_path, notice: "註冊成功"
    else
        render :new
    end
end
```


### 2-6、第六步驟：密碼加密

官網詳細callback => [callback流程](https://guides.rubyonrails.org/active_record_callbacks.html)  
  
Ps. creating an object -> 會有一整個週期，所以我們現在create物件時，要在某個階段，對密碼作加密  

#### 2-6-1、before_save  

**密碼加密原理**    
為啥要密碼加密呢？因為沒加密的話，存到資料庫的資料是使用者真實輸入的值，沒有轉換過，這樣是不對的  
使用雜湊演算法 MD5 SHA1 SHA256  
假設你今天的輸入值是123，他的輸出值會一樣  
    
第一次輸入的時候，密碼會用加密演算法，轉成一段亂碼後，並把他丟進資料庫  
之後還要進入此網站的話，使用者會再輸入一次密碼，如果密碼一樣，演算法轉出來的輸出值也會一樣  
這樣就可以登入了  

:::tip
資料庫存的是被hexdigest演算法轉換後的結果  
:::


```md
> User model

> class User < ApplicationRecord
> 
>     ....這邊有兩個validate(先省略)
> 
>     before_save :encrypt_password                                 -> 設定好加密後，在before_save階段
> 
>     private
>     def encrypt_password
>         # HASH 
>         # self.password = 編碼過的password
>         self.password = Digest::SHA1.hexdigest(self.password)         -> ruby官網有標準函式庫可以做此加密
>     end
> end
```


#### 2-6-2、before_create 
但！！！！有個很重要的東西，不只在create object有before_save，update object也有before_save  
所以假設model寫before_save，就會在你更新實體的時候，再次加密(加密兩次)，這會導致密碼直接不見喔！！  
要改成before_create   
```md
> before_create :encrypt_password                                 -> before_save改成before_create
```



#### 2-6-3、灑鹽

透過sha-1加密後，還是覺得不夠安全，我們現在來增加密碼複雜度(這個動作稱為灑鹽 salting)
```md
> self.password = Digest::SHA1.hexdigest("xy#{elf.password}zz")        # 在原本密碼上加別的字串，增加複雜度
```

:::tip
**鹽(密碼學)**  
在密碼學中，是指在雜湊之前將雜湊內容（例如：密碼）的任意固定位置插入特定的字串。  
這個在雜湊中加入字串的方式稱為「加鹽」。其作用是讓加鹽後的雜湊結果和沒有加鹽的結果不相同，  
在不同的應用情景中，這個處理可以增加額外的安全性。  
:::



:::info
**清掉資料庫所有資料**  
到rails c  
```shell
$ User.destroy.all  
```
Ps. 這個指令使用要小心  
:::



### 2-7、第七步驟：登入路徑
```md
> 登入路徑預計為 : /user/login (不過目前沒有此路徑)
```

```md
> resource :user, as: "users" do
>     # /user/login                     # 我想要的網址是這條
>     collection do                     # 用collection可以製造出上面那一條路徑
>         get :login
>     end
> 
> end
```

創好路徑後，就可以把login的action、view設定好了


#### 2-7-1、member VS. collection

**member**
如果今天想要新增的網址是有帶ID的，ex. /user/:id/login(.:format)	    
就可以使用member，用member做出的路徑是這樣  
  
|path|verb|url|Controller#Action |
|:-:|:-:|:-:|:-:|
|login_users_path|GET|/users/:id/login(.:format)|users#login|
  
    
**collection**  
不過如果今天要沒有要讓新分頁有id，ex. /user/login(.:format)	 
  
|path|verb|url|Controller#Action |
|:-:|:-:|:-:|:-:|
|login_users_path|GET|/users/login(.:format)|users#login|
  

:::tip
**那什麼時候會不想要讓連結內帶有id呢？？**  
其實以登入的網址來說，就不會帶ID，因為帶ID的話，不就讓別人知道你的ID是多少了  
:::
  
  
  
### 2-8、第八步驟：session路徑

**新增session的路徑**  
為什麼要新增這個路徑呢？  
因為在做登入/登出的時候，其實算是額外不屬於user model的行為  
因此你拿著只要拿著號碼牌，就可以登入、登出，所以我們把兩者分開寫  
  
Ps. 我們的session路徑只需要新增和刪除  
```rb
resource :session, only: [:create, :destroy]
```



### 2-9、第九步驟：login表單

User的login表單完成(這邊是user的view)  
注意事項：  
1. 要記得這是User Model的表單，不過我們要Post到的地方是session的path(原因第八步驟有寫)  
2. 千萬要記得form_with用括號寫的話，和後面的url不能有空格，要不然會讀不到  
3. form_with可以不用寫model的原因是，今天不是在做新增，如果寫model的話，他會幫你做幾件事   
  - (1) 幫input欄位用:user包起來，這樣在params時候可以一包抓  
  - (2) rails 會幫你猜路徑  

不過今天url是我們客制的(導到session_path)，再加上也不太需要包成一包，兩個欄位分別params就好  
所以這邊model可以不用寫。 
  
```md
> <%= form_with(url: session_path) do |form| %>
>   
>     <div>
>       <%= form.label :email %>
>       <%= form.email_field :email %>
>     </div>
>     <div>
>       <%= form.label :password %>
>       <%= form.password_field :password %>
>       
>     </div>

>     <%= form.submit "登入" %>  
> 
> <% end %>
```

:::info
如果今天用form_for，這邊就不能不寫model，會報錯誤  
:::

### 2-10、第十步驟：session create

登入表單設計好後，要來寫完session的create了
**流程是這樣**  
1. 先抓住剛剛送過來來的兩個資料email、password，不過要記得分開抓，因為我們form表單沒寫model，不會包成一包  
2. 接著把登入者輸入的密碼，在做一次跟註冊帳號時一樣的加密流程，用find_by方法跟已經寫進資料庫的帳號**比對**  
3. 如果比對的結果是一樣的，就發一張號碼牌給該為使用者(現在還沒寫，後面會提到)  



```md
> session的create action

> def create
>     # render html: params        
>     email = params[:email]
>     password = params[:password]
> 
>     # hashed_password ...(password)
>     hashed_password = Digest::SHA1.hexdigest("xy#{password}zz")
> 
>     user = User.find_by(email: email, password: hashed_password)
> 
>     if user
>         # 發牌                                                # 這個等等寫
> 
>         # 重導
>         redirect_to root_path, notice: "登入成功"
>     else
>         redirect_to login_users_path, alert: "登入失敗"
>     end
> 
> end
```



#### 2-10-1、create流程優化

不過上面的寫法有點髒亂，我們整理一下  
在User model多創一個login方法，把剛剛那些加密的邏輯寫進去，這樣就可以讓controller乾淨一點

此方法的邏輯  
1. 把剛剛加密、比對的兩行程式碼放過來
2. 因為這個是User的類別方法，記得要加上self

```md
> User model

> class User < ApplicationRecord
> 
>     def self.login(email, password)
>         hashed_password = Digest::SHA1.hexdigest("xy#{password}zz")
>         find_by(email: email, password: hashed_password)
>     end
> 
> end
```

session的create變這樣(原本兩行變一行)
```md
> 舊
> hashed_password = Digest::SHA1.hexdigest("xy#{password}zz")
> user = User.find_by(email: email, password: hashed_password)

> 新(取代上面兩行)
> user = User.login(email, password)
```



session 的 create 變乾淨了
```md
> class SessionsController < ApplicationController
>     def create
>         
>         email = params[:email]
>         password = params[:password]
> 
>         user = User.login(email, password)
>         if user
>             # 發牌
> 
>             # 重導
>             redirect_to root_path, notice: "登入成功"
>         else
>             redirect_to login_users_path, alert: "登入失敗"
>         end
> 
>     end
> end
```




:::tip **為啥大部分商業邏輯放在model那邊**  
大部分的商業邏輯都整理在model裡面  
因為model很常重複使用  
controller基本上不常重複使用  
所以會把一些很常重複使用的方法寫在model那邊  
:::




### 2-11、第十一步驟：發牌-session[:handsome]

:::tip
**什麼叫登入成功？？**    
成功拿到號碼牌，才算登入成功  
如果只是帳號密碼比對成功，只是比對資料，不算是登入喔  
:::

####  2-11-1、session vs cookie

使用者今天進來網站，登入成功會拿到一張號碼牌，這個號碼牌就是cookie，session則是會存在server那邊  
1. session是server端的(櫃檯是session)  
2. cookie是使用者端的(瀏覽器會存cookie，使用者拿cookie)  
  
最後這兩張號碼牌會對起來，假設server、使用者的號碼牌有對起來，這樣就可以讓使用者登入  
假設server重開，導致session今天整個重設，就算cookie一樣還活著，使用者一樣登不進去  


接著我們就把到session的create，把號碼牌發給登入資料比對成功的使用者
```md
> if user
>     # 發牌
>     session[:handsome] = user.id                    # 如果資料比對成功的話，發一張號碼牌給這個使用者
>     # 重導
>     redirect_to root_path, notice: "登入成功"
> else
>     redirect_to login_users_path, alert: "登入失敗"
> end
```


:::info
**號碼牌的注意事項**  
session[:handsome] = user.id  
(1) [:handsome] > 這個變數可以隨便亂取，只要記得就好  
(2) user.id > 這個也可以隨變取，但是通常我們知後會要用到剛剛登入者的資料，顯示在瀏覽器上，所以給他有意義的變數  
:::



### 2-12、第十二步驟：session登出

把號碼牌丟掉，把session設定為nil，這樣就可以登出了
```md
> def destroy
>     session[:handsome] = nil
>     redirect_to root_path, notice: "已登出"
> end
```






### 2-13、第十三步驟：按鈕狀態顯示

到設定navbar的地方，把判斷登入要出現的按鈕和登出會顯示的按鈕切開 
假設現在有拿到號碼牌，navbar應該要顯示登出，如果現在還沒拿到號碼牌，應該要顯示登入、註冊
  
這裡在shared/_navbar/html/erb新增判斷  
```md
> <nav id="nav">
>     <% if session[:handsome] %>
>         <%= link_to "登出", session_path, method: "delete" %>
>     <% else %>
>         <%= link_to "註冊", new_users_path %>
>         <%= link_to "登入", login_users_path %>
>     <% end %>
> </nav>
```




### 2-14、第十三步驟：current_user

接著我想在navbar顯示使用者的名字，可以怎麼做  
Ps. 還記得我們前面session[:handsome]有先 = user.id嗎？這時候我們就可以拿來用了

#### 2-14-1、第一種方法 - 直接寫在navbar
```md
> <nav id="nav">
>   <% if session[:handsome] %>

>       <% u = User.find_by(id: session[:handsome]) %>          # 先抓到登入者id
>       <%= u.nickname %>                                       # 使用者id的nickname

>       <%= link_to "我的許願卡", new_wish_path %>
>       <%= link_to "登出", session_path, method: "delete" %>
>   <% else %>
>       
>       <%= link_to "註冊", new_users_path %>
>       <%= link_to "登入", login_users_path %>
>   <% end %>
> </nav>
```


#### 2-14-2、第二種寫法，寫在helper上  
第一種寫法，會讓view頁面看起來好醜(太多不屬於view頁面的判斷式了)   
  
到user的helper寫上current_user方法
```md
> module ApplicationHelper
>     def current_user                                 # 新增這個方法
>         User.find_by(id: session[:handsome])         # 如果有找到id，會回傳一個實體，否則回傳nil
>     end
> end
```


不太清楚的話，我們把current_user印出來看看(所有的view頁面都可以印出來)
```md
> <%= current_user %>               # <User:0x000000010a7be9f8>，印出一個實體
```


接著我們就可以用current_user把東西印出來了
```md
> <nav id="nav">
>     <%# if session[:handsome] %>
> 
>         <%= current_user.nickname %>                          # 這一行
> 
>         <%= link_to "註冊", new_users_path %>
>         <%= link_to "登入", login_users_path %>
>     <%# else %>
>         <%= link_to "登出", session_path, method: "delete" %>
>     <%# end %>
> </nav>
```




#### 2-14-3、優化current_user效能

如果今天要連續抓current_user五次，這樣會非常耗效能(連續賦值五次)  
使用||=的預設值概念，有了這個東西後，一開始沒有該值，就給User.後面那一串，有的話就@_user_   

因此來稍微改寫一下current_user的寫法    
```md
> module ApplicationHelper
>     def current_user
>         @_user_ ||= User.find_by(id: session[:handsome])
>     end
> end
```

把值存在實體變數裡面(那個變數的名字不重要，只要跟別的做好區隔就可以)，如果第一次呼叫current_user這個方法  
就把抓到的實體賦予到一個變數裡面，如果後面再繼續呼叫這個方法，就直接把剛剛那個變數丟給他就好  
  
Ps. 這個手法叫做memorization(紀錄)，可以幫助SQL語法不會重複做事  



#### 2-14-4、製作"user_signed_in?"方法

除了current_user外，我們再把navbar的view檔案優化一樣，把原本如果有發號碼牌，改成比較好閱讀懂的方法

首先到helper，製作新方法
```md
> def user_signed_in?
>     !!session[:handsome]
> end
```

Ps. 兩個驚嘆號的意思是，可以強制把一個字串、數字轉成boolean，第一個!可以轉成false，第二個!轉成true
```md
> 詳解兩個驚嘆號

> session[:handsome]     # 一開始 = user.id
> !session[:handsome]    # 一個驚嘆號 = false
> !!session[:handsome]   # 第二個驚嘆號 = true
```


這樣修改後，就可以讓這個方法取代原本的是否有登入的判斷
```md
> <% if user_signed_in? %>
> <% end %>
```





### 2-15、第十四步驟：登入狀態顯示頁面

接下來設定有登入才能有權限看的頁面，如果沒有登入，把某些頁面關掉(不能看到)  
Ps. 如果你沒有號碼牌，就有把你踢走  


到wishlist的new頁面，多加一段判斷，如果有號碼牌，才讓你可以new新實體
```rb
def new
  if session[:handsome]
    @wish_list = WishList.new
  else
    redirect_to login_users_path, alert: "請先登入"
  end
end
```



### 2-16、第十五步驟：優化未登入狀態authenticate_user!

因為除了new，其他create......等等action，也要用到是否登入的判斷     
因此我們多做一個方法，讓這些action在啟動前就先觸發      
  
  
```md
> wishlist controller

> before_action :authenticate_user!         # 全部的action都會做登入判斷

> 如果沒有號碼牌，那就跳回登入頁面，並且顯示請你登入的訊息 
> def authenticate_user!
>     redirect_to login_users_path, notice: "請先登入" unless session[:handsome]
> end
```
  
  
不過authenticate_user在其他地方也會用到，所以把這個加到application那邊  
```md
> class ApplicationController < ActionController::Base
> 
>     ...上面還有其他的方法先省略
> 
>     def authenticate_user!
>         redirect_to login_users_path, alert: "請先登入" unless session[:handsome]
>     end
> end
``` 
  
  
### 2-17、第十六步驟：include helper

我們再來優化一下剛剛的authenticate_user!，把號碼牌的領取，用剛剛寫在helper的的方法user_signed_in?
```md
>  換比較好懂的寫法(不要用unless)，如果沒有登入就踢出去
> 
>  def authenticate_user!
>   if not user_signed_in?                              
>      redirect_to login_users_path, alert: "請先登入"
>   end
>  end
```

#### 2-17-1、helper使用注意事項
:::tip
但!!!!上面那個寫法會報錯喔，會噴 `MethodError!!` 為什麼呢??   
原因就是因為，寫在helper的方法，是給所有的view用的，不是給controller用的，所以現在    
要在controller使用，當然是不行的  
不過要解決的方法也非常簡單，只要在controller，import小幫手的模組就好了，這樣就可以讓controller使用helper  
:::
  
  
讓寫在helper的方法，讓controller可以用
```md
> class ApplicationController < ActionController::Base
>     rescue_from ActiveRecord::RecordNotFound, with: :record_not_found
> 
>     include ApplicationHelper                                     # 把helper import過來
> 
>     def record_not_found
>           render file: "#{Rails.root}/public/404.html", status: 404, layout: false
>     end
> 
>     def authenticate_user!
>       if not user_signed_in?                                      # 這裡就可以用helper了 
>         redirect_to login_users_path, alert: "請先登入"
>       end
>     end
> end
```



:::tip
**讓controller的東西可以給view用**  
在controller的檔案，寫上下面這一行    
helper_method :current_user, :user_signed_in?  
Ps. 那兩個方法一樣是寫在controller喔，只是我們把方法匯入過去  
:::


### 2-18、rails 的會員套件推薦
:::tip
devise 會員套件  
sorcery 會員套件  
:::






3、user、wishlist的關聯性
------

建立user、wishlist的關聯性  
幫wishlist開一個user_id欄位，連結到user  


### 3-1、migration add_index、add_belongs_to

#### 3-1-1、第一種寫法 - 開一個數字欄位
```md
> def change
>     add_column :wish_lists, :user_id, :integer
>     add_index :wish_lists, user_id
> end
```
  
  
#### 3-1-2、第二種寫法 - 開一個references欄位
```md
> def change
>     add_belongs_to :wish_lists, :user
> end
```


:::tip
add_belongs_to = add_references  
:::


上面兩種寫法，都會在wish_lists的table產生下面兩行
```rb
t.integer "user_id"
t.index ["user_id"], name: "index_wish_lists_on_user_id"
```


:::tip **二元樹**  
[無聊可以看一下](https://kopu.chat/%E4%BA%8C%E5%85%83%E6%A8%B9/)  
:::




到user的model，去寫has_many(一個使用者，有很多張的許願卡)
```md
> User Model

> has_many :wish_lists
> 
> 這樣寫，可以讓user多幾種方法用
> #wish_lists  
> #wish_lists= 
> #build  
> #create
```

還有到WishList的model，寫上belongs_to
```md
> WishList Model

> belongs_to :user
```



:::tip
has_many、belongs_to沒有一定要寫，他們只是幫你創造出一些方法，讓你方便存取  
:::



### 3-2、建立關聯後，新增新卡片
  
但是因為我們現在多了一個user_id的欄位，而且已經建立關聯了，所以在新增實體的時候，此欄位一定要存在   
所以用current_id設定為新增許願卡的人的id(還記得我們前面有號碼牌才進得來，所以才能這樣設定)  
```md
> def create
>   @wish_list = WishList.new(clean_params)
>   @wish_list.user_id = current_user.id
>
>   ...下面是資料儲存後成功與否的判斷
> end
```


不過上面那樣寫很醜，這兩行被下面那一行取代  
```md
> @wish_list = WishList.new(clean_params)
> @wish_list.user_id = current_user.id
> 
> @wish_list = current_user.wish_lists.new(clean_params)
> 由user的角度，創造出一張新的許願卡，而不是把user_id塞進去
```

最終wish_list的create的樣子
```md
> def create
> 
>     @wish_list = current_user.wish_lists.new(clean_params)
> 
>     if @wish_list.save
>         redirect_to make_a_wish_path, notice: "success new wish_list"
>     else
>         render :new
>     end
> end
```



### 3-3、登入者只能看到自己的東西

還記得我們before_action有設定一個find_wish_list的方法嗎？他是在做edit等等action可以操作的範圍   
現在我們要限制，只有登入者能對自己的資料crud(總不會讓別人也能修改你的資料)  
```md
> before_action :find_wish_list, only: [:edit, :update, :show, :destroy]
```

因此我們修改一下方法，一樣用current_user實體的角度出發，去找許願卡的id
```md
> def find_wish_list
>     # @wish_list = WishList.find_by(id: params[:num])         # 舊的，可以看到所有的資料
>     @wish_list = current_user.wish_lists.find(params[:id])    # 新的，只能看到自己登入的資料
> end
```