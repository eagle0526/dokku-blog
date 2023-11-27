---
title: ROR/Rails google login 跳出(popup)頁面
sidebar_label: "7. [ROR] Rails google login 跳出(popup)頁面"
description: google login popup
last_update:
  date: 2023-11-07
keywords:
  - ROR
  - google-login
sidebar_position: 7
---


:::tip
**前情提要：**         
如果今天想要對google登入或者facebook登入，做popup頁面  
也就是點擊登入按鈕後，不是進入特定頁面，而是彈出一個視窗出來，這樣怎麼做呢？
:::



安裝devise
------


照著下面的流程做

(1) 先安裝devise

```shell
$ bundle add devise
```

(2) 產生devise設定檔案
```shell
$ rails g devise:install
```

(3) 測試信件會寄送到local:3000

```md
config/environment/development.rb

> config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

Ps. 在開發檔案放上這一段code，可以讓寄送信件的時候寄到localhost:3000，如果之後正式上線，在production.rb這個檔案，要放上正是網域的位置，寄送信件才會是正確位置


(4) 創建User的model

```shell
$ rails g devise User
$ rails db:migrate
```

Ps. 這樣新增後，之後想要登入，就可以用current_user這個方法，如果今天不想要用User也可以，改成rails g devise Manager就好，之後就會變成這樣，current_manager

(5) 更改devise外觀

```shell
$ rails g devise:views
```


(6) 客製化controller     

後面那個scope，假設今天你剛剛前面的model是用User，這邊就把scope改成user
```shell
$ rails generate devise:controllers [scope]
```

來生成user的詳細controller，輸入rails g devise:controller user後，會生成這幾個controller的檔案給你
```shell
$ rails generate devise:controllers users

...
create  app/controllers/users/confirmations_controller.rb
create  app/controllers/users/passwords_controller.rb
create  app/controllers/users/registrations_controller.rb
create  app/controllers/users/sessions_controller.rb
create  app/controllers/users/unlocks_controller.rb
create  app/controllers/users/omniauth_callbacks_controller.rb
```



### 增加User欄位

我想要讓登入者在註冊的時候，可以多註冊一個欄位，要怎麼加呢？

(1) 在user migration新增一行

```rb
class class DeviseCreateUsers < ActiveRecord::Migration[6.1]
  def change
    ...

    t.string :name

    ...
  end
end
```

(2) 把migration重新設定
```shell
$ rails db:migrate:reset
```

(3) 加上強參數

到剛剛新增的user controller，來增加幾段code，這樣新增好後，就可以讓使用者在註冊新會員時，新增name欄位
```md
app/controllers/users/registrations_controller.rb

> module Users
>   class RegistrationsController < Devise::RegistrationsController
>     before_action :configure_sign_up_params, only: [:create]
> 
>     ...略
> 
>     private
> 
>     # If you have extra params to permit, append them to the sanitizer.
>     def configure_sign_up_params
>       devise_parameter_sanitizer.permit(:sign_up, keys: [:name])
>     end    
> 
>   end
> end
```

(4) 設定畫面

設定登入的HTML
```md
views/shared/_navbar.html.erb

> <nav class="fixed top-0 left-0 z-50 w-full px-3 py-4 bg-white shadow md:flex md-items-center md:justify-between">
>   <div class="flex items-center justify-between w-full">
>     <span class="text-2xl cursor-pointer">
>       <%= link_to "租屋評價", root_path %>
>     </span>
>   <ul class="flex" data-controller="nav">
>     <div class="flex">
>       <div class="flex items-center">
>         <% if current_user %>          
>           <li><%#= link_to "更新暱稱", edit_admin_user_path(current_user.id), class:"mr-2 text-lg hover:text-major" %></li>
>           <li><%= image_tag current_user.avatar %></li>
>           <%#= image_tag current_user.avatar.variant(:thumb) %>
>           <li><%= link_to current_user.email, root_path, class:"mr-2 text-lg hover:text-major" %></li>
>           <li class= "text-lg btn-primary">
>             <%= link_to '登出', destroy_user_session_path, method: "delete" %>
>           </li>
>         <% else %>
>           <li class= "mr-2 text-lg hover:text-major">
>             <%= link_to '註冊', new_user_registration_path, class: "btn-primary" %>
>           </li>
>           <li class= "text-lg hover:text-major">
>             <%#= link_to '登入', new_user_session_path, class: "second-btn", data: {action: "click->nav#popup"} %>
>             <button class= "second-btn", data-action= "click->nav#popup">
>               <p>登入</p>
>             </button>
>             
>           </li>  
>         <% end %>
>       </div>
>     </div>
>   </ul>
> </nav>
```

渲染navbar的程式碼
```md
views/layouts/applications.html.erb

> <html>
>   ...略
> 
>   <body>
>     <div>
>       <p class="notice"><%= notice %></p>
>       <p class="alert"><%= alert %></p>    
> 
>       <%= render "shared/navbar" %>
>       <%= yield %>
>     </div>    
>   </body>
> </html>
```


到devise註冊的畫面修改程式碼，照下面這樣寫，倒註冊頁面的時候，會多出一個name的欄位


```md
views/devise/registrations/new.html.erb

> ... 略
>
>  <div class="field">
>    <%= f.label :name %><br />
>    <%= f.text_field :name, autofocus: true, autocomplete: "email" %>
>  </div>
>  
> ... 略

```

！！！！最重要的地方，變更路徑
```rb
Rails.application.routes.draw do
  # 原本的路徑
  # devise_for :users

  # 變更後的路徑
  devise_for :users, controllers: { registrations: 'users/registrations' }


  # Define your application routes per the DSL in https://guides.rubyonrails.org/routing.html

  # Defines the root path route ("/")
  root "home#index"
end

```






設定google登入
------

(1) 安裝需要的gem

```rb
gem "omniauth", "~> 2.1"
gem "omniauth-google-oauth2", "~> 1.1"
gem "omniauth-facebook", "~> 9.0"
gem "omniauth-rails_csrf_protection", "~> 1.0"
```

執行安裝上面幾的gem
```shell
$ bundle
```

(2) 申請google API接口

照個這個[教學](https://medium.com/tingyiiii/rails%E5%AF%A6%E4%BD%9C%E7%AC%AC%E4%B8%89%E6%96%B9%E7%99%BB%E5%85%A5-google-2a0851b74193)申請google登入需要的ID、密碼


(3) 設定環境變數

把剛剛得到的ID、密碼設定成環境變數
```md
> GOOGLE_CLIENT_ID=XXXX
> GOOGLE_CLIENT_SECRET=XXXX
```


(4) 在以下檔案加入環境變數

```md
config/initializers/devise.rb

Devise.setup do |config|
  .....
  config.omniauth :google_oauth2, ENV['GOOGLE_CLIENT_ID'], ENV['GOOGLE_CLIENT_SECRET']
end
```


(5) 幫剛剛的User Model，新增一格給google ID
```shell
$ rails g migration add_omniauth_to_users provider:string uid:string
```

上面那樣輸入後，會產生這這個檔案
```md
> class AddOmniauthToUsers < ActiveRecord::Migration[6.1]
>   def change
>     add_column :users, :provider, :string
>     add_column :users, :uid, :string
>   end
> end
```

```shell
$ rails db:migrate
```


(6) 在User的model中建立類別方法，等等要登入用、還有幾個devise要用到的東西

```md
model/user

> class User < ApplicationRecord
>   
>   devise :database_authenticatable, :registerable,
>          :recoverable, :rememberable, :validatable,
>          :omniauthable, omniauth_providers: [:facebook, :google_oauth2]
> 
> 
> 
>   // 加上這一段
>   def self.create_from_provider_data(provider_data)
>    where(email: provider_data.info.email).first_or_create do |user|
>      user.email = provider_data.info.email
>      user.password = Devise.friendly_token[0, 20]
>      user.name = provider_data.info.last_name
>      user.provider = provider_data.provider
>      user.uid = provider_data.uid
>    end
>   end
> end
```


(7) 增加對應的路徑

```md
routes

> Rails.application.routes.draw do
>   // 增加這一段
>   devise_for :users, controllers: { omniauth_callbacks: "users/omniauth_callbacks" }

>   root "home#index"
> end
```

(8) 在controller增加相對應方法
```md
app/controllers/user/omniauth_callbacks_controller.rb

> class User::OmniauthCallbacksController < Devise::OmniauthCallbacksController
> 
>   # 新增此段第三方登入的方法
>   def google_oauth2
>     @user = User.create_from_provider_data(request.env["omniauth.auth"])
>     if @user.persisted?
>       flash[:notice] = I18n.t "devise.omniauth_callbacks.success", :kind => "Google"
>       sign_in_and_redirect @user, :event => :authentication
>     else
>       session["devise.google_data"] = request.env["omniauth.auth"]
>       redirect_to new_user_registration_url
>     end
>   end
>   # 及這段第三方登入失敗的處理方法
>   def failure
>     redirect_to root_path
>   end
>   
> end
```

Ps. 這樣都設定好之後，就可以實際登入看看了，應該會成功才對！！



登入/註冊使用POPUP的方法
------



(1) 新增popup區塊

Ps. 一開始這個section是隱藏的，當我們點擊右上角的登入後，才跳出這個popup，因此要把此區塊的hidden刪掉

```md
_navbar.html.erb

> <!-- ads/popup.html.erb -->
> <section class="hidden" data-controller="nav-pop-up">
>   <div class="fixed inset-0 z-50 flex items-center justify-center bg-black bg-opacity-50">
>     <%# <i class= "absolute text-xs fa-solid fa-x"></i> %>
>     <!-- 背景是半透明的黑色蓋板廣告 -->
>     <div class="p-20 bg-white rounded-lg w-96">
> 
>       <!-- 白色蓋板廣告的內容 -->
> 
>       <%# 關閉icon，因為是用絕對定位，所以今天如果改變白色框框的大小，叉叉icon都要再調整過 %>
>       <i class= "absolute text-xs fa-solid fa-x right-[47em] top-[25em]"></i>
>             
>       <h2 class="mb-4 text-2xl font-bold">社群登入</h2>
> 
>       <%# 原始的登入code %>
>       <%#= button_to "google登入", user_google_oauth2_omniauth_authorize_path, data: {turbo: "false"}, class: "btn-primary rounded-full w-full " %>    
>           
>       <%# google登入按鈕 %>
>       <%= button_to user_google_oauth2_omniauth_authorize_path, data: {turbo: "false"}, class: "inline-flex px-6 py-2 cursor-pointer hover:bg-blue-100 items-center 
>       rounded-full w-full border bg-white flex text-black" do %>        
>         <div class="w-1/6 mr-8">
>           <img src="/assets/google_logo.png" class="object-cover w-full h-full">
>         </div>
>         <div class="">
>           <span>google登入</span>
>         </div>                     
>       <% end %>
> 
>     </div>
>   </div>
> </section>
```





(2) 把navbar按鈕加上controller、action


1. 先在navbar設定controller
2. 把社群登入按鈕，加上action

```md
_navbar.html.erb

> <nav class="fixed top-0 left-0 z-50 w-full px-3 py-4 bg-white shadow md:flex md-items-center md:justify-between" data-controller="nav-bar">
>    ...略
>    <button class= "mx-5 second-btn", data-action="click->nav-bar#login">
>      <p>社群登入</p>
>    </button>   
> </nav>
```

(3) 設定stimulus js，要讓兩個js溝通
```md
nav_bar_controller.js

>  login(){
>    const evt = new CustomEvent("popup")
>    window.dispatchEvent(evt)
>  }
```

(4) pop頁面接收剛剛那一個action，並且多設定一個target，當我們按下登入按鈕，要把這個section的hidden css給刪掉


```md
_navbar.html.erb

> <section class="hidden" data-controller="nav-pop-up"
>                         data-nav-pop-up-target="page"
>                         data-action="popup@window->nav-pop-up#showPop">
> 
> </section>                        
```

(5) 設定nav-pop-up的js，當我們接收到按下登入按鈕的動作時，刪除hidden

```md
nav_pop_up_controller.js

> showPop() {    
>   this.pageTarget.classList.remove("hidden")
> }
```


(6) 針對頁面的叉叉，增加一個target，按下叉叉的時候，要把hidden加上去，所以我們對icon增加一個action


```md
_navbar.html.erb

> <section class="hidden" data-controller="nav-pop-up"
>                         data-nav-pop-up-target="page"
>                         data-action="popup@window->nav-pop-up#showPop">
>   ...略

>   <i class= "absolute text-xs fa-solid fa-x right-[47em] top-[25em] cursor-pointer" data-action="click->nav-pop-up#close"></i>

>   ...略
> </section>                        
```


(7) 寫叉叉action的JS
```md
nav_pop_up_controller.js

> close() {
>   this.pageTarget.classList.add("hidden")
> }
```


### 結論

這樣做就完成google登入的popup頁面設定，不過要記得，rails 6 和 rails 7 的設定有點不一樣，因為 rails 7 turbo 的關係，所以今天想要點連結用POST action送出的時候，方式有改變。


#### rails 6

google登入連結，用post送出
```md
> <%= link_to "google登入".html_safe, user_google_oauth2_omniauth_authorize_path, method: :post %>  
```
#### rails 7

google登入連結，用 post 送出，並且把 turbo 關掉
```md
> <%= button_to "google登入", user_google_oauth2_omniauth_authorize_path, data: {turbo: "false"}, class: "btn-primary rounded-full w-full " %>    
```

在 button_to 按鈕中，加上其他文字 or 圖片
```md
> <%= button_to user_google_oauth2_omniauth_authorize_path, data: {turbo: "false"}, class: "inline-flex px-6 py-2 cursor-pointer hover:bg-blue-100 items-center 
> rounded-full w-full border bg-white flex text-black" do %>        
>   <div class="w-1/6 mr-8">
>     <img src="/assets/google_logo.png" class="object-cover w-full h-full">
>   </div>
>   <div class="">
>     <span>google登入</span>
>   </div>                     
> <% end %>
```


rails 7 特別注意事項
----

### rails 7 登入正確範例
多新增一個google登入連結在pop頁面上，如果今天你是用rails 7想要做popup的登入頁面，有一個地方要修改，像下面這樣

```md
> <section>
>   <div class="fixed top-0 left-0 z-50 flex items-center justify-center w-full h-full bg-gray-500 opacity-70">    
>     <div class="w-2/5 py-10 bg-white">
>       <p>123</p>      

>       -下面這一行-
>       <%= button_to "google登入", user_google_oauth2_omniauth_authorize_path, data: {turbo: "false"} %>  

>     </div>
>   </div>
> </section>
```

Ps. 一定要用button_to、把turbo關掉，要不然會遇到CORS的問題



### rails 7 登入錯誤範例

#### (1) 一樣使用 link_to - method: :post

```md
> <%= link_to "google登入".html_safe, user_google_oauth2_omniauth_authorize_path, method: :post %>      
```

會噴出以下錯誤
```md
Not found. Authentication passthru.
```

#### (2) 改成rails 7 專用的寫法
```md
> <%= link_to "google登入".html_safe, user_google_oauth2_omniauth_authorize_path, data: {turbo_method: "post"} %>      
```

會噴出以下錯誤
```md
Access to fetch at 'xxx' (redirected from 'http://localhost:3000/users/auth/google_oauth2') from origin 'http://localhost:3000' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.
```


#### (3) 把turbo關掉

```md
> <%= link_to "google登入".html_safe, user_google_oauth2_omniauth_authorize_path, method: :post, data: {turbo: false} %>      
```

會噴出以下錯誤
```md
Not found. Authentication passthru.
```




