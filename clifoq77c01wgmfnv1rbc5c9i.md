---
title: "五分鐘學前端系統設計面試（三）- RADIO Framework 下"
datePublished: Sat Feb 11 2023 05:45:30 GMT+0000 (Coordinated Universal Time)
cuid: clifoq77c01wgmfnv1rbc5c9i
slug: frontend-system-design-facebook-feed-3
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685778051245/da372604-af2b-4ca2-958a-4555a2e2c9d8.png
tags: frontend-development, system-design

---

此為該文章的重點整理：[https://www.greatfrontend.com/system-design/framework](https://www.greatfrontend.com/system-design/framework)

> *系列文第三篇文章繼續介紹如何系統性回答前端系統設計面試的問題*

### Data Model（分配 10%的面試時間）

接著我們討論在 client 端的資料欄位，client 端的資料來源有兩種

* server 來的資料（database）
    

常見的例子包含使用者資料（名字、個人資料、頭貼）和使用者產生的資料（貼文、留言）

* client-only 的資料（state）
    

留在 client 端的 store，可細分為兩種

1. 須永久保存的資料：如使用者輸入的表單資訊，需送往 server 儲存於 database
    
2. 暫時性的資料：如表單驗證狀態、目前 tab、區塊是否展開…，這類型資料在關掉分頁或重新整理會消失
    

一樣是動態牆的例子：

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778044558/91c8360e-34c9-4af2-a768-31369b4e8224.png align="left")

### Interface（分配 15%的面試時間）

現在我們有了一些核心元件和內部的資料欄位，接著我們討論元件間的通訊API，API是一個軟體元件間通訊方式的總稱，Client 和 server 溝通可以透過網路層的 APIs (HTTP / WebSockets)；Client 元件間可以透過瀏覽器 runtime 的函數來溝通，所有的 APIs 共通有三個東西：名稱&功能、input、output

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778046739/1ccb8694-54d9-4053-994e-0ca714adb987.png align="left")

***Server-Client API Example***

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778049160/f6c2e051-b9b2-493e-a52a-1e4be9c415ef.png align="left")

**Input**

貼文是一個分頁列表，所以 API 的輸入需要分頁相關資訊，如 size 代表要請求 10 則新貼文，cursor 代表當前最後一筆資料在資料庫內的標記（pointer，可以簡單理解為 id 的概念）

```json
{
    "size": 10,
    "cursor": "=dXNlcjpXMDdRQ1JQQTQ"
}
```

**Output**

```json

{
    "pagination": {
        "size": 10,
            "next_cursor": "=dXNlcjpVMEc5V0ZYTlo"
    },
    "results": [
        {
            "id": "123",
            "author": {
                "id": "456",
                "name": "John Doe"
            },
            "content": "Hello world",
            "image": "https://www.example.com/feed-images.jpg",
            "reactions": {
                "likes": 20,
                "haha": 15
            },
            "created_time": 1620639583
        }
        // ... More posts.
    ]
}
```

***Client-Client API Example***

Client 元件間的溝通 API 可以跟 server-client API 類似，差別只是變成以 JS 函數的形式溝通，或者透過他們所 listen 的事件

***API for UI Component System Design***[​](https://www.greatfrontend.com/system-design/framework#api-for-ui-component-system-design)

如果是 UI 元件的設計，可以討論元件提供的 options / props

### Optimizations（分配 40%的面試時間）

優化部分沒有固定的方式，取決於不同領域的產品，以下是一些 guidelines：

* 專注在該產品的重要領域，例如電商，效能很重要，你會需要花多點時間去討論多種效能優化的技巧；例如協作軟體，多人同時編輯和 race conditions 問題是需要多討論的
    
* 專注你的強項，如果你在無障礙使用這塊很強，多討論其相關在該產品可能發生問題的地方與解法；如果你在效能優化很強，多討論該產品能優化的地方，提供更滑順的使用者體驗[​](https://www.greatfrontend.com/system-design/framework#general-optimizationdeep-dive-areas)
    

總結在優化這部分，可以討論的主題列點如下：

* 效能（Performance）
    
* 使用者體驗（User Experience）
    
* 網路（Network）
    
* 無障礙使用（Accessibility，a11y for short）
    
* 多國語系（Multilingual Support）
    
* 跨裝置支援（Multi-device Support）
    
* 資安（Security）
    

要注意這不是菜單，要看產品著重的點而定，而不是全部翻出來討論一輪

### 結語

上篇和本篇文章中，我們介紹了用來回答前端系統設計問題的RADIO Framework，再讓我們 recap 一下：

* **R**equirements（&lt;15%的時間）：徹底了解需求，並不斷問問題確認需求範圍
    
* **A**rchitecture（~20%的時間）：定義出幾個關鍵的元件和他們的關聯，著重於high level的架構，而不是各元件如何實作
    
* **D**ata Model（~10%的時間）：定義關鍵元件基本會需要哪些資料欄位
    
* **I**nterface（~15%的時間）：定義元件間通訊的API，包含這些API的功能、input & output格式
    
* **O**ptimizations（~40%的時間）：討論可能的優化效能方法，和深入探討這類型產品可能會特別需要著重的功能（見系列文一的應用元件表格）
    

下一篇開始我們會更完整深入的針對 Facebook 動態牆這個主題來探討 RADIO 各層面

And that’s a wrap! Enjoy. 🎆

👏