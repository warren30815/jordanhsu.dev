---
title: "五分鐘學前端系統設計面試（五）- 來做 Facebook 貼文串的前端吧！中"
datePublished: Mon Feb 27 2023 14:41:18 GMT+0000 (Coordinated Universal Time)
cuid: clifopwxx01w9mfnvfwj8eky7
slug: e4ba94e58886e99098e5adb8e5898de7abafe7b3bbe7b5b1e8a8ade8a888e99da2e8a9a6-e4ba94-e4be86e5819a-facebook-e8b2bce69687e4b8b2e79a84e5898de7abafe590a7-e4b8ad-fd2fb34bde19
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685778038103/bded0bf6-eb34-485c-bc61-19ef22edfe4b.png

---

此為該文章的重點整理：[https://www.greatfrontend.com/questions/system-design/news-feed-facebook](https://www.greatfrontend.com/questions/system-design/news-feed-facebook)

Written by 許竣翔, Frontend Web Engineer in BioPro Scientific

> 系列文第五篇文章我們會深入探討實作 Facebook 的貼文串有哪些優化方向與細節

### 優化與探討（Optimizations）

貼文串中主要有幾個元件：

*   貼文串 UI
*   貼文 UI（下篇探討）
*   新增貼文 UI（下篇探討）

本篇主要先討論貼文串優化的相關細節

#### ***渲染方式***

**\*\*\* 篇幅關係，此處筆者假設讀者已有 SSR / CSR 的背景知識 \*\*\*\***

為了兼顧 SEO 與互動性，Facebook 採用了混合兩者（Isomorphic SSR）的方式，初次載入採用 SSR，渲染出帶有基本資料的 HTML，送到前端後再進行 hydration（在 HTML 加上事件處理器），讓網站產生可互動性，之後往下滑獲取新貼文時就走 CSR 方式，流程可見下圖：

credit: [https://ithelp.ithome.com.tw/articles/10279519](https://ithelp.ithome.com.tw/articles/10279519)

要實作這種混合模式，可透過 Next.js（for React）或 Nuxt.js（for Vue）

#### 無限滾動（Infinite Scrolling)

當使用者下滑快到底時，我們要 call API 抓取新貼文，除了在滑到底時加一個 loading UI，更好的 UX 為提前在使用者真正滑到底前就先 call API了，而這個提前距離的計算可以動態地根據網速和使用者滾動速度來決定，一個理想的距離為短到避免 false positives 造成頻寬浪費；同時又長到使用者不會感覺到有 loading 延遲

**如何實作：**

我們會需要加一個標記元素（marker element）來指示當前貼文串的底部

*   監聽一`scroll` 事件或一 timer (`setInterval`) 來確認該標記元素是否距離底部太近，標記元素的距離資訊可透過`[Element.getBoundingClientRect](https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect)`取得
*   使用 [Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API) 來監聽標記元素何時進入、離開容器（container）或可見區域（viewport）

Intersection Observer 是原生的 Web API，只需註冊一 callback 函數來處理當相交事件發生時要做什麼，瀏覽器負責管理元素間的相交，相比用 `scroll + Element.getBoundingClientRect()`來實作，每次滾動位置變更就要計算一次，Intersection Observer API 只有在標記元素進入、離開容器或可見區域時才會觸發上述計算，因此更推薦使用來實作無限滾動，同時也有更簡潔的程式碼（只需處理 callback）

#### 虛擬列表（Virtualized Lists）

如果有 1000 篇貼文，我們又將這些貼文同時渲染的話，就必須生成 1000 個 DOM 節點，很容易造成頁面的卡頓，虛擬列表就是優化長列表的一種技巧

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778028771/4a741e3f-5450-49da-adf9-e56db9ab961b.png)

credit: [https://ithelp.ithome.com.tw/m/articles/10271764](https://ithelp.ithome.com.tw/m/articles/10271764)

實務上，Facebook 以空的 `<div>` 替換離開螢幕的貼文，以及動態計算行內樣式（e.g. `style="height: 300px"`），藉此保存滾動的位置資訊並加入`[hidden](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/hidden)`屬性，這可以提升渲染效能：

*   Browser painting：要渲染的 DOM 節點與 layout 的計算更少
*   Virtual DOM reconciliation (React)：React 的渲染機制，因為現在要渲染的 DOM 節點更少，對於 diff 算法也可以更快地得出結果

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778030967/8c27048b-d5fe-4974-8d77-0d1a21822c62.png)

credit: [https://hackmd.io/@Heidi-Liu/virtual-dom#%E8%A3%9C%E5%85%85%EF%BC%9AVirtual-DOM-%E7%9A%84%E8%B5%B7%E6%BA%90](https://hackmd.io/@Heidi-Liu/virtual-dom#%E8%A3%9C%E5%85%85%EF%BC%9AVirtual-DOM-%E7%9A%84%E8%B5%B7%E6%BA%90)

Facebook 和 Twitter 都使用了虛擬列表

#### 程式分割（Code Splitting）[​](https://www.greatfrontend.com/questions/system-design/news-feed-facebook#code-splitting-javascript-for-faster-performance "Direct link to Code Splitting JavaScript for Faster Performance")

通常程式分割可以分兩個層級探討：

*   頁面層級：每個頁面只會載入自己需要的 JavaScript 和 CSS
*   懶載入（Lazy loading）頁面內所需資源：有些頁面中不會馬上用到的元件就不用馬上載入進來（e.g. 彈出視窗、對話窗…）

在貼文串應用中，只有一個頁面，所以頁面層級的程式分割就不太必要，但貼文的互動功能元件很多，懶載入就很必要了

#### 鍵盤快捷鍵（Shortcuts）[​](https://www.greatfrontend.com/questions/system-design/news-feed-facebook#keyboard-shortcuts "Direct link to Keyboard Shortcuts")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778033273/61040110-d187-4be9-94b6-9f01bcec63f8.png)

#### 載入動畫[​](https://www.greatfrontend.com/questions/system-design/news-feed-facebook#loading-indicators "Direct link to Loading Indicators")

不可避免地使用者快速下滑或網速很慢時，我們還是得呈現載入動畫，與其用轉圈圈的方式，更好的 UX 為使用微光載入效果（shimmer loading effect），如下 GIF

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778035789/bd08f613-7c64-4d0a-a719-a25bb8dc21ee.gif)

除了更好看外，因為骨架已經出來，可以省去瀏覽器再次 reflow 的 cost，只需要做內容的 repaint 更新即可

#### 動態更新一頁拿取的貼文數量

由於我們使用基於游標的分頁實作，我們在一頁的貼文數量上面可以指定任意數字，當初次載入時，我們不知道視窗高度，所以會保守性的多要一點資料，但接下來 query API 時，我們就可以根據視窗高度來動態決定一次需要抓多少貼文

#### 保存滾動位置[​](https://www.greatfrontend.com/questions/system-design/news-feed-facebook#preserving-feed-scroll-position "Direct link to Preserving Feed Scroll Position")

使用者很可能導覽到其他頁面（比如個人資料設定頁面），這時他再回來時會傾向看到同樣的畫面，我們可以拿 client store 中暫存的貼文資訊與當前滾動位置

#### 錯誤處理[​](https://www.greatfrontend.com/questions/system-design/news-feed-facebook#error-states "Direct link to Error States")

處理當貼文無法獲取或網路斷線時的情況

### 結語

篇幅長度關係，剩下的貼文 UI、新增貼文 UI（下篇探討）的優化細節留在下篇文章介紹

本篇文章中，我們探討了實作貼文串所需注意的細節與優化

And that’s a wrap! Enjoy. 🎆

如果覺得對你有幫助的話可以幫我拍下手，最多可以50下～👏

My Linkedin: [https://www.linkedin.com/in/%E7%AB%A3%E7%BF%94-%E8%A8%B1-3188a41a1/](https://www.linkedin.com/in/%E7%AB%A3%E7%BF%94-%E8%A8%B1-3188a41a1/)