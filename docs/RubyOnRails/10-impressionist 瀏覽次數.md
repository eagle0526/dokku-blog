---
title: ROR/Rails 物件瀏覽次數(impressionist)
sidebar_label: "10. [ROR] Rails 物件瀏覽次數(impressionist)"
description: swiper carousel
last_update:
  date: 2023-11-07
keywords:
  - ROR
  - impressionist
sidebar_position: 10
---


:::tip
**前情提要：**         
我想要查看每一間房子被不論是訪客還是使用者瀏覽的次數
:::




安裝impressionist
------

```shell
$ bundle add impressionist
```


創建impressionist專用table

```shell
$ rails g impressionist
$ rails db:migrate
```

### 在想要查看瀏覽數的物件上做設定

在指定的controller、action加上這一段
```rb
# controllers/houses_controller.rb

class HousesController < ApplicationController

  def show
    # 計算瀏覽次數
    impressionist(@house)

    #...忽略
  end
end
```


在指定的model加上這一段
```rb
class House < ApplicationRecord  
  is_impressionable
end
```




### 顯示瀏覽次數


```md
> 同一個人一直刷新就可以增加好多次
> @house.impressionist_count(:filter => :all)

> 一個ip只顯示一次瀏覽量  
> @house.impressionist_count(:filter => :session_hash)

> 一段時間內的瀏覽量
> @house.impressionist_count(:start_date: 1.day.ago, end_date: 1.day.from_now)
```


參考連結 ： https://github.com/charlotte-ruby/impressionist



### 增加 counter_cache

:::tip **counter_cache 是什麼？**

counter_cache 是 Rails 中一種方便的緩存技術，可以使關聯對象的計數器被緩存在主對像中。
例如，在一個BLOG應用程式中，每篇文章都有很多評論。每次獲取文章時都要計算該文章的評論數，這會導致查詢頻繁和性能下降。
使用 counter_cache 技術，我們可以將文章的評論計數器緩存在文章中，以避免在每次獲取文章時重新計算評論數。
:::


以我們現在的案例，要怎麼增加 counter_cache 呢？

#### 1. 在 house 的 model 加上一個欄位

```shell
$ rails g migration add_counter_cache_to_house
```

```rb
# migration/add_counter_cache_to_house.rb
class AddCounterCacheToHouse < ActiveRecord::Migration[7.0]
  def change    
    add_column :houses, :houses_count, :integer, default: 0
  end
end
```

```shell
$ rails db:migrate
```

#### 2. 在 house 的 model 增加一行

```rb
# house
class House < ApplicationRecord
  is_impressionable :counter_cache => true, :column_name => :houses_count

  # ... 略
end
```

這樣之後就可以把紀錄存取在 house 的這個欄位中



要怎麼確認 `counter cache` 有運作呢？只要有在 LOGS 中看到類似這種的CODE，就代表有在運作
```shell
17:01:37 web.1  |   House Update All (1.1ms)  UPDATE "houses" SET "houses_count" = COALESCE("houses_count", 0) + $1 WHERE "houses"."id" = $2  [["houses_count", 1], ["id", 13]]
```



