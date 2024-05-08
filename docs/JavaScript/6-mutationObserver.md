---
title: mutationObserver 的使用方式
sidebar_label: "6. [JavaScript] MutationObserver"
description: MutationObserver 範例講解
last_update:
  date: 2024-05-08
keywords:
  - JavaScript
  - mutationObserver
sidebar_position: 6
---


## mutationObserver 的使用方式

* 參考連結：https://developer.mozilla.org/zh-TW/docs/Web/API/MutationObserver

簡單說 mutationObserver 的用途，就是用在當你網站上某個區塊有變動時，可以直接偵測到該區塊的變動，變抓到變動區塊裡面的元素。   
ex. 某個網站有表單註冊的事件，不過該表單完成註冊後不會轉跳到完成註冊頁面，而是直接轉換該區塊的元素，顯示出感謝報名的文字，這時候就可以使用 mutation 的功能




### 參數介紹

```md
Property	      |  Description
childList	      | Set to true if additions and removals of the target node's child elements (including text nodes) are to be observed.
attributes	      | Set to true if mutations to target's attributes are to be observed.
characterData	      | Set to true if mutations to target's data are to be observed.
subtree	              | Set to true if mutations to not just target, but also target's descendants are to be observed.
attributeOldValue     | Set to true if attributes is set to true and target's attribute value before the mutation needs to be recorded.
characterDataOldValue | Set to true if characterData is set to true and target's data before the mutation needs to be recorded.
attributeFilter	      | Set to an array of attribute local names (without namespace) if not all attribute mutations need to be observed.
```



### 實際應用

範例網站 - https://events.lolinya.com.tw/content/consultation-472?utm_source=fb_consultation_teacher&utm_medium=ad&utm_campaign=fb_consultation_teacher_ad_teacher_20240412&utm_content=20240412


#### 介紹

這個客戶的報名網頁比較特別，因為他們的表單要滾動到一定的程度，表單才會跑出來，因此網頁監聽是使用 `scroll` 的方法來偵測，當我們偵測到 `frame` 這個元素的時候，代表表單跑出來了，這時候就可以抓到表單。  
再來這個客戶對於傳送事件有兩個需求：   
1. 點擊 `表單按鈕` 的時候，要送一個事件
2. 確定成功送出表單， `感謝區塊跳出來` 的時候，在送其他事件

#### 事件一
這邊不多作介紹，因為主要是要講 mutation

#### 事件二
1. 步驟一 : 這邊我們先抓到整個表單，因為等等是要偵測這個表單區塊的變動
```js
const form = frame[0]?.contentDocument.querySelector("form")
```
2. 步驟二 : 再來建立 `mutationObserver`

這個 `ele` 就是變動後的所有元素，想要的話可以把它全部印出來看是什麼樣子，而 index 就是這些元素的索引值。   
這邊主要的目的是，當我們抓到第一個元素`(index === 0)` + 這個元素有包含 `感謝您的填寫` 這幾個字，就送出完成註冊的事件。

```js
const mutationObserver = new MutationObserver(function (mutations) {
  mutations.forEach((ele, index) => {              
    if (index === 0 && ele.target.textContent.includes("感謝您的填寫")) {
      logger("==========success submit form===========")
      tracker('lead')
    }
  })
})
```

3. 步驟三 : 最後最重要的地方，呼叫 `mutationObserver.observe`

這邊就是啟動 `mutationObserver`，而裡面第一個參數就是一開始抓的 `form` 區塊，當這個區塊變動的時候，就會觸發步驟二的程式碼。  
而下面的 `childList` 參數，就是指變動後的所有元素，還有其他變數在一開始的時候有介紹了。
```js
mutationObserver.observe(form, {
  childList: true,            
})
```



#### 程式碼

```js
config.events = new Event({
  name: 'trackForm',
  trigger: pageView.path.contains('/content'),
  delay: 3000,
  fn: () => {
    /** use scroll event to determine whether form appears, if form is captured, do not capture it again */
    let isFrameCaptured = false
    window.addEventListener('scroll', () => {
      const frame = Array.from(document.querySelectorAll('iframe')).filter((frame) =>
        frame.contentDocument?.querySelector('button')
      )
      if (frames.length > 0 && !isFrameCaptured) {        
        const form = frame[0]?.contentDocument.querySelector("form")
        if (form) {
          // click form button event - event 1
          const submitBtn = form.querySelector("button")
          submitBtn.addEventListener('click', handleButtonClick)
          isFrameCaptured = true


          // transfer section after success submit - event 2
          const mutationObserver = new MutationObserver(function (mutations) {
            mutations.forEach((ele, index) => {              
              if (index === 0 && ele.target.textContent.includes("感謝您的填寫")) {
                logger("==========success submit form===========")
                tracker('lead')
              }
            })
          })
          mutationObserver.observe(form, {
            childList: true,            
          })
        }
      }
    })

    function handleButtonClick() {
      const form = this.closest('form')
      const name = form.querySelector('#section_1 input').value
      const phone = form.querySelector('#section_3 input').value
      const age = form.querySelector('#section_4 input').value
      const address = form.querySelector('#section_5 input').value
      const inquire = form.querySelector('#section_6 input').value
      const failMethod = form.querySelectorAll('#section_175 .css-uo0dhx')
      const hasClass = Array.from(failMethod).some((div) => div.innerHTML.includes('yyh2dn'))

      if (name && phone) {
        tracker('unitrack', 'add', { name, phone })
      }

      if (name && phone && age && address && inquire && hasClass) {
        tracker('完成註冊')
      }
    }
  },
})
```


