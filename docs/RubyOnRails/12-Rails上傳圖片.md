---
title: ROR/Rails上傳圖片-s3使用
sidebar_label: "12. [ROR] Rails上傳圖片-s3使用"
description: Rails上傳圖片-s3使用
last_update:
  date: 2023-11-07
keywords:
  - ROR
  - s3
sidebar_position: 12
---


:::tip
**前情提要：**         
圖片上傳 + s3 的使用
:::



安裝指令、一些插件
------

下面這個指令，會產生三個表單，一個是blobs、active_storage_attachments、active_storage_variant_records
```shell
$ bin/rails active_storage:install
-----
Copied migration 20230423073109_create_active_storage_tables.active_storage.rb from active_storage
-----



$ rails db:migrate
-----
== 20230423073109 CreateActiveStorageTables: migrating ========================
-- create_table(:active_storage_blobs, {:id=>:primary_key})
   -> 0.0023s
-- create_table(:active_storage_attachments, {:id=>:primary_key})
   -> 0.0010s
-- create_table(:active_storage_variant_records, {:id=>:primary_key})
   -> 0.0007s
== 20230423073109 CreateActiveStorageTables: migrated (0.0040s) ===============
-----
```




### 三個table個代表的意思

1. active_storage_blobs表格存儲了所有的附件（blobs），包括圖像、影片、聲音等等。每個blob都是一個文件的數據，由一個key來唯一標識。該表格中的id列是主鍵，key列是唯一索引。

2. active_storage_attachments表格存儲了附件和模型之間的關係。
每個附件可以被多個模型所關聯，每個關聯都有一個附件ID和一個模型ID。該表格中的id列是主鍵，name列存儲了附件的名稱，record_type列存儲了模型的類別名稱，record_id列存儲了模型的ID，blob_id列存儲了附件的ID。

3. active_storage_variant_records表格存儲了所有的附件變體（variants），也就是對原始附件進行裁剪、縮放等操作後生成的新附件。每個變體對應一個原始附件和一組轉換參數，例如縮放大小、裁剪位置等等。該表格中的id列是主鍵，blob_id列存儲了原始附件的ID，variation_digest列存儲了轉換參數的摘要。

這些表格是Active Storage用於儲存附件的重要組成部分，它們提供了一個強大的API，讓我們可以輕鬆地在Rails應用程序中管理和處理各種類型的附件。



使用者加上大頭貼(單圖)
------

### model部分
```rb
class User < ApplicationRecord

  has_one_attached :avatar

  # ... 省略
  
end
```

### 新增欄位部分，text_field 加上去

```md
<!-- views/devise/registrations/new.html.erb -->


<h2>Sign up</h2>

<%= form_for(resource, as: resource_name, url: registration_path(resource_name)) do |f| %>
  <%= render "devise/shared/error_messages", resource: resource %>

  // ...省略

  <div class="field">
    <%= f.label :avatar %><br />
    <%= f.file_field :avatar %>
  </div>

  // ...省略

<% end %>
```


### 圖片強參數

name這個強參數是之前新增的，這次新增的avatar，這樣新增後，才可以賞用者新增大頭貼
```rb
# app/controllers/users/registrations_controller.rb

def configure_sign_up_params
  devise_parameter_sanitizer.permit(:sign_up, keys: [:name, :avatar])
end
```


### 頁面顯示

接著我們把來判斷，如果今天使用者提供圖片，我們就顯示在navbar上


```md
> <nav>
>   ...省略
> 
>     <li><%= image_tag current_user.avatar %></li>
> 
>   ...省略
> </nav>
```

不過這樣會遇到一個問題，就是圖片大小太大了，我們來把它縮小一點


### 限制大小

首先先下載一個外掛，這個插件可以幫我們控制圖片大小

```shell
$ bundle add image_processing
```


下載好後，在model地方這樣設定，這樣顯示的圖片就不會太大張

```rb
class User < ApplicationRecord

  has_one_attached :avatar do |attachable|    
    # 依照實際大小     
    attachable.variant :thumb, resize_to_limit: [100, 100]

    # 每張圖會依照比例縮放
    attachable.variant :standard, resize_to_fill: [1200, 900]
  end

  # ...省略
end
```


### 圖片更新


如果今天沒有特別修改路徑的話，預設編輯資料的路徑應該是這樣
```md
<!-- view/shared/_navbar.html.erb -->

> <nav>
>   ...省略
> 
>     <li><%= link_to "更新暱稱", edit_user_registration_path(current_user.id), class:"mr-2 text-lg hover:text-major" %></li>
> 
>   ...省略
> 
> </nav>
```


有了路徑，我們來增加edit頁面應該要多兩個欄位

```md
<!-- views/devise/registrations/edit.html.erb -->

> <div>
>   ...省略
>   
>   <div class="field">
>     <%= f.label :name %><br />
>     <%= f.text_field :name %>
>   </div>
>   
>   <div class="field">
>     <%= f.label :name %><br />
>     <%= f.file_field :avatar %>
>   </div> 
> 
>   ...省略
> </div>
```

最後我們把強參數加上去，記得不論是新增使用者還是編輯使用者資料，都要記得有強參數這個東西喔！

```rb
# app/controllers/users/registrations_controller.rb

class Users::RegistrationsController < Devise::RegistrationsController

  before_action :configure_sign_up_params, only: [:create]

  # 這個是編輯資料的強參數方法   
  before_action :configure_account_update_params, only: [:update]


  # If you have extra params to permit, append them to the sanitizer.
  def configure_account_update_params
    devise_parameter_sanitizer.permit(:account_update, keys: [:name, :avatar])
  end
end
```




### 加上圖片大小限制

為了避免使用者都上傳過大的圖片，所以我們幫圖片上傳加上一點限制


參考檔案 - [https://github.com/igorkasyanchuk/active_storage_validations](https://github.com/igorkasyanchuk/active_storage_validations)


#### 安裝插件
```shell
$ bundle add active_storage_validations
```


#### model加上限制

```rb
class User < ApplicationRecord

  has_one_attached :avatar do |attachable|    
    attachable.variant :thumb, resize_to_limit: [100, 100]
  end

  # 這一行，加上去後，使用者之後無法上傳超過 1MB 的圖片
  validates :avatar, attached: true, size: {less_than: 1.megabytes, message: '不能超過1MB'} 

  #   ...省略
end

```




房屋加上各式圖片(多圖)
------


### model 設定

如果今天要設定一個房子可以有很多張圖片的話，這邊是這樣設定

```rb
class House < ApplicationRecord
  has_many_attached :images    
end
```


### view 設定

接著我們把使用者上傳圖片表單的地方加上去，很重要的一點， `multiple: true` 一定要加上去，要不然會運作不了喔
```md
views/houses/_form

> <%= form_with model: @house do |f| %>
>   <%= f.file_field :images, multiple: true, class: 'form-field' %>
> <% end %>
```

### controller 設定

最後，我們把強參數加上去，這樣才可以讓圖片新增進來

```rb
class HousesController < ApplicationController

  #...省略

  private
  def params_house    
    params.require(:house).permit(:title, :content, images: [])
  end
end
```




### 環境設定 -> 避免新增圖片的時候刪掉之前的圖

額外提醒，也超重要，這邊一定要加上這個，才能避免你每次在新增圖片的時候，前面新增的圖片都不見
```rb
# config/environments/development.rb 

config.active_storage.replace_on_assign_to_many = false
```

解決多圖片上傳的問題，這一篇文章有寫
https://stackoverflow.com/questions/61933070/how-to-edit-multiple-attached-images-using-activestorage-has-many-attached-in




增加圖片限制
------


### 安裝插件

一定要安裝下面這個插件，等等驗證圖片大小才能使用
```shell
$ bundle add active_storage_validations
```

文件在這邊 - [https://github.com/igorkasyanchuk/active_storage_validations](https://github.com/igorkasyanchuk/active_storage_validations)

### model 設定


```rb
class House < ApplicationRecord

  has_many_attached :images do |attachable|

    # 每張圖是固定size
    attachable.variant :thumb, resize_to_limit: [100, 100]
    attachable.variant :normal, resize_to_limit: [300, 300]

    # 每張圖會依照比例縮放
    attachable.variant :standard, resize_to_fill: [1200, 900]
    attachable.variant :first, resize_to_fill: [1200, 904]
  end

  validates :images, attached: true, size: {less_than: 1.megabytes, message: '不能超過1MB'} 

end
```



### view 設定



```md
views/houses/_form

> <%= form_with model: @house do |f| %>
>   <%= f.file_field :images, multiple: true, accept: 'image/png, image/gif, image/jpeg', maxlength: 1.megabytes, class: 'form-field' %>
> <% end %>
```















