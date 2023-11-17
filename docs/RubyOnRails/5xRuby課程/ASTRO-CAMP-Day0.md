---
title: ROR/Ruby
sidebar_label: "DAY0. [ASTROCamp] Ruby"
description: RUBY安裝
last_update:
  date: 2022-10-02
keywords:
  - 5xRuby
  - Ruby
sidebar_position: 1
---

### ruby安裝套件相關指令

```md
ruby || rvm、gem、bundle
        #rvm管理ruby的工具
        #gem管理套件的工具
        #bundle install - 找一個gemfile檔案 開啟此檔案裡面的所有套件
```


## bundle 和 gem 的差異

如果現在有一個狀況

- Ａ套件需要Ｂ套件 1.0.0 版本
- Ｂ套件需要Ｃ套件 1.0.0 版本
- Ｄ套件需要Ｂ套件 2.0.0 版本

這樣會導致一個問題，今天如果多個檔案, 要的B套件不同的版本,就有可能發生錯誤


### gemfile解決版本相容的問題

gemfile => gem "a", 1.0.0 版本  gem "c". 2.1.3版本

gemfile.lock => 會產生一個“多個檔案相依性”的檔案

bundle install 就是呼叫眾多的gem  =>  一堆gem組合起來才是bundle

### package.json 是前端的gemfile   

＝＝＝gem 版本代表的意思＝＝＝

```md
gem 'xxx' 3.1.7
          3 => major   => 完全不同的產品
          1 => minor   => 有可能會壞掉        
          7 => patch   => 增加不太重要的功能  （可以隨便更新來用）
```




:::tip gem ( ~> 、 > ) 差異
gem 'xxx', '~> 3.1.7'   3.2.0 不會裝   
gem 'xxx', '>3.1.7'     3.2.0   會裝  
~的意思是  會去裝比較安全的版本  所以今天如果改動的是 patch 就會安裝     
:::
