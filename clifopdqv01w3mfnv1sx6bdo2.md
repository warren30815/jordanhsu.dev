---
title: "五分鐘學前端系統設計面試（四）- 來做 Facebook 貼文串的前端吧！上"
datePublished: Mon Feb 27 2023 03:01:43 GMT+0000 (Coordinated Universal Time)
cuid: clifopdqv01w3mfnv1sx6bdo2
slug: e4ba94e58886e99098e5adb8e5898de7abafe7b3bbe7b5b1e8a8ade8a888e99da2e8a9a6-e59b9b-e4be86e5819a-facebook-e8b2bce69687e4b8b2e79a84e5898de7abafe590a7-e4b88a-11656786456d
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685778013390/a8321649-11f9-4f1d-b64b-5c0d25149add.png

---

此為該文章的重點整理：[https://www.greatfrontend.com/questions/system-design/news-feed-facebook](https://www.greatfrontend.com/questions/system-design/news-feed-facebook)

Written by 許竣翔, Frontend Web Engineer in BioPro Scientific

> 系列文第四篇文章我們會實際用 RADIO Framework 來組織做出一個 Facebook 的貼文串需要考慮的事

### **問題範圍定義（**Requirements）

如下圖，設計一個貼文串，包含使用者可與之互動的貼文列表

#### **需要哪些核心功能**

*   瀏覽自己與朋友建立的貼文
*   對貼文按讚（或心情）
*   建立和發佈新貼文

留言和轉發貼文可以延伸討論，不需要包含在核心功能

#### 要支援哪些種類的貼文 [​](https://www.greatfrontend.com/questions/system-design/news-feed-facebook#what-kind-of-posts-are-supported "Direct link to What kind of posts are supported?")

純文字和圖片，如果時間允許可以討論更多種

#### 貼文要使用什麼樣的分頁 UX[​](https://www.greatfrontend.com/questions/system-design/news-feed-facebook#what-pagination-ux-should-be-used-for-the-feed "Direct link to What pagination UX should be used for the feed?")

無限滾動（Infinite scrolling），當使用者滑到底時載入更多貼文

#### 需要支援行動裝置版嗎[​](https://www.greatfrontend.com/questions/system-design/news-feed-facebook#will-the-application-be-used-on-mobile-devices "Direct link to Will the application be used on mobile devices?")

不是當前最重要，但好的行動裝置體驗是加分的

### 整體架構（Architecture）

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778002616/88308bec-78fb-4eaf-a905-b210d9d1f26a.png)

#### 元件職責[​](https://www.greatfrontend.com/questions/system-design/news-feed-facebook#component-responsibilities "Direct link to Component Responsibilities")

*   Server：提供抓取和建立貼文的 HTTP APIs
*   Controller：控制應用程式的資料流和向 server 發送網路請求
*   Client Store：儲存應用程式的 data，在此為儲存從 server 獲取的貼文資訊，供應 UI 顯示
*   Feed UI：包含貼文串和建立新貼文的 UI  
    Feed Posts：呈現貼文資料和包含貼文互動按鈕（like / react / share）Post Composer：建立新貼文的 UI

### 資料模型（Data Model）

貼文串應用的資料大多是從 server 來的，除了建立新貼文的資料源是從 client 來的

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778005016/b9f315a6-32de-4a7b-8b1a-e4cb097a3212.png)

Feed 為整個貼文串應用的最上層，包含貼文列表和當前最後一則貼文在哪裡的資訊（for 使用者滑到底時抓取接續貼文 data 用）  
Post 為貼文，包含貼文所需的基本資訊  
User 為使用者資訊  
NewPost 為建立新貼文時的貼文資訊

#### 正規化 data store

簡單來說，正規化 data store 包含：

*   類似於資料庫正規化，每種類型的數據存儲在自己的 store 中
*   每項資料有自己的唯一 ID（i.e. 第一正規化）
*   跨類型（跨表）查詢資料時，使用 ID 來查（foreign key），而不是巢狀物件（i.e. 第二正規化）

這種方式能帶來的好處有：

*   降低重複資料，例如許多貼文是同一作者，如不正規化我們會存很多 author 欄位在 client store
*   更簡單地更新資料，如上的例子，假設作者換頭貼，就不需要遞迴地更新巢狀結構裡 author 欄位的資訊

在正規化 data store 上，Facebook 使用 Relay；Twitter 使用 Redux

### API（Interface）

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778007404/0bcb04f5-bb4f-41b3-9ab9-a9c958ae4ec4.png)

貼文串所需的 API 中，最值得探討的是分頁機制，有兩種實作分頁機制的方法：

*   基於位移（Offset-based）
*   基於游標（Cursor-based）

#### 基於位移的分頁

基於位移的分頁 API 需要兩個參數：

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778009929/da8777a4-88d9-4ace-a244-9fad0d77287a.png)

假設我們有 20 則貼文，`{size: 5, page: 2}` 代表我們會拿到第 6–10 篇，回傳的分頁描述資料（metadata）概念如下：

{  
  "pagination": {  
    "size": 5,  
    "page": 2,  
    "total\_pages": 4,  
    "total": 20  
  },  
  "results": \[  
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
      "created\_time": 1620639583  
    }  
    // ... More posts.  
  \]  
}

後端在進行 SQL 語法查詢時會像：

SELECT \* FROM posts LIMIT 5 OFFSET 0; \-- First page  
SELECT \* FROM posts LIMIT 5 OFFSET 5; \-- Second page

基於位移的分頁實作有如下優缺點：

**優點：**

*   使用者可以跳到任意頁
*   容易知道總共有多少頁
*   後端實作容易，offset 的計算即為`(page - 1) * size`

**缺點：**

*   瀏覽貼文時，很常會有其他用戶同時發新貼文，造成抓到重複資料，如：

// Initially  
Posts: A, B, C, D, E, F, G, H, I, J  
       ^^^^^^^^^^^^^ Page 1 contains A \- E  
  
// New posts added over time  
Posts: K, L, M, N, O, A, B, C, D, E, F, G, H, I, J  
                      ^^^^^^^^^^^^^ Page 2 also contains A \- E

*   無法自由變更一頁要抓幾則貼文，如第一次查 `{page: 1, size: 5}` ，第二次查`{page: 2, size: 7}` ，這樣就會漏掉第6, 7則貼文的資料
*   後端查詢資料庫時，如 SQL 內包含 OFFSET，則資料庫實際上需要讀取 `count` + `offset` 數量的資料，並只回傳`count`筆資料，當查詢的位移太大時會降低效能

由上可知，**對貼文串應用而言**，基於位移的分頁實作弊大於利

#### 基於游標的分頁

游標代表當前最後一則貼文的標記，當要查詢新貼文資料時，後端可以藉由這個標記找到資料庫內接續的資料（linked list的概念），這可以解決上述因為新貼文的加入，造成的位移數量不準確的問題

基於游標的分頁 API 需要兩個參數：

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778011605/72958b38-e15b-46ef-b735-aa6aca55d482.png)

回傳的分頁描述資料（metadata）概念如下：

{  
  "pagination": {  
    "size": 10,  
    "next\_cursor": "=dXNlcjpVMEc5V0ZYTlo" # 用來找到下一則貼文資料所需的標記  
  },  
  "results": \[  
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
      "created\_time": 1620639583  
    }  
    // ... More posts.  
  \]  
}

Facebook, Slack 等都是採用基於游標的分頁方法

**優點：**

*   解決因新貼文加入造成抓到重複資料的問題
*   降低資料庫查詢負擔（此標記即為 primary key，因此不用用 OFFSET 的查詢語法）

**缺點：**

*   沒辦法跳到任意頁，因為是一則一則串起來的，要跳到下一則需要前一則的 next\_cursor 資訊（即 linked\_list 中的 next pointer概念）
*   實作相比基於位移的方法稍複雜

當我們在滑貼文串時，使用者並不在意能不能跳到任意頁，因此綜上所述，基於游標的分頁實作是更佳的選擇

### 補充 (2023/03/07)

在電商應用中，offset-based的方式更好，因為對電商來說，事先知道總共有幾頁商品以及可自由切換頁數，對於查找商品來說是很方便的，而且平常沒事不會有太高頻地新商品加入，offset-based的缺點（高頻有新加入的item時會不斷拿到重複資料）在此應用情境下就還好

### 結語

篇幅長度關係，剩下的 Optimizations 留在下兩篇文章介紹

本篇文章中，我們介紹了實作一個 Facebook 的貼文串的前端架構與分頁 API 設計，基於游標的分頁實作比基於位移的更適合應用於貼文串這種會頻繁新增資料的情境中，其相對多付出的實作時間成本是必要的

And that’s a wrap! Enjoy. 🎆

如果覺得對你有幫助的話可以幫我拍下手，最多可以50下～👏

My Linkedin: [https://www.linkedin.com/in/%E7%AB%A3%E7%BF%94-%E8%A8%B1-3188a41a1/](https://www.linkedin.com/in/%E7%AB%A3%E7%BF%94-%E8%A8%B1-3188a41a1/)