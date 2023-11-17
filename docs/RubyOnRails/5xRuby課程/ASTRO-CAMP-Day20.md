---
title: ROR/Vue
sidebar_label: "DAY20. [ASTROCamp] Vue"
description: Vue
last_update:
  date: 2022-11-03
keywords:
  - 5xRuby
  - Vue
sidebar_position: 21
---


vue是什麼
------

- Vue 的核心功能是聲明式渲染：通過擴展於標準HTML 的模板語法，我們可以根據JavaScript 的狀態來描述HTML 應該是什麼樣子的。當狀態改變時，HTML 會自動更新。  
   
- **能在改變時觸發更新的狀態被認為是響應式的。**在Vue 中，響應式狀態被保存在組件中。在示例代碼中，傳遞給 createApp() 的對像是一個組件。  




### 專案製作

在終端機打上這一行，可以取用老師的原始檔案
```md
> npx degit kurotanshi/shopping-cart-vite3 [自訂專案名稱]
```

下載下來後，補上下面這幾句，就可以開始實作了
```md
> cd [自訂專案名稱]
> npm i
> npm run dev
> 
> 可以線上跑vue的[網站](https://stackblitz.com/)
```


### vue檔案有三個部分
  
- template - 放html  
- script - 放js  
- style - 放css  


setup() 是所有component所有說及的起點，該語法不能更改
const title = ref('hello')


:::info
用CDN載入插件的時候，記得不要使用latest，這樣很容易壞掉  
:::


:::tip
vite.config.hs - 是一個開發環境設定檔，就是啟用local::host的東西  
:::


***


ref、computed、watch是什麼
------
- 被ref包起來的東西，只要裡面的資料被更新了，就會被vue偵測到，並重新渲染在畫面上    
- 用ref拉 + fetch json檔案回來，可以避免狀態不同步  

在專案開始前，記得先用import把ref匯入
```md
> import { ref } form 'vue'
```

用ref包起來的格式，都要用XX.value來裝
```md
> const list = ref([])
> 
> list.value = res
```

### 老師補述 computed

ref你就想像說，今天把一個變數丟進去了，然後我根據這個值得更新，他都可以即時反映在畫面上，但是如果今天你有另外一個值，就要使用另外一個功能 -> computed  
  
Ps.如果今天有兩個值以上變動，還是使用ref的話，會變得很麻煩，要使用到另外一個功能 - watch  
ref你就想像成他是用來宣告一個被觀察的變數、(就像我們平常宣告let、const一樣)，不過他是一個增強版的變數  
  
一個是computed - computed的用意是說，在一個fc裡面，他可以去觀察到，這個fc裡面的資料，當你有兩個值以上的值被更新，computed可以幫你同步更新  
  

### computed舉個例子
```md
> const list = ref([]) 
> const searchName = ref("")
> const filterList = computed(() => {
>     return list.value.filter(d => d.name.includes(searchName.value))
> })
> 
> 可以看到filterList用computed的時候，裡面有兩個被ref宣告的變數(list、computed)
```


### 另外一個例子 - 組合firstName、lastName

首先給HTML
```md
> <template>
>     <input type="text" class="border" v-model="firstName">
>     <input type="text" class="border" v-model="lastName">
> 
>     <p>{{ fullName }}</p>                    # 會印出在input輸入欄位輸入的組合字               
> </template>
```

用computed的方式，讓input同步
```md
> const firstName = ref("")
> const lastName = ref("")
> const fullName = conputed(() => `${firstName.value} ${lastName.value}`)
```



用watch的方式，讓input同步，但使用watch的話，會變得很囉唆
```md
> const firstName = ref("")
> const lastName = ref("")
> const fullName = ref("")
> 
> watch([firstName, lastName], ([v1, v2]) => {
>     fullName.value = `${v1}${v2}`
> })
```




***

### v-for = for i in list
vue裡面的for迴圈寫法
```md
> <div class="col-sm-2" v-for="cat in list">

> 後面v-for的意思，是用for把"list"裡面的資料全部印出來(cat就是i之類的)
```


### v-for是仿造for迴圈的作法

```md
> v-for="cat in list" 這一句的意思 同一個陣列會重複跑幾次

> for (let cat in list) {
>     console.log(cat)       # 0,1,2,3,4,5
> }
> 
> 
> for (let cat of list) {
>     console.log(cat)       # 陣列所有元素一行一行印出來
> }
```



:::tip
換圖片用{{}}雙層括號會壞掉，因為雙層括號是用來給innner content的  
----  
要用這個，圖片是屬於屬性 -> 因此換要用v-bind:src  
v-bind是屬於vue的語法黑魔法 -> 如果以後聽到v-directive，v-屬性，就是在說這東西  
不過v-bind大家懶得寫，就是直接在屬性前面寫個: 就可以  
:src="cat.cover"  
:::



:::tip
如果今天要做假資料 - v-for="cat in 4" -> 可以直接生成4隻貓  
:::



v-on:click
------
上層的部分增加一個點擊事件，點擊按鈕的時候，就呼叫addToCart事件
```md
> v-on:click="addToCart(cat)" - 普遍的寫法
> @click="addToCart(cat)" - 更短的寫法
```
  
  
:::tip
nextTick的概念跟setTimout一樣，非同步回調  
:::
  
  
  
  
  
#### 今天的更改完成的檔案，在day20-vue的裡面，如果想要初始檔，直接執行這一段就可以  
```md
> npx degit kurotanshi/shopping-cart-vite3 [自訂專案名稱]
```
