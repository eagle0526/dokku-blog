---
title: tagtoo/CSP 白名單設定
sidebar_label: "12. [tagtoo] CSP白名單"
description: google 強化轉換
last_update:
  date: 2024-02-17
keywords:
  - Muffet
tags:
  - Muffet
sidebar_position: 12
---



## CSP 白名單

如果今天客戶的網站有開 CSP 設定，導致 google 事件無法正常送出，導致客戶要你提供一份白名單，讓他們網站那邊可以設定，不要讓網站阻擋事件送出，這邊提供一份名單。


Ps. 示範案例是之前合作的客戶 - https://www.dlsveg.com.tw/，到他的 network 找到 `https://www.dlsveg.com.tw/`，點下去可以看到 `Content-Security-Policy:` 有一大串，就是這一大串

以下是直接複製那一串上來：
```md
default-src blob: 'unsafe-inline' 'self'; worker-src 'self' blob:; connect-src *.smartweb.tw www.google-analytics.com *.google.com *.google.com.tw *.bing.com s.yimg.com *.doubleclick.net www.facebook.com www.googletagmanager.com www.google.com.hk www.google.com.sg fcm.googleapis.com r.lr-in-prod.com cdn.lr-in-prod.com/LogRocket.min.js pagead2.googlesyndication.com https://api-bdc.net/data/client-ip accounts.google.com/gsi/client pta-api.tagtoo.co api.tagtoo.com.tw ecs.tagtoo.co event.tagtoo.co clouderrorreporting.googleapis.com/v1beta1/projects/tagtoo-ecs/events:report *.tiktok.com analytics.pangle-ads.com/api/v2/pangle_pixel match.adsrvr.org/track/cmf/generic receiver-feed.tagtoo.com.tw/products/ 'self'; script-src 'nonce-94e3b777ca7e65b6b22b945c5b34dbbe' *.smartweb.tw static.smartweb.tw *.googletagmanager.com *.facebook.net api.facebook.com apis.google.com tagmanager.google.com syndication.twitter.com www.googleadservices.com googleads.g.doubleclick.net unpkg.com/popper.js@1.12.6/dist/umd/popper.js *.google-analytics.com cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js *.yimg.com *.bing.com sp.analytics.yahoo.com ajax.googleapis.com www.youtube.com s.ytimg.com cdnjs.cloudflare.com/ajax/libs/popper.js/1.12.9/umd/popper.min.js cse.google.com www.google.com platform.twitter.com d.line-scdn.net/n/line_tag/public/release/v1/lt.js tpc.googlesyndication.com cdn.ampproject.org www.instagram.com cdn.lr-in-prod.com accounts.google.com/gsi/client ad.tagtoo.co cdn.tagtoo.com.tw api.tagtoo.com.tw track.tagtoo.com.tw ecs.tagtoo.co *.tiktok.com analytics.pangle-ads.com/api/v2/pangle_pixel 'unsafe-eval' 'unsafe-inline' 'self'; style-src *.smartweb.tw use.fontawesome.com fonts.googleapis.com *.google.com static.smartweb.tw cdn.ampproject.org https://www.googletagmanager.com/debug/badge.css accounts.google.com/gsi/style 'unsafe-inline' 'self'; font-src *.smartweb.tw use.fontawesome.com fonts.gstatic.com data: 'self'; child-src *.smartweb.tw *.twitter.com *.facebook.com *.google.com bid.g.doubleclick.net www.youtube.com tpc.googlesyndication.com www.googletagmanager.com www.instagram.com td.doubleclick.net ad.tagtoo.co tracking.shopmarketplacenetwork.com api.tagtoo.com.tw track.tagtoo.com.tw ecs.tagtoo.co api.ezorderly.com 'self'; img-src *.smartweb.tw *.google-analytics.com *.google.com *.google.com.tw google.pl googleads.g.doubleclick.net lh3.googleusercontent.com www.google.cn www.google.co.in www.google.co.jp www.google.co.nz www.google.com www.google.com.hk www.google.com.mx www.google.com.my www.google.com.pa www.google.com.ph www.google.com.sg www.google.com.vn www.google.fr www.googleadservices.com www.googleapis.com www.googletagmanager.com *.bing.com *.facebook.com *.g.doubleclick.net *.gstatic.com *.ytimg.com connect.facebook.net cx.atdmt.com scdn.line-apps.com static.smartweb.tw static.xx.fbcdn.net stats.g.doubleclick.net tr.line.me *.analytics.yahoo.com *.privacysandbox.googleadservices.com www.f-counter.net pixel.tagtoo.co track.tagtoo.co track.tagtoo.com.tw api.tagtoo.com.tw ecs.tagtoo.co match.adsrvr.org 'self' data:
```


### 整理過後的白名單

把上面那一大串整理過後，就是以下：

```md
*.google-analytics.com
*.google.com
*.google.com.tw
*.googletagmanager.com
*.googleapis.com
*.googlesyndication.com
*.googleadservices.com
*.privacysandbox.googleadservices.com
*.doubleclick.net
*.g.doubleclick.net
google.pl
www.google.cn
www.google.co.in
www.google.co.jp
www.google.co.nz
www.google.com
www.google.com.hk
www.google.com.mx
www.google.com.my
www.google.com.pa
www.google.com.ph
www.google.com.sg
www.google.com.vn
www.google.fr
accounts.google.com/gsi/client
accounts.google.com/gsi/style
lh3.googleusercontent.com
clouderrorreporting.googleapis.com/v1beta1/projects/tagtoo-ecs/events:report
https://www.googletagmanager.com/debug/badge.css
```


### 另一種方式

#### 安裝 gtag
```js
function installGtag() {
  const installed = !!(window.gtag && document.querySelector('script[src*="/gtag/js"]'))  
  if (!installed) {
    window.dataLayer = window.dataLayer || []    
    window.gtag = function gtag() {
      dataLayer.push(arguments)
    }
    gtag('js', new Date())    
  }
}
```


用上面那一串來找，或者用 安裝 gtag 的方式，隨便找一段 conversion，把
```js
<!-- Google tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=AW-11202968721"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'AW-11202968721');
</script>
```
這一段拿去發送，就可以知道 google ads 到底送了哪些東西