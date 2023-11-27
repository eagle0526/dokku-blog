---
title: ROR/Rails 客製file_field上傳圖片的樣式
sidebar_label: "13. [ROR] Rails 客製file_field上傳圖片的樣式"
description: Rails 客製file_field上傳圖片的樣式
last_update:
  date: 2023-11-07
keywords:
  - ROR
sidebar_position: 13
---


:::tip  
**前情提要：**         
由於rails預設的 file_field 上傳圖片預設樣式太醜，所以今天來教學如何客製造自己想要的樣式
:::



先隱藏預設樣式
------


### 原本樣式

先給一個最基本的樣式，他的預設樣式很醜，所以等等我們來改變它
```html
<div class="my-4">
  <%= f.label :images, '廣告照片：' %>
  <%= f.file_field :images, multiple: true, accept: 'image/png, image/gif, image/jpeg', maxlength: 1.megabytes, class: 'form-field' %>
</div>
```



### 修改樣式

首先我們先把原本的樣式用hidden把他隱藏起來(用hidden把file_field包住)，並且在整個label地方設定一個按鈕css，裡面就可以設定你想要放的文字或圖片
```html
<div class="flex flex-wrap justify-center w-full py-20 border">
  <label class="btn-primary">
    <i class="mr-5 fa-solid fa-camera"></i>
    <p>上傳圖片</p>
    <span class="hidden">            
      <%= f.file_field :images, id: "fileUploader", multiple: true, accept: 'image/png, image/gif, image/jpeg', maxlength: 1.megabytes %>
    </span>
  </label>
</div>
```


### 增加上傳的文字

但是上面那樣設定會有問題，就是你把預設的東西都藏起來了，這樣別人把圖片上傳的時候，會看不到到底傳了哪些東西，因此我們要把上傳圖片的檔案名稱，顯示出來。  

那要怎麼做呢？我們在下面上傳圖片的區塊裡面設定div區塊，我們之後要用JS，把檔案名稱塞到這一區塊

```html
<div class="flex flex-wrap justify-center w-full py-20 border">
  <label class="btn-primary">
    <i class="mr-5 fa-solid fa-camera"></i>
    <p>上傳圖片</p>
    <span class="hidden">            
      <%= f.file_field :images, id: "fileUploader", multiple: true, accept: 'image/png, image/gif, image/jpeg', maxlength: 1.megabytes %>
    </span>
  </label>

  <!-- 這一區塊就是之後要檔案名稱塞進來的地方 -->
  <div class="w-full px-4 mt-3">
    <p id="filename" class="ml-2"></p>
  </div>
</div>
```

### 寫JS把文字塞進來

最後，我們寫一段JS

(1) 先抓到兩個東西，一個是上傳圖片的區塊，一個是要塞進檔案名稱的區塊
(2) 今天當我們觸發上傳檔案的動作時，我們先抓到所有上傳的檔案(fileList)
(3) 把塞進檔案名稱的區塊文字清空
(4) 接著來跑迴圈，因為上傳的檔案有可能是多個檔案，我們把每個檔案的名稱，用' / ' 分開，讓使用者更好確認自己上傳了幾個檔案
(5) 最後我們用appendChild，把檔案全部塞到指定的地方

```JS
<script>
  const fileUploader = document.getElementById('fileUploader');
  const filename = document.getElementById('filename');

  fileUploader.addEventListener('change', (event) => {
    const fileList = event.target.files;
    filename.innerHTML = '';

    for (let i = 0; i < fileList.length; i++) {
      const file = fileList[i];
      const span = document.createElement('span');
      span.innerText = file.name;
      
      if (i < fileList.length - 1) {
        span.innerText += ' / ';
      }
            
      filename.appendChild(span);
    }    
  });
</script>
```





### 用 stimulus JS 書寫

但是上面那樣是使用傳統的JS寫，我們今天改成rails的專用stimulus JS來寫：

(1) 最外層掛 controller
(2) 原本兩個用getElementById抓的目標，改用target來寫
(3) 上傳的動作改用data-action來寫

```html
_form.html.erb


<div class="w-full my-4" data-controller="house-image">
  <%# 客製的上傳圖片區塊 %>
  <div class="flex flex-wrap justify-center w-full py-20 border">
    <label class="btn-primary">
      <i class="mr-5 fa-solid fa-camera"></i>
      <p>上傳圖片</p>
      <span class="hidden">            
        <%= f.file_field :images, id: "fileUploader", multiple: true, accept: 'image/png, image/gif, image/jpeg', maxlength: 1.megabytes, 
                                  data: { house_image_target: "fileUploader", action: "change->house-image#upload" } %>
      </span>
    </label>
    <div class="w-full px-4 mt-3">
      <p id="filename" class="ml-2" data-house-image-target="filename"></p>
    </div>
  </div>
</div>
```


(4) 完成JS

```JS
// house_image_controller.js

import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = [ 'fileUploader', "filename" ]
  connect() {}

  upload(e) {    
    const fileList = e.target.files;    
    filename.innerHTML = '';

    for (let i = 0; i < fileList.length; i++) {
      const file = fileList[i];
      const span = document.createElement('span');
      span.innerText = file.name;
      
      if (i < fileList.length - 1) {
        span.innerText += ' / ';
      }           
      filename.appendChild(span);
    }    
  }
}

```


### 補上刪除圖片的區塊

前面都好了，還差一點點就是，如果今天使用者想編輯房子、刪除圖片，那要怎麼做呢？

(1) 設定路徑
(2) 在上傳圖片下面的區塊，把原本上傳的圖片都顯示出來，並在圖片右上角加上叉叉
(3) 在controller使用purge把圖片刪掉


#### 首先設定路徑
```rb
# routes.rb

namespace :admin do
  resources :users do
    member do
      delete '/images/:image_id' => 'users#destroy_image', as: :destroy_image
    end
      
    resources :houses, shallow: true do
      member do
        # 刪除房子的路徑在這邊
        delete '/images/:image_id' => 'houses#destroy_image', as: :destroy_image
      end
    end
  end
end
```


#### 設定HTML
```html
<%# 整個圖片區塊，包含上傳、刪除圖片 %>
<div class="w-full my-4" data-controller="house-image">
  <%# 客製的上傳圖片區塊 %>
  <div class="flex flex-wrap justify-center w-full py-20 border">
    <label class="btn-primary">
      <i class="mr-5 fa-solid fa-camera"></i>
      <p>上傳圖片</p>
      <span class="hidden">            
        <%= f.file_field :images, id: "fileUploader", multiple: true, accept: 'image/png, image/gif, image/jpeg', maxlength: 1.megabytes, 
                                  data: { house_image_target: "fileUploader", action: "change->house-image#upload" } %>
      </span>
    </label>
    <div class="w-full px-4 mt-3">
      <p id="filename" class="ml-2" data-house-image-target="filename"></p>
    </div>
  </div>


  <%# 刪除圖片的畫面在這 %>
  <div class="grid grid-cols-4 gap-2 mt-2">
    <% house.images.each do |image| %>
      <div class="relative">
        <% if image %>            
          <%= link_to destroy_image_admin_house_path(house, image), data: {turbo_method: "delete", confirm: "確定刪除?"} do %>            
            <div class= 'absolute top-0 right-0 flex items-center justify-center w-5 h-5 mt-2 mr-2 text-white bg-red-600 rounded-full '>
              <i class= "text-xs fa-solid fa-x"></i>
            </div>
          <% end %>
          <%= image_tag(image) %>
        <% end %>
      </div>
    <% end %>
  </div>
</div>
```

#### 設定controller

```rb
# admin/houses_controller.rb

module Admin
  class HousesController < ApplicationController
    before_action :authenticate_user!
    before_action :find_house, only: [:show, :edit, :update, :destroy, :destroy_image]
    

    # ...省略

    def destroy_image
      # render html: params      
      @house.images.find(params[:image_id]).purge
      redirect_to admin_house_path(@house), alert: "成功刪除圖片"

    end

    private

    def params_house      
      params.require(:house).permit(:title, :description, :tel, :address, :owner, images: [])
    end

    def find_house
      @house = House.find(params[:id])
    end

  end
end
```












參考文件：
(1) [https://stackoverflow.com/questions/55183950/rails-file-field-input-as-a-button](https://stackoverflow.com/questions/55183950/rails-file-field-input-as-a-button)   
(2) [https://stackoverflow.com/questions/33028255/rails-4-how-to-apply-custom-css-to-rails-form-file-field](https://stackoverflow.com/questions/33028255/rails-4-how-to-apply-custom-css-to-rails-form-file-field)   
(3) [https://www.ruby-forum.com/t/how-change-css-style-fo-file-field-tag/245244/3](https://www.ruby-forum.com/t/how-change-css-style-fo-file-field-tag/245244/3)