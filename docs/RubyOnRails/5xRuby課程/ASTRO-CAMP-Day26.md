---
title: ROR/Ruby
sidebar_label: "DAY26. [ASTROCamp] Ruby-4"
description: Ruby
last_update:
  date: 2022-11-14
keywords:
  - 5xRuby
  - Ruby
sidebar_position: 27
---


1、Rack
------

使用者點擊網站，傳request的流程
```md
> request -> web server -> rack -> ruby(erb)  
```

### 1-1、Rake定義   
Rake的定義是，只要做一個物件，這個物件可以call，並且回傳一個包含三個元素的陣列，三個元素包含  
(1) 網頁狀態(數字)  
(2) HTTP Header(hash)  
(3) HTTP Body(陣列)  
  
Ps. 只要支援call方法就可以，中介軟體也支援rack  
  
  
```md
> run Proc.new { |env|                      -> 用 run 去執行rack
>     [
>         500,                              -> http 狀態
>         {'Content-Type' => 'text/html'},  -> http header (這個也有其他型態'text/jpg/img'之類的)  
>         ['hellos rack']                   -> http body
>     ]
> }
```

### 1-2、http status 網頁常見狀態
資訊回應 (Informational responses, 100–199),  
成功回應 (Successful responses, 200–299),  
重定向 (Redirects, 300–399),  
用戶端錯誤 (Client errors, 400–499),  
伺服器端錯誤 (Server errors, 500–599).  

:::tip
**301、302的差異**  
301會永久轉傳網址，302只是暫時  
永久換網址用301  
暫時修網站用302  
:::




### 1-3、run回傳一個環境變數
環境變數會給一包東西 -> 對方瀏覽器的情況(ip位置、瀏覽器一些資訊......)
```md
> class Cat
>     def call(env)                               -> 這裡可以把一大包環境變數印出來
>         p env
>         [
>             500,
>             {'Content-Type' => 'text/html'},
>             ['hellos rack']
>         ]
>     end
> end
> kitty = Cat.new
> run kitty
```




2、中介軟體 middleware
------

中介軟體執行時間，是在run物件開始前，他除了可以當run的入口，也可以當插在入口前的東西    

```md
> class Backdoor
>     def initialize(app, who = "no one")
>         @app = app
>         @who = who
>     end
> 
>     def call(env)
>         status, header, body = @app.call(env)
> 
>         body << "<br /> hacked by #{@app}"
> 
>         [ status, header, body ]
>     end
> end
> 
> 
> use Backdoor, "YeeeeeeeChen"                  -> 執行中介軟體
> 
> 
> run Proc.new { |env|                          -> 入口
>     [
>         500,
>         {'Content-Type' => 'text/html'},
>         ['hellos rack']
>     ]
> }
```


### 2-1、use Backdoor
use Backdoor -> 在use的時候，會做下面兩件事，之後初始化的參數，就是下面那個(B)  
Ps. 我最想知道的東西，就是這個，中介變數傳上去的app變數，就是這裡來的  

```md
> B = Backdoor.new(R)
> run B
```



3、rails 路徑、架構
------

1. 瀏覽者的request傳進web server  
2. web server去找路徑  
  - (1) 有的話就去找相對應controller  
  - (2) 沒有的話丟一包狀態404的rack給瀏覽者  
3. 設定routes時，會controller會有相對應的action，在controller設定好action  
4. 最後進到action會要你去view頁面找相對應的erb檔案  

```md
>                                    action action ----->|   |       |     一堆的
>                                      ^    ^            |   |       |     table、table
>       |            |                 |    |            |   |       | SQL | - - - - - - - - - - - -  
> req ->| web server | -> routes -> Controller -> action---->| model | =>  |  DB (SQ Lite、MySQL)  |
>       |            |                             |     |   |       |     | - - - - - - - - - - - -
>                                                  |     |   |       |            | 
>  webview <-----------------------------  view <--------    |       |  <=========|       
>                                                            |       |     資料集(給SQL抓到的陣列)
>                                                            |       |     
```

### 3-1、routes setting

routes寫法，這一行的意思是，進入/about網址後，要去找pages這個controller，並且controller裡面要有ab這個action  
```rb
get "/about", to: "pages#ab"  
```


:::tip
如果使用者request網址，沒有這一條路徑，就會丟404+另外兩個狀態回去，如果有的話，就連到controller  
ex. /about -> 連到about相關的controller  
:::   

### 3-2、rails g/d controller
```md
> rails g controller pages   -> 生成一個指定controller檔案
> rails d controller pages   -> 刪除一個指定controller檔案
```


### 3-3、render html: "hi" 

這樣寫在action上，可以直接產出html的內容(不用連到view頁面)  
```md  
> def ab
>     render html: "hi"           -> 這樣畫面可以印出hi
> end
```  


:::tip
**首頁路徑寫法**  
get "/", to: "pages#home" -> 舊寫法  
root "pages#home" -> 新寫法  
::: 




4、Turbolinks
------

### 4-1、Turbolinks做了什麼？
Turbolinks會攔截畫面所有的a-link，如果發現此link是站內連結，會啟用這個方法   

當你在A頁面按下去B頁面連結的時候，會用ajax的方式去抓連結頁面的內容，會直接把B頁的內容，改成A頁的內容  
header換header、body換body、網址換網址  
  
### 4-2、Turbolinks為什麼要這樣做
為什麼要這樣換?因為這樣速度很快，傳統換頁，速度再怎麼快還是會閃一下，用turbo方法的話會超快  
Ps. 可以到f12的地方，點選網路狀態，會發現網頁載入狀態是xml  
  

### 4-3、Turbolinks缺點  
不過這樣會發生一件事，還記得JS的defer渲染嗎，當你每當換一頁，就會渲染一次  
不過現在Turbolinks不會有換頁行為，所以dom事件只會在網頁載入當下觸發一次，之後換網頁都不會觸發  
那其他頁面的dom元素要怎麼觸發呢?下面這一篇文章可以解決這個問題  
  
:::tip
rails 怎麼解決JS DOM無法觸發的問題  
[這篇一定要讀，解決了dom無法觸發的問題](https://www.writershelf.com/article/rails-turbolinks%E2%84%A2-5-%E6%B7%B1%E5%BA%A6%E7%A0%94%E7%A9%B6?locale=zh-TW)
::: 



5、a-link vs link_to
------

### 5-1、連結舊寫法 a-link
```HTML
<a href="/about">關於我們</a>
```

### 5-2、連結新寫法 link_to
使用這個好處是，link_to是一個方法，可以增加一些想要的參數進去
```md
> <%= link_to '聯絡我們', "/about" %> . -> a網址寫法
> <%= link_to '聯絡我們', about_path %> -> rails routes寫法 (相對路徑 -> 只會有/about)
> <%= link_to '聯絡我們', about_url %> -> rails routes寫法 (絕對路徑 -> 完整的網址(會員驗證信))
```

:::tip
**rails routes介紹**  
prefix = 自首、前綴  
Verb = 動詞  
URI Pattern  =  連結類型  
Controller#Action = action的指向controller  
用prefix_path當做路徑的好處在，想要換網址的時候，直接在路徑的地方換就好，會全換  
:::



### 5-3、.:format介紹

**queryString -> 查詢字串(這個很重要)**  
透過網址的參數列，你想要傳給controller的時候就會用到，在網址列的東西會被params抓下來
Ps. 一般的關鍵字查詢用get就可以做了，不需要用到post  


首先在view的erb打上，接下來就會印出搭配網址所抓到的一堆hash  
```rb
<%= params %>
```

假設網址為/contact  
```rb
{"controller"=>"pages", "action"=>"contact"}
```

假設網址為/contact?a=2&b=2  
```rb
{"a"=>"2", "b"=>"2", "controller"=>"pages", "action"=>"contact"}
```

假設網址為/contact.apple  
這個.apple就會被params抓下來，變成 "format"=>"apple"
```rb
{"controller"=>"pages", "action"=>"contact", "format"=>"apple"}
```

假設今天想要在路徑那邊設複雜一點  
```md
> get "/contact(.:query)", to: "pages#contact"   -> contact後面多加一個.:query
```

這樣params起來會變怎樣呢？網址為/contact  
```md
> {"controller"=>"pages", "action"=>"contact"}
```

上面看起來沒啥變對不對，我們現在幫網址多加一點料  
網址為/contact.apple.banana  
```md
> {"controller"=>"pages", "action"=>"contact", "query"=>"apple", "format"=>"banana"}

> 可以發現路徑設定的query，變成key值了
```
  
之後如果要設定搜尋表單，就可以利用指定的key值，吐給你value值，並把value塞進search bar欄位  



6、Model
------

### 6-1、rails g model

model規格定好後，就可以下指令了
```md
> rails g model WishList title description:text

> 會產生已向兩個檔案
>  create    db/migrate/20221114073329_create_wish_lists.rb     -> migrate
>  create    app/models/wish_list.rb                            -> model
```

建立好model後，記得實體化
```md
> rails db:migrate
```



:::danger
**產生model注意事項** 
(1) 專案名稱、model名稱一樣，會遇到model無法產生的問題  
(2) t.timestamps -> 這個會產生兩個欄位 -> created_at、updated_at  
:::


### 6-2、spring stop
:::info **Spring地雷**  
如果今天migrate完，又再migrate一次，會出現一行spring preloader(預先載入)的東西  
如果把專案丟到垃圾桶，但是 Spring 已經先幫你預載了，所以有機會在新開rails new，會卡住  
遇到這種狀況，把spring停下來，新專案就可以繼續了，那要怎麼停下來呢？  
輸入**spring stop**，就可以把spring停下來    
:::


### 6-3、model VS table

|model|table|補充|
|:-:|:-:|:-:|
|WishList|wish_list|在rails的格式|
|O|O|兩者都有最常見|
|O|X|購物車，只有model，不用table|
|X|O|rails內建table|

Ps. 做購物車，不用把資料存到table，可以使用sessions就好  



### 6-4、rails g migration 增加某個model的欄位

直接開一個新的migration，直接打在schema打資料是無效的!!  
Ps. 欄位名字不要亂取  

```shell
$ rails g migration add_online_to_wish_list
```

:::tip
**migration沒有存檔**  
要小心migration沒存檔，就db:migrate，會導致欄位沒有新增，就把migration用掉了  
解決方法，用db:rollback把migration倒回去，並存檔migration檔案，最後再一次db:migrate  
:::


### 6-5、rails db:rollback
:::info
把表格刪除，不建議直接對表格做，把表格砍掉有風險(因為資料會全部不見)  
:::

### 6-5、rails db:migrate:reset 
:::tip
這個指令可以把整個資料表倒掉，並且快速重新db:migrate  
:::



7、ps - process status 查看程序執行狀況
------

輸入 `ps` 可以看目前指定程序的執行狀況，假設今天想看local:3000的所有程式執行狀況，可以這樣輸入：

```shell
$ ps aux | grep 3000  
```

這個可以抓出local:3000執行中的程式，下面是正在執行的程式，36327是該執行序的編號：  
```shell
$ ps aux | grep 3000  

------
yee0526          36327   0.0  0.0 408104048   1456 s005  S+    2:15下午   0:00.00 grep 3000
```

使用kill把某個執行中的程式強制停止
```shell
$ kill 36327
```

Ps. 要找aux是啥可以打下面的指令 
```shell
$ man ps
```
