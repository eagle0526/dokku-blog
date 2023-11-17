---
title: ROR/HTML&CSS(10)
sidebar_label: "DAY19. [ASTROCamp] HTML&CSS-10"
description: HTML&CSS
last_update:
  date: 2022-11-01
keywords:
  - 5xRuby
  - HTML
  - CSS
sidebar_position: 20
---


1、按鈕旋轉 - 看影片把code補上
------

11/1 - 早上的影片
寫在手機才會出現的地方  max-width: 767


2、Bootstrap 教學
------
  
- Bootstrap的橫向排列，都是用flex製作的  
- 所有屬性的等級都會訂到6個 -> ex. border-0~5  
- 色彩分成八大主色  
- 文字色彩除了八大主色，還多了一些透明度的色彩  
- 有某個物件正在作用中，他都會套上一個active的屬性  
  
  
### 2-1、Gutters  
欄跟欄之間的距離  
設定在col上  
g-4 => 上下左右都有  
gx - 上下有  
gy - 左右有  

  
  
### 2-2、spacer是預設的文字大小 - 16px  
```md
> .ms-1 {
>   margin-left: ($spacer * .25) !important;  -> (16*0.25) = 4px
> }
```

### 2-3、viewpoint

vw - 視窗寬  
viewpoint width  
  
vh - 視窗高  
viewpoint height  
  
:::tip
如果vw設定100%，橫向就會出現卷軸    
:::

  
### 2-3、下面四個是跟導覽列比較有相關的  
- breadcrumb  
- list group  
- navbar  
- pagination  
  

如果一開始從bootstrap複製nav，他的container-fluid是全寬的意思，可以把fluid拿掉 - 不要全寬  
並且在把xl加上去，代表桌機的時候不要全寬，等到平板、手機的時候在全寬  
```md
> 最終改成這樣 <div class="container-xl">
```

可以去更改mx-auto、order-1來更改搜尋表單、nav、logo之間的相對位置  
```md
> <ul class="navbar-nav order-1 mb-2 mb-lg-0"></ul>
> <form class="d-flex mx-auto" role="search"></form>
```
  
  

可以修改已存在的bootstrap變數函式  
```md
> .xx {
>     var(--bs. "x")
> }
> 
> 後面"x"是預設值，如果後面x沒有，就會叫出--bs
```
  
  
3、用mac看手機網頁的開發者設定
------

0. 123.0.0.1 是本地端的測試ip位置  
1. 手機safari的進階 -> 網頁檢閱器要打開  
2. 手機和mac都要連結到同一個ip位置(不能為本地端ip)，兩者要同一區網ip  
3. 電腦查目前ip: 開啟網路設定，網路偏好設定裡面有ip  
4. 接著用vs code打開該檔案的測試頁面，再用區網ip把剛剛的本地端ip換掉  
5. 把最後用chrome網址可以轉成qrcode的設定，用手機拍qrcode就OK  
6. 最後打開mac的safari，並看向左上角中，開發中會有手機網頁開啟的狀態，並打開就ok  

  
  
4、有問題可以詢問老師公開的line
------
@csscoke



