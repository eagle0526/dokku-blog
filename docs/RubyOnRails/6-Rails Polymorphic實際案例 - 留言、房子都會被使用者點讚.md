---
title: ROR/Rails Polymorphic實際案例 - 留言、房子都會被使用者點讚
sidebar_label: "6. [ROR] Rails Polymorphic實際案例 - 留言、房子都會被使用者點讚"
description: Polymorphic-Association 資料庫多型介紹
last_update:
  date: 2023-11-07
keywords:
  - ROR
  - Polymorphic-Association
sidebar_position: 6
---


:::tip
**前情提要：Polymorphic 多型的實際應用**         

一間房子可以被使用者點讚/倒讚  
一則留言可以被使用者點讚/倒讚  
:::



### 控制留言喜歡/不喜歡按鈕的JS頁面
```md
comment_like_controller.js

> import { Controller } from "@hotwired/stimulus"
> import { fetchWithoutParams } from "../controllers/lib/fetcher"
> import { library, dom } from '@fortawesome/fontawesome-svg-core'
> import { faThumbsUp as regularThumbsUp , faThumbsDown as  regularThumbsUDown } from '@fortawesome/free-regular-svg-icons'
> import { faThumbsUp as solidThumbsUP, faThumbsDown as solidThumbsDown } from '@fortawesome/free-solid-svg-icons' 
> 
> export default class extends Controller {
> 
>   static targets = [ 'thumbsUp', "thumbsDown", "commentLikeCount", "commentDislikeCount" ]
> 
>   initialize(){    
>     library.add(regularThumbsUp, regularThumbsUDown, solidThumbsUP, solidThumbsDown, )
>   }
>   connect() {
>     dom.watch()
> 
>     
>     const commentLikeState = this.element.dataset.commentLikeState
>     if (commentLikeState === 'true') {
>       // 如果是正讚，讓正讚是solid，倒讚是regular
>       this.likeButtonSolid()
>       this.dislikeButtonRegular()
> 
>     } else if (commentLikeState === 'false') {
>       this.dislikeButtonSolid()
>       this.likeButtonRegular()
>     }
>     
>   }
> 
>   like() {        
> 
>     const commentId = this.element.dataset.commentId
>     const token = document.querySelector("meta[name='csrf-token']").content    
> 
>     // // 使用者點擊正讚後，傳到後端的東西
>     fetch((`/comments/${commentId}/like`), {
>       method: "PATCH",
>       headers: {
>         "X-CSRF-Token": token
>       }
>     })
>     .then((resp) => {
>       return resp.json()
>     })
>     .then(({status, likeCommentCount, dislikeCommentCount}) => {
>       
>       if (status === "liked comment") {
>         this.likeButtonSolid()
>       } else if (status === "delete comment like_state") {        
>         this.likeButtonRegular()
>       } else {
>         // 倒讚Regular        
>         this.dislikeButtonRegular()
>         // 正讚Solid
>         this.likeButtonSolid()
>       }
> 
>       this.setTextContent(likeCommentCount, dislikeCommentCount)
> 
>     })
>     .catch((err) => {
>       console.log(err);
>     })    
> 
>   }
> 
>   dislike() {        
> 
>     const commentId = this.element.dataset.commentId
>     const token = document.querySelector("meta[name='csrf-token']").content
> 
>     // 使用者點擊倒讚後，傳到後端的東西
>     fetch((`/comments/${commentId}/dislike`), {
>       method: "PATCH",
>       headers: {
>         "X-CSRF-Token": token
>       }
>     })
>     .then((resp) => {
>       return resp.json()
>     })
>     .then(({status, likeCommentCount, dislikeCommentCount}) => {
>       
>       if (status === "disliked comment") {
>         this.dislikeButtonSolid()
>       } else if (status === "delete comment like_state") {        
>         this.dislikeButtonRegular()
>       } else {
>         // 正讚Regular
>         this.likeButtonRegular()
>         // 倒讚Solid
>         this.dislikeButtonSolid()
>       }
> 
>       this.setTextContent(likeCommentCount, dislikeCommentCount)
> 
>     })
>     .catch((err) => {
>       console.log(err);
>     })
>   }
> 
> 
>   // 正讚solid
>   likeButtonSolid() {
>     this.thumbsUpTarget.classList.add("fa-solid")
>     this.thumbsUpTarget.classList.remove("fa-regular")
>   }
> 
>   // 正讚regular
>   likeButtonRegular() {
>    this.thumbsUpTarget.classList.remove("fa-solid")
>    this.thumbsUpTarget.classList.add("fa-regular")
>   }
> 
>   // 倒讚solid
>   dislikeButtonSolid() {
>     this.thumbsDownTarget.classList.add("fa-solid")
>     this.thumbsDownTarget.classList.remove("fa-regular")
>   }
> 
>   // 倒讚regular
>   dislikeButtonRegular(){
>     this.thumbsDownTarget.classList.add("fa-regular")
>     this.thumbsDownTarget.classList.remove("fa-solid")
>   }
> 
> 
> 
>   setTextContent(likeCommentCount, dislikeCommentCount) {
>     this.commentLikeCountTarget.textContent = likeCommentCount
>     this.commentDislikeCountTarget.textContent = dislikeCommentCount
>   }
> 
> }
```


### 控制房子喜歡/不喜歡按鈕的js頁面
```md
house_like_controller.js

> import { Controller } from "@hotwired/stimulus"
> import { library, dom } from '@fortawesome/fontawesome-svg-core'
> import { faHeartCrack } from '@fortawesome/free-solid-svg-icons' 
> import { faHeart } from '@fortawesome/free-regular-svg-icons'
> 
> export default class extends Controller {
> 
>   static targets = [ 'heart', "heartCrack", "houseLikeCount" ]
> 
>   initialize(){    
>     library.add(faHeart, faHeartCrack)
>   }
>   connect() {
>     dom.watch()    
> 
>     // 一進來就先判斷，LikeState中的state到底是空的還是false還是true
>     const likeState = this.element.dataset.likeState
> 
>     if (likeState === 'true') {      
>       this.heartBlue()
>     } else if (likeState === 'false') {      
>       this.heartCrackBlue()
>     }
> 
>   }
> 

>   // 點擊愛心的動作    
>   heart() {    
>     const houseID = this.element.dataset.id
>     const token = document.querySelector("meta[name='csrf-token']").content   
> 
>     // 這個是新增like的路徑 - /houses/:id/like
>     fetch(`/houses/${houseID}/like`, {
>       method: "PATCH",
>       headers: {
>         "X-CSRF-Token": token
>       },
>       body: JSON.stringify(params),
>     })
>     .then((resp) => {
>       return resp.json()
>     })
>     .then(({status, houseLikeCount, houseDislikeCount}) => {      
> 
>       if (status === "liked house") {
>         this.heartBlue()
>       } else if ( status === 'delete house like_state' ) {
>         this.heartWhite()
>       } else {
>         this.heartBlue()
>         this.heartCrackWhite()
>       }
> 
>     
>     // 這邊把後端傳來的實際數量，再用dispatch傳到顯示數量的controller
>     const increaseCount = new CustomEvent("increase", {
>       detail: {houseLikeCount: houseLikeCount, houseDislikeCount: houseDislikeCount}
>     }) 
>     window.dispatchEvent(increaseCount)
> 
>     })
>     .catch((err) => {
>       console.log(err);
>     })
>     
>   }
> 
>   // 點擊破碎愛心的動作
>   heartCrack() {
> 
>     const houseID = this.element.dataset.id
>     const token = document.querySelector("meta[name='csrf-token']").content
> 
>     fetch(`/houses/${houseID}/dislike`, {
>       method: "PATCH",
>       headers: {
>         "X-CSRF-Token": token
>       }
>     })
>     .then((resp) => {
>       return resp.json()
>     })
>     .then(({status, houseLikeCount, houseDislikeCount}) => {      
>       if (status === "disliked houses") {
>         this.heartCrackBlue()
>       } else if (status === "delete house like_state")  {
>         this.heartCrackWhite()
>       } else {
>         this.heartCrackBlue()
>         this.heartWhite()
>       }
> 
>       // 這邊把後端傳來的實際數量，再用dispatch傳到顯示數量的controller
>       const decreaseCount = new CustomEvent("decrease", {
>         detail: {houseLikeCount: houseLikeCount, houseDislikeCount: houseDislikeCount}
>       }) 
>       window.dispatchEvent(decreaseCount)
>     })
>     .catch((err) => {
>       console.log(err);
>     }) 
>   }
> 
> 
>   // 把愛心底色加上藍色
>   heartBlue() {
>     this.heartTarget.classList.add("bg-blue-100")
>   }
> 
>   // 把愛心底色加上白色
>   heartWhite() {
>     this.heartTarget.classList.remove("bg-blue-100")
>   }
> 
>   // 把破碎愛心底色加上藍色
>   heartCrackBlue() {
>     this.heartCrackTarget.classList.add("bg-blue-100")
>   }
> 
>   // 把破碎愛心底色加上白色
>   heartCrackWhite() {
>     this.heartCrackTarget.classList.remove("bg-blue-100")
>   }
> 
> }
```






