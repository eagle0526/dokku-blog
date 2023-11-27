---
sidebar_position: 1
---

0、前言
------
:::tip
**寫此篇文章的動機**  
為了練習Redis的功能，因此直接做了一個比較簡單的Project，此專案用到ROR、Redis、TailsWind CSS    
:::



1、Redis優點、特性
------
在實作之前先介紹一下Redis的特性，Redis是非關聯式資料庫。

:::tip
所有Redis資料都存放在記憶體中，從而實現低延遲和高輸送量的資料存取。 與傳統資料庫不同，記憶體內資料儲存不需要存取磁碟，從而將引擎延遲縮減到微秒。      
因此，記憶體內資料儲存能夠支援更大規模的操作，而且回應時間更快。 這項優勢提供超快速的效能，平均讀取和寫入操作時間低於一毫秒，並支援每秒百萬個操作。  
FROM [AMAZON](https://aws.amazon.com/tw/redis/)   
:::

### 1-1、Redis優點

#### 1-1-1、速度
而最大特性就是速度很快，Redis能讀的速度是110000次/s，寫的速度是81000次/s。     
PS. 雖然查到說Redis可以這麼快，不過沒碰上大量資料之前，完全無感XD    

#### 1-1-2、豐富的數據類型
Redis支持二進制案例的Strings、Lists、Hashes、Sets及Ordered Sets 數據類型操作。

#### 1-1-3、原子
Redis的所有操作都是原子性的，意思就是要馬成功執行要馬失敗完全不執行

#### 1-1-4、其他特別特性
Redis還支持publish/subscribe、通知、key expire(過期)等等特性。



2、開始實作
------

介紹完Redis的特性了，那就先把實作的大綱、步驟先寫出來吧～    
     
### 2-1、實作大綱 

#### 2-1-1、環境設定
1、先產生一個存放連結資料的資料庫 - model取名為Link  
2、設定一個方法，當使用者輸入完整連結後，會亂數產生一個slug，這個slug會是之後的短網址其中的字串  
3、接著安裝Redis，等等準備用Redis Hash來儲存使用者輸入的連結內容     
4、把路徑設定好  


#### 2-1-2、使用者模擬
把東西都設定好後，可以模擬使用者輸入連結後的狀況     
1、使用者輸入連結 - https://google.com/  
2、輸入的時候會使用SecureRandom方法，直接亂數產生slug，對應到使用者輸入的連結 - 此時會先存進資料庫   
3、接著再存進Redis Hash裡面  
4、當使用者點擊剛剛亂數產生的短連結時，會先去Redis找有沒有這一筆資料(我們在步驟3，有存一份在Redis)   
5、如果沒有在Redis找到(可以用expire把redis的資料先刪掉，好處等等會說)，去資料庫找該短連結，並寫一份到Redis裡面   



### 2-2、實作細節


#### 2-2-1、設定環境

1、新增一個rails專案
```shell
$ rails new _6.1.7_ ShortLink
```

2、新增Link model - model的column包含以下幾項    
(1) url(使用者輸入完整連結)  
(2) slug(每個連結對應的亂數)     
(3) clicked(點擊次數)    

```shell
$ rails g model Link url slug clicked:integer
```


3、新增亂數方法
```rb
# Link Model

def generate_slug
  self.slug = SecureRandom.uuid[0..5] if self.slug.nil? || self.slug.empty?
end
```

4、使用before_validation，讓link在產生前就會用到此方法、再加上幾個url、slug限制
```rb
# Link Model
validates :url, format: URI::regexp(%w[http https])
validates_uniqueness_of :slug

before_validation :generate_slug

def generate_slug
  self.slug = SecureRandom.uuid[0..5] if self.slug.nil? || self.slug.empty?
end
```


5、安裝Redis

用brew來安裝redis
```shell
$ brew install redis
```

啟動redis環境
```shell
$ redis-server
```

啟動redis console
```shell
$ redis-cli
```

新增一個資料夾，設定全域變數$redis，這樣之後就不用每個action都要重新設定
```rb
# config/initializes/redis.rb

$redis = Redis.new
```


6、制定redis hash的設定

```md
> (1) key - 每個連結的redis key是資料庫連結的id-數字:slug
> (2) 第一個hash - url https://fintechrich.com
> (3) 第二個hash - short http://localhost:3000/shorts/6365a2
> (4) 第三個hash - slug 6365a2
> (5) 第四個hash - visit 0

> 格式設定好後，等等用hset新增
> Ex. hset id-43:6365a2 url https://fintechrich.com short http://localhost:3000/shorts/6365a2 slug 6365a2 visit 0
```

7、路徑設定  

```rb
# routes.rb

resources :links
```



#### 2-2-2、完整連結新增到資料庫、redis
環境、方法都設定好後，就可以來完成controller

首先我們來把Link_controller補完    
   
(1) 當使用者輸入連結的時候，先儲存一份在資料庫中  
(2) 此時link因為資料庫已經存入並且亂數產生slug，把此slug設定為一個變數   
(3) 用hset存一份到redis裡面，hash裡面包含url、short、slug、visit，並且key的格式為"資料庫的id-數字:slug"  

```rb
# LinkController

before_action :link_params, only: [:create]

def new
   @link = Link.new
end

def create    
  @link = Link.new(link_params)
  if @link.save
      
    @shortcode = @link.slug

    # 用hash新增 - hset id-43:6365a2 url https://fintechrich.com short http://localhost:3000/shorts/6365a2 slug 6365a2 visit 0
    hash = {"url" => @link.url, "short" => "#{ENV["WEB_DOMAIN"]}/shorts/#{@link.slug}", "slug" => @link.slug, "visit" => 0}
    $redis.hset("id-#{@link.id}:#{@link.slug}", hash)      
  end
  redirect_to root_path
end  

private
def link_params
  params.require(:link).permit(:url, :slug)
end
```


#### 2-2-3、點擊連結後，增加點擊次數

資料庫、redis都有數據後，就可以來設計，當使用者點到連結，連結被點擊的次數就要加一  

1、首先設定路徑  
```rb
# routes.rb

resources :links do 
  collection do
    get :increase_visit
  end
end
```

2、設定view   

```md
<!-- index.erb.html -->

> <table class="w-full text-base text-left text-gray-500 dark:text-gray-400 table-auto">
>     <thead class="text-base text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400">
>     <tr>
>         <th class="px-6 py-3">redis的key</th>
>         <th class="px-6 py-3">原本的連結</th>
>         <th class="px-6 py-3">短連結</th>
>         <th class="px-6 py-3">slug</th>
>         <th class="px-6 py-3">點擊次數</th>                          
>         <th class="px-6 py-3">redis刪除</th>        
>     </tr>
>     </thead>
>     <% @keys.each do |k| %>    
>     <tbody>
>     <tr>
>         <td class="px-6 py-4"><%= k %></td>
>         <td class="px-6 py-4"><%= $redis.hgetall(k)["url"] %></td>
>         <td class="px-6 py-4"><%= link_to $redis.hgetall(k)["short"], increase_visit_links_path(k) %></td>
>         <td class="px-6 py-4"><%= $redis.hgetall(k)["slug"] %></td>        
>         <td class="px-6 py-4"><%= link_to $redis.hgetall(k)["visit"], increase_visit_links_path(k) %></td>     
>         <td class="px-6 py-4"><%= link_to 'delete', expire_key_links_path(k) %></td>   
>     </tr>
>     </tbody>
>     <% end %>  
> </table>
```

重點在這一行，先用hgetall抓到該則短連結，並且如果使用者點到此則短連結，會連到剛剛新建的action那邊，因此等等要來寫action
```md
> <td class="px-6 py-4"><%= link_to $redis.hgetall(k)["short"], increase_visit_links_path(k) %></td>
```

2、設定controller  

假如今天點擊到此路徑，先用hgetall抓到該短連結，並且用hincrby方法，針對visit這個field增加值，點一次增加1，並且最後連到該網頁
```rb
# LinksController

def increase_visit
  
  if $redis.hgetall(params[:format])["url"]
    $redis.hincrby(params[:format], "visit", 1)
    redirect_to $redis.hgetall(params[:format])["url"]
  end
end
```




#### 2-2-4、用expire刪除redis資料

如果今天資料有超級多筆，這些資料一直存在記憶體中(搞不好有些連結根本沒啥點擊)，這樣會造成儲存成本上升(存在資料庫比較便宜)，因此我們可以使用redis提供的expire功能，來讓存在redis中的key過期(刪除)。  

常用的手法可能是，在創建出該短連結的時候，就直接把expire時間加上去(7天後過期)，如果7天內這個連結被點過，這個7天就會重置，這樣就可以達到熱資一直存在redis裡面，冷資料等到有人點擊的時候才會再出現。   

:::tip
用expire是刪除記憶體的資料喔！不是刪除資料庫裡面的，所以今天如果不小心刪掉redis的某一筆資料，直接從資料庫拿就好  
:::


1、路徑設定  

不過現在因為我想要直接體驗expire的功能，所以我直接設定一個連結，當我點下這個連結，該key就會失效，所以我們先來設定路徑  

```rb
# routes.rb

resources :links do 
  collection do

    # 這一行 
    get :expire_key         
        
    get :increase_visit
  end
end
```



2、設定view  

當使用者點擊該連結時，會觸發這個action   
```md
> <td class="px-6 py-4"><%= link_to 'delete', expire_key_links_path(k) %></td>
```

3、設定controller  

觸發action後，會把抓到的key值，直接設定expire為0秒，這樣就可以直接把key刪掉
```rb
# LinksController

def expire_key    
  # redis = Redis.new
  key = params[:format]
  
  $redis.expire(key, 0)
  redirect_to root_path
end
```


#### 2-2-5、redis資料到期，從資料庫抓出該資料，並寫一份到redis

接著來實作，假設今天在redis的key到期了，使用者想要點擊該短連結，就會從原本記憶體拿資料，變成先去資料庫拿資料，並且寫一份到記憶體裡面，這樣redis又會出現這一個key了

1、設定路徑
```rb
# routes.rb

resources :shorts, only: [:show]
```

2、設定view
```md
<!-- index.html.erb -->

> <table class="w-full text-base text-left text-gray-500 dark:text-gray-400">
>     <thead class="text-base text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400">
>     <tr>
>         <th class="px-6 py-3">id</th>
>         <th class="px-6 py-3">使用者輸入連結</th>
>         <th class="px-6 py-3">slug</th>
>         <th class="px-6 py-3">點擊次數</th>        
>         <th class="px-6 py-3">短連結</th>        
>         <th class="px-6 py-3">資料庫刪除</th>        
>     </tr>
>     </thead>
>     <% @links.each do |link| %>
>     <tbody>
>     <tr>
>         <td class="px-6 py-4"><%= link.id %></td>
>         <td class="px-6 py-4"><%= link.url %></td>
>         <td class="px-6 py-4"><%= link.slug %></td>
>         <td class="px-6 py-4"><%= link.clicked %></td>        
>         <td class="px-6 py-4"><%= link_to link.short, link.short, data:{ turbolinks: "false" } %></td>        
>         <td class="px-6 py-4"><%= link_to 'delete', link_path(link), data: {method: "delete"} %></td>   
>     </tr>
>     </tbody>
>     <% end %>  
> </table>
```

重點在這一行，點下去後，會觸發show的action
```md
> <td class="px-6 py-4"><%= link_to link.short, link.short, data:{ turbolinks: "false" } %></td>
```

Ps. 會把turbolinks關掉的原因，等等會提到   


3、完成controller

這邊有幾件事情要設定   
(1) 點擊該連結的時候，先看一下redis有沒有該連結  
(2) 如果redis有此連結的話，直接幫此連結的visit + 1   
(3) 如果redis裡面沒有的話，從資料庫裡面找出該連結(@link)   
(4) 再寫一份到redis裡面  
(5) 並且幫此redis的visit + 1   

```rb
class ShortsController < ApplicationController

  before_action :find_link, only: [:show]  

  def show    

    # 確認redis中有沒有這個key(用*可以拿到相關字串)
    key = $redis.keys("*#{params[:id]}").first
    if key    
      # 如果redis裡面有存此id，幫visit + 1
      $redis.hincrby(key, "visit", 1)

      redirect_to $redis.hgetall(key)["url"]
    else
      # 如果redis中沒有這個key，就寫一份到redis裡面
      hash = {"url" => @link.url, "short" => "#{ENV["WEB_DOMAIN"]}/shorts/#{@link.slug}", "slug" => @link.slug, "visit" => @link.clicked}
      $redis.hset("id-#{@link.id}:#{@link.slug}", hash)

      # 寫一份到redis後，直接幫點擊數 + 1
      $redis.hincrby("id-#{@link.id}:#{@link.slug}", "visit", 1)

      redirect_to @link.url
    end

  end
  private

  # 這個就是資料庫中的link
  def find_link
    @link = Link.find_by!(slug: params[:id])
  end

end
```

:::tip
如果今天沒有把這個連結的turbolink關掉的話，會導致visit + 2喔！所以記得要關掉   
:::


#### 2-2-6、redis的key到期的時候，把資料匯進資料庫

如果今天使用者數量很大的話，如果要新增使用者點擊連結的次數，就要先撈出該連結，並且讓點擊次數增+1，最後再寫進資料庫，這樣一直操作資料庫，量一大的時候就會造成資料庫有負擔，因此我們把增加點擊次數的操作，改成由redis來操作，並且在一個固定的時間點，再定時匯入資料庫，這樣不就可以用最小的負擔，達成增加短連結的點擊次數！  


1、改寫controller

原本expire_key的action，只有把key刪除，先在我們在刪除前，還要把資料匯進資料庫
```rb
def expire_key
  
  # 抓到傳過來的key
  key = params[:format]

  # 把link找出來，並且把快取中的visit回填資料庫中
  id = key[3..4]    
  @link = Link.find_by!(id: id)
  @link.clicked = $redis.hgetall(key)["visit"].to_i
  @link.save
          
  # 刪除快取
  $redis.expire(key, 0)
  redirect_to root_path
end
```

#### 2-2-7、額外補充：一次把redis中的資料，匯入資料庫

前面我們提到，可以訂一個時間，用active job把所有redis中的資料回填給資料庫，不過今天我先改成直接按一個按鈕，就可以把所有資料回填給資料庫(這樣比較直觀)  

1、設定路徑
```rb
resources :links do 
  collection do
    get :expire_key        
    get :increase_visit

    # 這個路徑
    get :import_clicked
  end
end
```


2、設定view  
   
新增一個按鈕，只要使用者點下去，會觸發import_clicked這個action   
```md
<!-- index.html.erb -->

> <div class="text-center">
>   <button><%= link_to '把redis瀏覽次數，匯入資料庫', import_clicked_links_path, class: "second-btn" %></button>
> </div>
```

3、設定controller  
```rb
# LinksController

def import_clicked

  # 列出redis中所有的key
  keys = $redis.keys("*")

  # 根據redis的key，把link所有visit數灌回資料庫的點擊數
  keys.each do |key|
    # 找到資料庫中對應的slug
    link = Link.find_by!(slug: $redis.hgetall(key)["slug"])

    # 資料庫中的連結點擊數 = redis中的visit數量
    link.clicked = $redis.hgetall(key)["visit"].to_i
    link.save
  end
  redirect_to root_path
end
```


### 2-3、增加連結的UTM

由於之前是數位行銷出生的，因此很常用到UTM這個連結參數，於是就順手增加了UTM的設定，不過由於這一段code比較多，所以只會說大概的運作模式，要看整段code就麻煩到github看拉～   
Ps. rails的JS是使用stimulus JS   

(1) 使用者增加填上連結
(2) UTM會有五個額外的欄位，分別是campaign、medium、source、term、content
(3) 使用者在這五個欄位填寫資料的時候，會直接把對應的字串，直接加到一開始填寫的欄位


舉個例子：
```md
> link field : https://google.com
> campaign field : 20230205
> medium field : social_post
> source field : facebook
> term field : yee
> content field : 3c
> 
> 最後的完整連結 ： https://google.com?utm_campaign=20230205?utm_medium=social_post?utm_source=facebook?utm_term=yee?utm_content=3c
```

3、小記
------
這樣就完成了rails、redis的短連結系統！再次總結一下這個小專案做了哪些事情～   
   
(1) 使用者新增的網址，可以生成一段短網址(短網址後段是亂數生成)   
(2) 生成的短網址會先存進資料庫   
(3) 再來存一份到Redis裡面  
(4) 使用者點擊短連結，會先從redis找該筆資料  
(5) 如果今天redis沒有該筆資料，而使用者還是點擊了，會去資料庫找連結，並且寫一份到redis裡面   
(6) 使用者點擊短連結後，會增加點擊次數   
(7) UTM功能  



### 3-1、使用到的工具
Rails On Ruby、Redis、Tailwind CSS


### 3-2、GitHub連結
[https://github.com/eagle0526/rails_redis_shortlink]https://github.com/eagle0526/rails_redis_shortlink

   
   
   
   







