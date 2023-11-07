---
sidebar_position: 1
---

如何使用 local preview
------

我們會在 local 開發的時候，會需要把頁面上的所 `HOST` 的那張 `JavaScript` 導入到 `local 端`。 用 `ModHeader` 這個小工具是因為，為了導到 `local` 開發，我們 `local` 可以起一個簡單的 server，來去 `Host` 現在這一家客戶的 `muffet` 檔案。

一般來說，我們在 muffet 的 repo 中，當我們發一個 pr merge 到 main 分支，並且跑完 `CI/CD` 後，會 `compile` 成一個 `json` 檔案，我們會 `Host` 在這個地方，這個東西我們放在 `GCP` 上面(在 cloud storage 裡面)。

> ---  
> 當我在本地開發時，會需要把 `https://ecs.tagtoo.co/js/1165.js` 導入到我們的本地，他會偵測 `chrome browser` 所發出的 `HTTP`， `request` 中的 `header`  
>    
> ---  
{:. block-tip}

### ModHeader設定
開啟 `ModHeader` chrome 插件，並點擊 `+` 選項，接著點擊 `Redirect URL` 選項，這時候會出現兩個 field，左邊的欄位輸入 `https://ecs.tagtoo.co/js/1165.js`，右邊的欄位輸入 `http://localhost:3000/1165.js`，再來進入這個文件 [muffet repo](https://github.com/Tagtoo/muffet/wiki/%5BMuffet%5D-%E5%B7%A5%E4%BD%9C%E6%89%8B%E5%86%8A)。

Ps. 一開始 ModHeader 設定好，去 dev-tool 中的 network，會看到 `307 Internal Redirect`，他會去找 local 下 1165 的 js 檔案

### muffet 檔案設定
照著教學，先 `clone` 下來 muffet 的檔案，接著把進入資料夾後， 輸入 `npm install` 把一些插件下載下來後，就可以輸入以下指令， `npm run preview 1665`，再來就可以打開客戶的網站、打開 `dev tool`，進入到 `console` 的分頁，就可以開始開發了！



Ps. 上面輸入的連結都是實際案例，實際上要根據客戶的 `ec ID` 來改變連結輸入，以上面的狀況來說 `1165` 就是不同客戶都會有不同的數字  
Ps. 記得一定要用 npm ，用 yarn 會噴錯誤