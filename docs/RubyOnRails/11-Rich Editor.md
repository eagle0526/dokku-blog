---
title: ROR/Rich Editor 進階編輯器
sidebar_label: "11. [ROR] Rich Editor 進階編輯器"
description: Rich Editor
last_update:
  date: 2023-11-07
keywords:
  - ROR
  - RichEditor
sidebar_position: 11
---


:::tip
**前情提要：**         
讓使用者可以對一個物件進行進階的輸入訊息
:::



官方文件
------

[https://guides.rubyonrails.org/action_text_overview.html](https://guides.rubyonrails.org/action_text_overview.html)


### 安裝trix


```shell
$ bin/rails action_text:install

------
      append  app/javascript/application.js
      append  config/importmap.rb
      create  app/assets/stylesheets/actiontext.css
      append  app/assets/stylesheets/application.tailwind.css
      create  app/views/active_storage/blobs/_blob.html.erb
      create  app/views/layouts/action_text/contents/_content.html.erb
Ensure image_processing gem has been enabled so image uploads will work (remember to bundle!)
        gsub  Gemfile
       rails  railties:install:migrations FROM=active_storage,action_text
Copied migration 20230420061501_create_action_text_tables.action_text.rb from action_text
      invoke  test_unit
      create    test/fixtures/action_text/rich_texts.yml
```

### model設定

因為我想要讓使用者可以設定的欄位是description，因此下面才會是那樣寫
```rb
# model/house

class House < ApplicationRecord
  # description 的 rich editor 設定
  has_rich_text :description
end
```


### view設定

針對該欄位使用rich_text_area，就可以讓欄位套用該插件
```md
> <div>
>   <%= form_with model: house, url: url do |f| %>
>     <div class="my-4">
>       <%= f.label :description, '新增房屋描述' %>
>       <%= f.rich_text_area :description, class: 'form-field' %>
>     </div>
>   <% end %>
> </div>
```

