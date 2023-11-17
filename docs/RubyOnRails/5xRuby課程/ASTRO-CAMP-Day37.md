---
title: ROR/Ruby
sidebar_label: "DAY37. [ASTROCamp] Ruby-14"
description: Ruby
last_update:
  date: 2022-12-02
keywords:
  - 5xRuby
  - Ruby
sidebar_position: 38
---



訂位系統
------


怎麼建立商店的seat
怎麼安排客人的座位排在鄰座
即時選位系統
多人同時間
保留多久
訂完位子，會把位子狀態鎖住，

action table






rails的concern資料夾
------

為什麼要有一個concern資料夾，裡面為什麼只會有一個.keep資料夾，而資料裡面啥都沒有？  
原因就是這個keep就是為了讓concern這個資料夾可以推上去，還記得前面在git單元的時候，有說過如果    
今天沒有資料夾裡面是空的，就會推不上GitHub，所以要放一個空資料夾在那邊  

不過為什麼concern這麼奇特，裡面會啥資料都沒有呢？在提到它的功用前，要先知道一個概念，就是關注點分離  

:::tip
> ### SOC關注點分離(引用WIKI)  
> 在計算機科學中，關注點分離（Separation of concerns，SoC），是將計算機程序分隔為不同部份的設計原則。   
> 每一部份會有各自的關注焦點。關注焦點是影響計算機程式程式碼的一組資訊。關注焦點可以像是將程式碼優化過的硬體    
> 細節一般，或者像實例化類別的名稱一樣具體。展現關注點分離設計的程序被稱為模組化程序。  
> 模組化程度，也就是區分關注焦點，通過將資訊封裝在具有明確界面的程序代碼段落中。封裝是一種資訊隱藏(電腦科學)  
> 手段。資訊系統中的分層設計是關注點分離的另一個實施例(例如，表示層，業務邏輯層，數據訪問層，持久數據層)。  
:::


這個concern資料夾，就是SOC中的concern，也就是該資料夾裡面就是關注點分離的檔案，不過看了WIKI的說明
一定覺得還是不知道他在幹嘛，其實就是把注意力放在重點上面就好，那些比較重複、瑣碎的資料，我可以抽出來
放到這個資料夾裡面。

Ex. 當我們在寫模組的時候，其實就是一種關注點分離。假如我們今天呼叫那個fc，其實不用太在意那個fc怎麼運作的
如果我太在意那些細節，我就要去關注fc的每一行每一個字，這樣太累了，所以這就像是JS我們在fc做抽象化一樣


### 實際舉例

今天有兩個Model，一個是book，一個是food，兩個model都有price的欄位   
現在要幫兩個model都加上一個功能，就是買書的時候都要加上稅額，因此這樣寫  

首先是book 
```md
> book.rb

> class Book < ApplicationRecord
>   def price_with_tax
>     price * 1.05 
>   end
> end
```
再來是food 
```md
> food.rb

> class food < ApplicationRecord
>   def price_with_tax
>     price * 1.05 
>   end
> end
```


還記得如果有兩個一樣的方法，可以抽出來用模組的概念寫嗎！？
因此我們就可以把兩個共通的方法寫在一個檔案裡面，並且把該檔案放在concern底下

像這樣，我們幫這個檔案取名為taxable.rb
```md
> module Taxable
>   def price_with_tax
>     price * 1.05
>   end
> end
```


這邊寫完後，就可以幫兩個model掛上模組了
```md
> food.rb/book.rb

> class food/book < ApplicationRecord
>   include Taxable 
> end
```



### refactoring 重構
不過concern不是只有這樣喔！！！
還記得重構這個東西嗎！？我們曾經在測試的時候提到這個概念

他的概念就是在不改變外部行為下，對內部行為調整，不個不叫做重寫，叫做重構    
簡單說就是你寫好測試後，你不改變測試的code，只改變被測試的code  

concern也是一種重構的寫法   
如果今天在model都寫很像的"方法"，可以用寫模組的方法，丟到model裡面的concern資料夾裡面   


現在我們來常識寫一段，幫剛剛的Taxable模組，加上concern的功能
Ps. 一般的模組不會有我們等等要做的功能
```md
> module Taxable
>   extend ActiveSupport::Concern      # 引入concern(我要讓這個模組變成concern的模組)
>   
>   include do                         # 當有人使用include，會做的事情
>     validates :title, presence       # 這樣等等food、book的title都會變成必填
>   end                                # 原本要在兩個model都寫驗證，現在只要寫一遍
>
>   class_method do                    # 放在這個裡面的方法都會變成類別方法
>     def hi                           # 你今天只要include此模組，那個model就會得到這個類別方法
>       p "hi!"
>     end
>   end
>
>   def price_with_tax
>     price * 1.05
>   end
> end
```

Ps. 一般沒有放在class_method裡面的方法，是實體方法喔！！




longly operator &.
------

如果今天新增一個cat類別，並且裡面給他一個方法，後面再new出一個新實體，並且呼叫sleep方法
```md
> class cat
>   def eat
>   end 
> end
> 
> kitty = Cat.new
> kitty.sleep
```
這個在我們之前學的，正常報錯是會說沒有sleep這個方法，不過如果你真的拿去跑程式，會發現他的錯誤是這個
private method 'sleep' called

為什麼是沒有這個私有方法呢？原因就是因為這個cat上面他有一個預設的sleep方法


#### 邏輯短路


不過如果有一天，這kitty是nil怎麼辦，有時候會遇到new出來的實體是nil  
我們會在nil方法執行sleep方法，然後就會掉了  

上面說法有可能不太清楚，換個說法好了！我們在寫rails的時候，很常發生我們去撈資料的時候，撈不到東西就會   
得到一個nil，然後我們對這個沒撈到的東西執行方法，就會得到錯誤   

所以通常我們會這樣寫判斷，如果有kitty存在，我們再執行方法   
這樣寫後就不會壞了  
```md
> if kitty
>   kitty.sleep
> end
```

不過上面那樣寫有點麻煩，比較短的寫法就是這個
這個是邏輯運算，假設前面得到false，後面就會執行
```md
> kitty && kitty.sleep
```

這個東西就叫做邏輯短路，再舉個完整例子好了  
如果今天kitty是存在的，後面的hi就會印出來   
```md
> kitty = 1
> kitty && p "hi"     印出hi
```

如果今天kitty是不存在的，後面的hi就不會執行
```md   
> kitty = nil
> kitty && p "hi"     啥都沒印出，因位kitty是false
```

:::tip
> 最終階段的縮寫就是一開始說的那個符號  
> kitty&.eat    
:::

```js
const hero = {
    name: "cc", 
    hi: () => console.log("hi")
}
```



#### JS的optional chaining

今天創造一個物件，他有一個方法在裡面
```md
const hero = {
    hi: () => console.log("hi")
}

hero.hi
```

如果今天hero是false，就不會執行hi的函式
```md
> let hero = nil    
> hero.hi               # X 不執行
```

如果前面的結果是true，才繼續執行後面的hi函式
如果hero是false，那就直接不執行
```md
> hero && hero.hi()
```

:::tip
最終階段的縮寫就是這個    
hero?.hi()    
:::


***

下午課程


discord notification
------

如果今天想做一個，當你訂單成功建立的時候，自動傳一則訊息到你的discord，這樣是不是很方便  
因此，我們要來學習webhook的概念，核心在做的事，就是對一個標的物，用post的方式，把資料傳給discord or 其他的平台  


首先到discord的頻道上，取得頻道專屬的webhook    
https://discord.com/api/webhooks/1048118318327939092/5HsIlwrYEY3tE0CIGe5kGF1m07QUSr6CZC7vOmqZj4InKdg04zJfVRQNYTnaPYwTRpn8   



[另外到這個頁面](https://www.digitalocean.com/community/tutorials/how-to-use-discord-webhooks-to-get-notifications-for-your-website-status-on-ubuntu-18-04)，取得一段POST的code
接著複製一段，POST的傳送方法，主要就是curl那一段    
```md
> if [[ "$status_code" -ne 200 ]] ; then

>     # POST request to Discord Webhook with the domain name and the HTTP status code
>     curl -H "Content-Type: application/json" -X POST -d '{"content":"'"${domain}returned: ${status_code}"'"}'  $url

> else
>     echo "${website} is running!"
> fi
```

我們主要看的就是這一段
```md
> curl -H "Content-Type: application/json" -X POST -d '{"content":"'"${domain}returned: ${status_code}"'"}'  $url  # 原始 
> curl -H "Content-Type: application/json" -X POST -d '{"content": "hi"}'   # 縮減過後的

> -x 是送的方式
> -d 是送的內容
```

我們實際在終端打這個指令試試，這樣直接打在終端機上，就可以主動傳訊息到Discord了  
Ps. 標點符號要注意，要不然送不出去
```md
> curl -H "Content-Type: application/json" -X POST -d '{"content": "hi"}' https://discord.com/api/webhooks/1048118318327939092/5HsIlwrYEY3tE0CIGe5kGF1m07QUSr6CZC7vOmqZj4InKdg04zJfVRQNYTnaPYwTRpn8
```



### rails and webhook

把剛剛那個東西跟ROR結合，只要把curl那一串放在你想要放的地方，像是我想要新增新商店的時候，就傳一則訊息
```md
> def create
>   @store = Store.new(store_params)
>   
>   if @store.save
>   
>     system %Q(curl -H "Content-Type: application/json" -X POST -d '{"content": "robot from rails"}' https://discord.com/api/webhooks/1048118318327939092/5HsIlwrYEY3tE0CIGe5kGF1m07QUSr6CZC7vOmqZj4InKdg04zJfVRQNYTnaPYwTRpn8)
    
>     redirect_to stores_path, notice: "成功新增商店"
>   else
>     render :new
>   end
>   
> end
```


不過這樣直接放在controller裡面怪怪的，所以我們把它放在另外一個地方，我們新開一個檔案，叫做discord_service.rb
在開始前之前，要先提到一個字詞，叫做PORO

:::tip
PORO = Plain Old Ruby Object  
意思是純物件(完全沒有繼承某個物件)
:::




接著我們就來寫一個完全沒有繼承某個物件的純物件
Ps. 這個檔案在day37裡面
```md
> discord_service.rb

> class DiscordService
>   def initialize(webhook: nil)
>     if webhook.nil?
>       @webhook = ENV["DISCORD_WEBHOOK"]
>     else
>       @webhook = webhook
>     end
>   end
> 
>   def notify(message)
>     content_type = "Content-Type: application/json"
>     method = "POST"
>     body = {content: message}.to_json
>     system %Q(curl -H "#{content_type}" -X #{method} -d '#{body}' #{@webhook}")
>   end
> 
> end
```



上面寫好後，就可以實際在想要增加通知的地方寫

```md
> stores_controller.rb

> DiscordService.new.notify("新增書本 #{@book.title}")
```

剛剛那一段不算在MVC架構裡面，所以我們會放在另一個地方，直接在最外面開一個service的資料夾，
把功能都放在service裡面，這樣就可以很適當的整理code



15:14
active job
------
剛剛有一問題，我們在送訊息到discord的時候，都會等個幾秒網站才動，這樣如果今天送一堆不同的訊息
不就會卡到爆嗎！？所以我們今天要來設定讓通知為背景運作

Ps. 這個背景工作就是ruby的非同步執行


首先創造一個工作
```md
> rails g job notify_discord
```

下完指令後，會產生這個檔案
```md
> jobs/notify_discord_job.rb

> class NotifyDiscordJob < ApplicationJob
>   queue_as :default                           # 去queue排隊
>    
>   def perform(*args)
>     # Do something later
>   end
> end
```



接著我們來嘗試使用，我們新增一家商店後，網站會馬上執行，但是五秒後傳訊息給我
使用job.set，就可以達成目的，前面的NotifyDiscordJob是job那邊的class
```md
> if store.save
>   NotifyDiscordJob.set(wait: 5.seconds).perform_later
> end
```


知道可以這麼做以後，我們來把剛剛的discord訊息，寫到剛剛新創的job裡面，並帶一些實體變數進去
我把perform_later帶引數進去，等等discord的訊息就可以接到
```md
> stores_controller.rb

> def create
>   @store = Store.new(store_params)
>   
>   if @store.save
>   
>     NotifyDiscordJob.set(wait: 5.seconds).perform_later("商店新增： #{@store.title}")
>   
>     redirect_to stores_path, notice: "新增成功"
>   else
>     render :new
>   end
> end
```

再來處理job，剛剛有傳訊息過來，所以我們這邊就要把service寫好的discord訊息寫到這邊
```md
> class NotifyDiscordJob < ApplicationJob
>   queue_as :default
> 
>   def perform(message)                        # 這個message會傳剛剛的"商店新增.."進來
>     puts "-" * 30
>     DiscordService.new.notify(message)        # 這一段就是我們原先寫的service
>     puts "-" * 30
> 
>     # Do something later
>   end
> end
```


:::tip
**訂票網站 - 演唱會、高鐵票**  
不是只有寄信會用到延遲功能，在create的時候也會用到    
像是遇到大流量網站，如果今天一堆人搶，要使用 `active job`，來延遲    
因為票是有限的，通常都是卡在create那一階段，如果沒用active job    
會全部卡在一個地方    
:::


#### sidekiq
這些工作會放在Server的記憶體裡面，放在記憶體的優點就是速度快，但是缺點就是如果電腦重開，就被清空了  

因此有一些套件、轉接頭，可以解決這件事，像是sidekiq就可以   
sidekiq是類似記憶體的一種東西，如果今天Server重開，sidekiq會把記憶體中的工作寫一份到另外一個檔案    
等你重開後，記憶體會讀一份回來，因此就算記憶體清空，那些工作也還是會存在  



16:00
拖拉效果
------

[sortable js](https://sortablejs.github.io/Sortable/)
```md
import Sortable form "sortablejs"

export default class extends Controller {
    connect() {
        new Sortable(this.element)
    }
}
```






真正改變它的效果
```md
onEnd: function() {

}
```




rails.ajax

rails有自己接收promise的方法


axios -> 處理fetch

act_as_list -> 處理排序的標籤

