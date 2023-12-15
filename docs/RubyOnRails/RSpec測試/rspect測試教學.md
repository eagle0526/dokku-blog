---
title: ROR/RSpec
sidebar_label: "RSpec 測試"
description: RSpec 教學
last_update:
  date: 2023-12-08
keywords:
  - 5xRuby
  - Ruby
sidebar_position: 1
---


## 生出專案

```shell
$ rails new classroom
```

### 安裝外掛

```rb
group :test, :development do
  gem "rspec-rails"
  gem "rails-controller-testing"
end
```


```shell
$ bundle install
```


### 安裝 rspec 相關檔案
```shell
$ rails generate rspec:install
```



### 修改檔案


```rb
# config/application.rb
module Classroom
  class Application < Rails::Application
    # Settings in config/environments/* take precedence over those specified here.
    # Application configuration should go into files in config/initializers
    # -- all .rb files in that directory are automatically loaded.

    config.generators.assets = false
    config.generators.helper = false
  end
end
```


### 產生 controller

```shell
$ rails g controller  courses
```


## 正式開始寫 controller 測試

1. 首先要確認測試資料庫是否乾淨，如果裡面有存資料，記得先輸入此行指令，重建測試資料庫 `RAILS_ENV=test rails db:reset`
2. 在 rails 7 中，進入 test console 環境的指令是這個 `rails console --environment=test`


```rb
# spec/controllers/courses_controller_spec.rb

RSpec.describe CoursesController, type: :controller do
  describe "GET index" do
    it "assigns @courses and render template" do
      course1 = Course.create(title: "foo", description: "bar")
      course2 = Course.create(title: "bar", description: "foo")

      get :index

      expect(assigns[:courses]).to eq([course1, course2])
      expect(response).to render_template("index")
    end
  end
end
```























