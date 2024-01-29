---
title: 163 - 修改 tiktok UTM 參數
sidebar_label: "3. [UTM] 修改 tiktok UTM 參數"
description: 修改 tiktok UTM 參數
last_update:
  date: 2024-01-29
keywords:
  - tagtoo
  - tiktok UTM
sidebar_position: 3
---

### 詳細描述
"小三美日的Tiktok目錄想要請RD幫忙把UTM直接帶在目錄中各個商品的「到達頁面」上，UTM內容如下：
```md
> utm_source=TikTok&utm_medium=TikTok_app&utm_campaign=TikTok_vsa"
```
**也就是說，小三美日的商品連結要帶上這一串的參數**

### 解決方法
1. 先詢問 op 目前是用哪一種 feed，依照現在使用的 feed 來做修改

下面三個是目前使用的 feed，可以看出是使用 v1，並且渠道是 tiktok，後面會有分 value，是因為有些渠道不能一次匯入這麼多，所以才會限制一次能匯入的數量
```md
https://api.tagtoo.com.tw/v1/feed/tiktok/163?begin_value=0&scope=5000
https://api.tagtoo.com.tw/v1/feed/tiktok/163?begin_value=5001&scope=5000
https://api.tagtoo.com.tw/v1/feed/tiktok/163?begin_value=10001&scope=5000
```


2. 打開 api 檔案，找到 cpa_config 的資料夾

```py
# api/configs/cpa_config.py

# ...省略
def cpa_s3(o, pb, **kwargs):
    if pb == 475:
        o.append_param({
            'utm_source': 'TikTok',
            'utm_medium': 'TikTok_app',
            'utm_campaign': 'TikTok_vsa',
        })
    return o


# ...省略
cpa_builder.register(163, cpa_s3, 'www.s3.com.tw')    
```


上面的段落解析：   
(1) 首先介紹 pb 是啥，pb 就是各種的渠道，像是 `fb - 71、google - 66、yahoo - 468、tiktok - 475`，如果今天想要針對某個渠道增加 utm，在判斷式那邊修改就好  
(2) cpa_builder 是寫好的 fc，照著前人的格式寫就可以觸發

Ps. line 的 utm 設置不在這邊


3. 本機確認是否有更改完成

最後我們就是要來確認有沒有加上 UTM 了，這裡我們用 tiktok 來確認，當我們改好之後，我們可以直接在網址輸入：
<!-- https://api.tagtoo.com.tw/v1/feed/tiktok/163 -->
```md
> http://localhost:8000/v1/feed/tiktok/163
```
因為 tiktok 是 CSV 檔案的關係，所以我們會直接得到一份檔案，我們打開檔案，並且找到有一個欄位叫做 `link`，下面的內容大概長這樣：
```md
> 檔案上的內容
> https://api.tagtoo.com.tw/v1/ad/click?pb=475&ind=08&u=https%3A%2F%2Fwww.s3.com.tw%2FTC%2FPDContent.aspx%3Fyano%3D15287066&ad=163

> 把上面的改成下面這樣
> http://localhost:8000/v1/ad/click?pb=475&ind=08&u=https%3A%2F%2Fwww.s3.com.tw%2FTC%2FPDContent.aspx%3Fyano%3D15287066&ad=163
```

前綴改成本機端後，我們就可以實際測試，最後連到的連結到底有沒有加上 UTM 了！！！


PS. 如果今天是用 facebook 或 google，都可以找到 link 的欄位，只要把前綴改成本機端，就可以確認到底有沒有成功修改 UTM


### PR、需求編號

[api]: https://github.com/Tagtoo/api/pull/1741