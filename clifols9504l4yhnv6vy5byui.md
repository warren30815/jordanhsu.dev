---
title: "五分鐘學前端系統設計面試（二）- RADIO Framework 上"
datePublished: Sat Feb 11 2023 04:04:11 GMT+0000 (Coordinated Universal Time)
cuid: clifols9504l4yhnv6vy5byui
slug: e4ba94e58886e99098e5adb8e5898de7abafe7b3bbe7b5b1e8a8ade8a888e99da2e8a9a6-e4ba8c-radio-framework-e4b88a-cc0174954103
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685777845238/b86c0d6e-7030-49a0-b5b6-163da0a184e5.jpeg

---

此為該文章的重點整理：[https://www.greatfrontend.com/system-design/framework](https://www.greatfrontend.com/system-design/framework)

Written by 許竣翔, Frontend Web Engineer in BioPro Scientific

> 系列文第二篇文章會介紹如何系統性回答前端系統設計面試的問題

### RADIO Framework

前端系統設計面試時，我們要把系統當成做一個產品來看待，RADIO框架由五個英文字所組成，分別是：

*   **R**equirements：徹底了解需求，並不斷問問題確認需求範圍
*   **A**rchitecture：定義出幾個關鍵的元件和他們的關聯，著重於high level的架構，而不是各元件如何實作
*   **D**ata Model：定義關鍵元件基本會需要哪些資料欄位
*   **I**nterface：定義元件間通訊的API，包含這些API的功能、input & output格式
*   **O**ptimizations：討論可能的優化效能方法，和深入探討這類型產品可能會特別需要著重的功能（見[系列文一](https://medium.com/@qaz7821819/%E5%89%8D%E7%AB%AF%E7%B3%BB%E7%B5%B1%E8%A8%AD%E8%A8%88%E4%B9%8B%E6%97%85-%E4%B8%80-be5d7ef68275)的應用元件表格）

下面我們會逐點詳細展開，以及他們應所佔的面試時間比例

### **R**equirements（分配 < 15%的面試時間）

系統設計面試問題通常都是很廣泛、沒有標準答案的問題，因此面試者需要不斷問問題確認好需求的範圍，把你的面試官當成日常工作時的PM，把系統設計面試當成你日常要打造一個產品的感覺

不斷問問題除了可以幫助釐清需求範圍外，也可以確認面試官到底對這需求最有興趣的部分在哪裡，這部分需求也正是該花最多時間探討的，下面列出幾個常見你該問的問題：

***這產品的主要使用情境是？***

想像一下，你被要求”設計一個Facebook”，一個如此大的平台，包含了一堆功能，如：動態牆、個人主頁、朋友、群組、限時動態…，短短的面試時間內是不可能全部都回答到的，面試官其實心中有希望你回答的部分，但希望你問他我要回答什麼，俗稱的傲嬌（？

通常來說，要回答該產品最獨特的功能，例如Facebook可能就是動態牆、動態的分頁和建立新貼文；YouTube可能就是觀賞影片的體驗，[系列文一](https://medium.com/@qaz7821819/%E5%89%8D%E7%AB%AF%E7%B3%BB%E7%B5%B1%E8%A8%AD%E8%A8%88%E4%B9%8B%E6%97%85-%E4%B8%80-be5d7ef68275)的應用元件表格詳細列出了回答不同類型應用重要的部分

在本文我們會以"設計Facebook”來作為主要的例子，並套用RADIO Framework去回答它

***功能性需求和非功能性需求分別是？***

*   功能性需求（Functional requirements）：讓使用者可以完成一個核心的使用流程的最基本功能需求，俗稱的能動
*   非功能性需求（Non-functional requirements）：可以提升產品品質的功能，如效能（頁面載入、交互多快）、擴展性（能顯示多少物件）、更好的使用者體驗…，俗稱的動得好

顯而易見，我們應先達成所有功能性需求（能動），有多餘的時間才更去探討非功能性需求（動得好）

而有兩個你可以得到這問題答案的方法；

1.  **列出你腦中直覺，並和面試官align（推薦）**
2.  直接問面試官，但通常他們希望你先自己整理

***有哪些核心 features 和 good-to-have？***

即使你定義好了功能性需求，但這需求仍包含一堆小features，舉例來說，建立一則Facebook貼文，要支援哪些貼文格式，除了最常見的文字外，使用者是否可以上傳圖片、影片、建立投票、打卡…，你應該先確認好核心features後，才去討論那些額外features

***其他問題***

*   需要支援哪些裝置 / 平台（桌機、平板、手機）
*   是否需要離線支援
*   誰是這產品的主要使用者
*   是否有任何效能要求？（通常是放在非功能性需求的確認）

上述問題是一些你可以視需要問的問題，但不是菜單不需要全問一輪，不同類型的問題往往還需要確認 domain-specific 的問題，在之後 case studies 的系列文會再介紹

### Architecture（分配 20%的面試時間）

確認好需求後，下一步是定義出產品的關鍵元件 & 這些元件間的關聯互動，記得要**專注在 client-side 的架構，而不是 server-side**

流程圖這時候很好用，每個元件可以表示為一個長方形盒子（如下圖），元件間的互動資料流用箭頭表示，元件裡面也可以有多個子元件，表示上為一個大長方形盒子裡面有許多小盒子

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777842715/83e52a4e-82df-43b8-b345-083152f779f3.png)

上圖為一個動態牆 high-level 前端架構設計的例子，分成幾個部分

*   Server：前端系統設計面試中，可以將其視為一個黑箱，並假設它有一些API可以call，透過 HTTP / WebSockets
*   View：使用者看到的部分，通常包含許多 subviews
*   Controller：回應使用者的互動，以及將 store / model 的資料處理成 View 預期的格式，如果產品比較小且元件間沒有太多的資料傳遞，這部分不一定需要
*   Model / Client Store：Data所在處，實務上一個應用內你可以包含多個 stores，stores 也可以包含其他 stores

以動態牆這例子來說，回答可以是這樣：

*   Server：處理貼文資料，並藉由 HTTP API 方式傳送
*   Controller：控制應用內的資料流，並向 server 送出網路請求
*   Client Store：在應用內全域儲存 data，在此多數 data 來源是 server，服務 feed UI 所需的資料
*   Feed UI：包含貼文列表，和撰寫新貼文的 UI
*   Feed Post：呈現貼文，並包含使用者的互動按鈕（如按讚、回應、分享貼文…）
*   Post Composer：撰寫新貼文的 UI

畫流程圖的工具推薦 [diagrams.net](https://app.diagrams.net/) 和 [Excalidraw](https://excalidraw.com/)，筆者自己用的是前者

### 結語

篇幅長度關係，剩下的 Data Model、Interface、Optimizations 留在下篇文章介紹

本篇文章中，我們介紹了前端系統設計面試中如何確認需求的範圍，以及如何定義 high-level 的架構

And that’s a wrap! Enjoy. 🎆

👏

My Linkedin: [https://www.linkedin.com/in/%E7%AB%A3%E7%BF%94-%E8%A8%B1-3188a41a1/](https://www.linkedin.com/in/%E7%AB%A3%E7%BF%94-%E8%A8%B1-3188a41a1/)