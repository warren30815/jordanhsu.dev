---
title: "五分鐘學前端系統設計面試（六）- 來做 Facebook 貼文串的前端吧！下"
datePublished: Tue Feb 28 2023 02:28:32 GMT+0000 (Coordinated Universal Time)
cuid: clifopkrm01w8mfnvcmql9dkt
slug: frontend-system-design-facebook-feed-6
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685778022449/52f745a0-4165-42bc-a16c-519830bfa86e.png
tags: frontend-development, system-design

---

此為該文章的重點整理：[https://www.greatfrontend.com/questions/system-design/news-feed-facebook](https://www.greatfrontend.com/questions/system-design/news-feed-facebook)

> 系列文第六篇文章我們繼續探討實作 Facebook 的貼文、新增貼文功能有哪些優化方向與細節

### 貼文優化

#### 只在需要時提供資料驅動的依賴性

Facebook 使用 Relay 來向後端抓資料，Relay 是一個基於 JavaScript 的 GraphQL 客戶端，其結合 React 元件和 GraphQL 來讓 React 元件聲明他需要哪些資料欄位，Relay 就會透過 GraphQL 的方式來抓取並提供資料給元件

```graphql
// Sample GraphQL query to demonstrate data-driven dependencies.
... on Post {
... on TextPost {
@module('TextComponent.js')
contents
}
... on ImagePost {
@module('ImageComponent.js')
image_data {
alt
dimensions
}
}
}
```

#### 提及 / 主題標籤（Hashtag）[​](https://www.greatfrontend.com/questions/system-design/news-feed-facebook#rendering-mentionshashtags)

上面的貼文中，我們可以看到有一些特殊語法，比如 “#AboutLastNight” 的主題標籤，以及 “HBO Max” 的提及，儲存這類包含特殊語法的格式的方式有以下幾種：

* HTML格式
    

最簡單的方式，直接存要渲染的 HTML

```xml
<a href="...">#AboutLastNight</a> is here... and ready to change the meaning of date night...

Absolute comedy 🤣 Dropping 2/10 on <a href="...">HBO Max</a>!
```

但也是最不好的方式，因為很容易導致 XSS 攻擊，而且後續要加 style 會很難加，以及儲存的 HTML 無法用於非 web 的客戶端（e.g. iOS / Android）

* 客製語法[​](https://www.greatfrontend.com/questions/system-design/news-feed-facebook#custom-syntax)
    

可預先定義一些語法規則，比如 “#” 開頭的字串就解析為主題標籤；提及的格式則如 `[[#1234: HBO Max]]` ，其他一律視為純文字，這種客製化的方式是輕量且安全（不會被 XSS）的解法，適用於之後不會再需要支援更多新種類的富文本（rich text）的情境

* 有名的富文本編輯器格式[​](https://www.greatfrontend.com/questions/system-design/news-feed-facebook#popular-rich-text-editor-format)
    

Draft.js 是 Facebook 開發的用來讀寫富文本的編輯器，範例如下：

```json
{
    content: [
        {
            type: 'HASHTAG',
            content: '#AboutLastNight',
        },
        {
            type: 'TEXT',
            content: ' is here... and ready to change ... Dropping 2/10 on ',
        },
        {
            type: 'MENTION',
            content: 'HBO Max',
            entityID: 1234,
        },
        {
            type: 'TEXT',
            content: '!',
        },
    ];
}
```

可以看出編輯器將上述例子表示成抽象語法樹（Abstract Syntax Tree），可以被序列化成 JSON 字串儲存，這種方式可以不用寫客製化的解析程式，且易於擴展新種類的富文本，而缺點是這種格式通常相對上面的客製語法方式有著更長的字串，因此需要更高的網路傳輸、硬碟儲存空間成本

#### 渲染圖片[​](https://www.greatfrontend.com/questions/system-design/news-feed-facebook#rendering-images)

* 使用 CDN 來快取圖片
    
* 使用 WebP 格式
    
* `<img>`s 應該要有合適的 `alt` 文字
    

Facebook 利用 AI 來辨識圖片並產生圖片描述，用來作為`alt` 中的文字

* 根據螢幕尺寸來載入合適大小的圖片
    

在發送請求時同時附上目前的螢幕尺寸，讓後端可以決定要回傳何種大小的圖片；或者使用 `srcset` 來決定要載入 @1x、@2x 還是 @3x 的圖片

* 根據網速來自適應圖片載入
    

網路好 / WiFi：預先擷取（Prefetch）還沒進入但即將要進入螢幕的資料

網路差：渲染低解析度的 placeholder 圖片，並需要使用者點進去才會去載入更高解析度的圖片

#### 懶載入初次渲染時不需要的 code [​](https://www.greatfrontend.com/questions/system-design/news-feed-facebook#lazy-load-code-that-is-not-needed-for-initial-render)

* 回應貼文心情的彈出式視窗
    
* 右上角的更多功能 dropdown 選單，通常以刪節號圖示表示
    

上述程式我們可以在這些時候去下載：

* 瀏覽器空閒時去 prefetch
    
* 使用者請求他們時，比如 hover 或點擊按鈕
    

#### 樂觀更新（Optimistic Updates）[​](https://www.greatfrontend.com/questions/system-design/news-feed-facebook#optimistic-updates)

正常情況我們會假定我們的請求能正常執行（status 200），因此我們可以在使用者做操作的當下先直接更新 client store（比如按讚就直接把畫面上的讚數 +1），讓使用者的響應更即時，而不必等到後端真的回傳結果才更新，除非後端回傳非成功的狀態，才再根據情況把 client store 改回來，並跳出錯誤訊息

#### 多國語系下的時間戳記（Timestamp）[​](https://www.greatfrontend.com/questions/system-design/news-feed-facebook#multilingual-timestamps)

有三種方式來實作：

* 後端回傳原始的 Unix timestamp，而客戶端依據使用者設定的語言與時區來渲染相對應的時間字串，這樣的方式最為彈性，但缺點是會顯著增加 JavaScript build 出來的 bundle 大小（因為要儲存翻譯各國語言的語法規則和翻譯字串）
    
* 後端回傳處理過的時間戳記，直接依照後端所設定的時區，這樣的方式簡單方便，但無法依照不同地區的客戶端來更新顯示的文字
    
* 使用`Intl.DateTimeFormat()` API 來轉換原始的 Unix timestamp 為對應的時區
    

```javascript
const date = new Date(Date.UTC(2021, 11, 20, 3, 23, 16, 738)); // 第二個月份的數值範圍為 0 ~ 11
console.log(
    new Intl.DateTimeFormat('zh-CN', {
        dateStyle: 'full',
        timeStyle: 'long',
    }).format(date),
);
// 2021年12月20日星期一 GMT+8 11:23:16
```

#### 相對的時間戳記會過期[​](https://www.greatfrontend.com/questions/system-design/news-feed-facebook#relative-timestamps-can-become-stale)

上述的時間戳記為絕對時間，但對用戶來說，相對時間（e.g. 3 minutes ago, 1 hour ago, 2 weeks ago, etc）在閱讀上會更直觀簡單，只是因為貼文串應用中，用戶基本上不會去刷新頁面，因此可以加入一計時器（setInterval） 來定期更新當前貼文們的時間戳記

#### 渲染圖示（icon）[​](https://www.greatfrontend.com/questions/system-design/news-feed-facebook#icon-rendering)

渲染圖示有幾種方式：

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778019719/5bf9ea48-3937-409e-8454-049cbba2c0ec.png align="left")

Spritesheet（精靈圖）概念上為將許多小icon合併成一張圖片，然後在依照指定的方式切割成一個一個 icon，缺點為需要事先定義好切割方式，如果之後要換方式，需要連程式都一起改動

Facebook 和 Twitter 使用行內 SVG（inlined SVG），這似乎也是最近的趨勢

#### 使用者體驗[​](https://www.greatfrontend.com/questions/system-design/news-feed-facebook#user-experience)

* 文字太多時的 “See more” 按鈕
    
* 按讚數顯示方式：
    

好：John, Mary and 103K others  
不好：John, Mary and 103,312 others

### 新增貼文優化

#### 懶載入依賴

需要時才載入，如：[​](https://www.greatfrontend.com/questions/system-design/news-feed-facebook#lazy-loading-dependencies)

* 圖片上傳器
    
* GIF 選擇器
    
* Emoji 選擇器
    
* 貼圖選擇器
    
* 背景圖片
    

#### 無障礙使用（a11y）[​](https://www.greatfrontend.com/questions/system-design/news-feed-facebook#accessibility)

* 貼文串
    

在貼文串的 HTML 元素中加入`role="feed"`

* 貼文
    

在每則貼文的 HTML 元素中加入 `role="article"`

`aria-labelledby="<id>"` 裡包含貼文作者名字和`id` 屬性

貼文內容區應該要可以透過鍵盤來 focus（加入`tabindex="0"`) 和適當的 `aria-role`.

* 貼文互動
    

Facebook 上的使用者可以將滑鼠 hover 在 “喜歡” 按鈕上來獲得更多回應選項，為了讓鍵盤也可以使用相同的功能，Facebook 提供了一個僅在 focus 時出現的按鈕，並且可以用該按鈕打開回應選單。

如果只有圖示、沒有附帶文字標籤的按鈕（icon-only button，e.g. Twitter），就應該要有 `aria-label`

### 結語

系列文四五六利用 RADIO Framework 來探討實作一個 Facebook 貼文串的架構與要注意的細節和優化

And that’s a wrap! Enjoy. 🎆