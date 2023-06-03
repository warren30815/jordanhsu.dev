---
title: "擴充 ChatGPT，查資料不必再受困於分頁叢林中"
datePublished: Tue Feb 21 2023 11:53:45 GMT+0000 (Coordinated Universal Time)
cuid: clifoqk5m04mqyhnvcozbgqdz
slug: e693b4e58585-chatgpt-e69fa5e8b387e69699e4b88de5bf85e5868de58f97e59bb0e696bce58886e9a081e58fa2e69e97e4b8ad-9beaa3b68d18
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685778068299/87c39f26-d11e-470f-8983-caf20a7d4f63.png

---

#### google chrome 插件分享與前端技術交流

> Highlight unknown sentence, and let’s ChatGPT explain to you

年後收假前筆者寫了一個輔助使用 ChatGPT 的 chrome extension  
痛點來自於筆者在查網頁資料時，如果有遇到不懂的詞或段落，就需要額外切分頁先查，查完再跳回來很容易忘記原本在幹嘛（日常分頁太多直接一個大腦霧）

因此筆者秉持著工程師天生就是懶（x，追求高度自動化（o 的特質，再思考有什麼解法可以讓使用 ChatGPT 查詢時更方便，串接這種 API 服務的介面有幾種常見的表現形式：

*   聊天機器人（比如line bot、discord）
*   網頁
*   桌面應用程式
*   瀏覽器擴充套件

由於筆者設定的情境主要是希望輔助使用者在查網路文章學習時，如果遇到不懂的關鍵字可以更快的了解大概意思，而不中斷原本正在學習的主題，前三者都需要一個新的介面或分頁來顯示資訊；而瀏覽器擴充套件可以直接操作當前網頁的DOM，秀出一個自訂的popup視窗用來顯示資訊，因此筆者覺得這樣的方式可以不用切換分頁來顯示查詢結果，對使用者是最為方便的

選定執行方向後，筆者模仿了google translation extension的做法，偵測反白的字詞，反白結束後跳出一個小 icon，點下去會出現一個 popup 顯示ChatGPT 對反白文字的解釋

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778057986/63ef7c86-9675-4285-bde4-475325df0dea.png)

如果只是要查詢單詞，也可以直接按右鍵，選取context menu選單中的選項

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778061232/464484ae-51ce-4125-82fa-0059e42e05f2.png)

demo影片可見此：

<iframe src="https://www.youtube.com/embed/5MAh_BZvF6Q?feature=oembed" width="700" height="393" frameborder="0" scrolling="no"></iframe>

目前已上架在 google chrome extension 商店，和筆者一樣覺得學習過程中有一樣痛點的人可以下載玩看看！有其他想要的feature / 發現的bug也可以直接在 google chrome extension 商店留言 or gitlab repo 上發 issue

chrome extension 下載連結：

[**Hey ChatGPT Explain This**  
*Highlight unknown sentence, and let's ChatGPT explain to you*chrome.google.com](https://chrome.google.com/webstore/detail/hey-chatgpt-explain-this/ncockfahljbgijljglbmmagefbdheofd "https://chrome.google.com/webstore/detail/hey-chatgpt-explain-this/ncockfahljbgijljglbmmagefbdheofd")[](https://chrome.google.com/webstore/detail/hey-chatgpt-explain-this/ncockfahljbgijljglbmmagefbdheofd)

gitlab repo：

[**warren30815 / hey-chatgpt-explain-this · GitLab**  
*GitLab.com*gitlab.com](https://gitlab.com/warren30815/hey_chatgpt_explain_this "https://gitlab.com/warren30815/hey_chatgpt_explain_this")[](https://gitlab.com/warren30815/hey_chatgpt_explain_this)

— — — — — — — — 下方為如何撰寫該插件的前端技術分享— — — — — — — —

這部分主要是 for 前端工程師看的

#### **專案基本結構：**

簡單來說寫 chrome extension 跟寫網頁差不多  
主要分成能跟 DOM 互動的 content script（foreground，main thread）  
與在 chrome 背景執行的 service worker（background，worker thread，不可操作 DOM 也不能取用 window）

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778063129/e038a8e7-00c9-4cc5-890d-c7b5fb342b7c.png)

credit: [https://hackmd.io/@lala-lee-jobs/service-worker](https://hackmd.io/@lala-lee-jobs/service-worker)

content script 我主要拿來控制 popup 的顯示、更新渲染內容、滑鼠事件處理  
service worker 我主要拿來 call API 和 streaming 結果給 content script

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778065327/9197b650-67b2-400c-a57e-41fc02d3d2a4.jpeg)

credit: [https://slideplayer.com/slide/15938823/](https://slideplayer.com/slide/15938823/)

Github上也有許多 starter project 可以直接作為 scaffold

[**Build software better, together**  
*GitHub's search supports a variety of different operations. Here's a quick cheat sheet for some of the common searches…*github.com](https://github.com/search?q=chrome+extension+starter "https://github.com/search?q=chrome+extension+starter")[](https://github.com/search?q=chrome+extension+starter)

#### 實作上的幾個重點：

首先我們先 recap 一下整體的使用流程：

1.  Install the extension, and you will see a popup window.
2.  Visit [OpenAI website](https://beta.openai.com/account/api-keys) to get API key.
3.  Register your OpenAI API KEY to the input field.
4.  Once finished, you can highlight some sentence in any website. You will find a icon button when you release your mouse and click it.
5.  The highlighted sentence will be sent to ChatGPT, and it will explain the sentence meaning for you.

*   為求快速開發，筆者這邊就沒有去串接 openai 登入驗證流程拿 access token（key for short），而是請使用者手動輸入 key，這邊想探討的主要是拿到 key 後要如何保存於瀏覽器，chrome extension 這邊有提供專門 for extension 版本的 storage，筆者這邊使用 chrome.storage.session，其將資料存於 memory，瀏覽器關閉時會自動清除，安全性較佳，當然還是有可能被其他惡意的 chrome extension or script running in the same origin 給 dump 出來，因此可以再加入時間過期自動清除機制，只是使用者就會需要時不時再授權一次，這個過期時間是一個資安與 UX 的 tradeoff

[**chrome.storage — Chrome Developers**  
*The Storage API provides an extension-specific way to persist user data and state. It’s similar to the web platform’s…*developer.chrome.com](https://developer.chrome.com/docs/extensions/reference/storage/#property-session "https://developer.chrome.com/docs/extensions/reference/storage/#property-session")[](https://developer.chrome.com/docs/extensions/reference/storage/#property-session)

*   偵測反白結束的事件，由於 Web API 沒有提供 selection end 的事件，只有 selectionchange，因此透過結合 mouseup 事件，實作一個帶有 debounce 機制的 selectionend 事件

// debounce version of selection event  
  
let selectionEndTimeout: NodeJS.Timeout = null  
  
document.addEventListener('mouseup', (evt: MouseEvent) => {  
    selectionEndTimeout = setTimeout(() => {  
        const haveText = window.getSelection().toString() !== ''  
        if (haveText) {  
            // handle select end event (show icon button)  
        }  
    }, 100)  
})  
document.addEventListener('selectionchange', () => {  
    if (selectionEndTimeout) {  
        clearTimeout(selectionEndTimeout)  
    }  
})

*   計算用來顯示 query 結果的 popup 位置（getBoundingClientRect，放置於選取文字區域的下方，需考慮有 scrolling 時的情況）

const getPosition = (el: Range) => {  
    const rect = el.getBoundingClientRect()  
    return {  
        left: rect.left + window.pageXOffset, // pageXOffset: consider scrolling  
        top: rect.top + window.pageYOffset, // pageYOffset: consider scrolling  
        width: rect.width,  
        height: rect.height,  
    }  
}  
  
const { left, top, height } = getPosition(  
    document.getSelection().getRangeAt(0), // 選取範圍區塊  
)  
popup = document.createElement('div')  
popup.style.position = 'absolute'  
popup.style.left = \`${left}px\`  
popup.style.top = \`${top + height}px\`

*   Query ChatGPT API and response with streaming mode（HTTP POST method + stream: true）

const resp = await fetch('https://api.openai.com/v1/completions', {  
    method: 'POST',  
    headers: {  
        'Content-Type': 'application/json',  
        Authorization: \`Bearer ${API\_KEY}\`,  
    },  
    body: JSON.stringify({  
        model: 'text-davinci-003',  
        prompt: \`Explain this: ${queryText}\`,  
        temperature: 0.5,  
        max\_tokens: 1000,  
        stream: true, // get response with streaming mode  
    }),  
})

*   Handle streaming API response (使用 eventsource-parser 套件來幫助解析 streaming data）

import { createParser } from ‘eventsource-parser’  
  
const readStream = (reader: ReadableStreamDefaultReader, callback: Function) => {  
    reader.read().then(({ done, value }: any) => {  
        if (done) {  
            reader.releaseLock()  
            return null  
        }  
        callback(value)  
        return readStream(reader, callback)  
    })  
}  
const streamAsyncIterable = (stream: ReadableStream, callback: Function) => {  
    const reader = stream.getReader()  
    readStream(reader, callback)  
}  
  
const parser = createParser(event => {  
    if (event.type === ‘event’) {  
        const message = event.data  
        if (message === ‘\[DONE\]’) { // chatgpt api streaming 結束時的 EOF  
            return  
        }  
        let data  
        try {  
            data = JSON.parse(message)  
            const { text } = data.choices\[0\]  
            if (text === ‘<|im\_end|>’ || text === ‘<|im\_sep|>’) {  
                return  
            }  
            // text is the streaming API response chunk  
        } catch (err) {  
        }  
    }  
})  
  
// resp 為上面的 await fetch 後回傳的 Response 物件  
streamAsyncIterable(resp.body!, (value: BufferSource) => {  
    const str = new TextDecoder().decode(value)  
    parser.feed(str)  
})

*   content script 與 service worker 的溝通（chrome.runtime.Port.postMessage 用來避免因為切換分頁造成接收 API response 的通道斷線 + chrome.tabs.sendMessage 用來處理右鍵選單的溝通）
*   因 popup 採用 singleton 設計模式，會有多個 request 的 response 資料重疊問題，因此筆者將其當作 MQTT 的方式來處理，開 request 時將傳輸 port name 加上 uuid，收到每個 API streaming response chunk 時，傳給 content script 也會附上其所屬的 port name，用以當成識別 id，避免不同通道的訊息互相干擾

// in background:   
  
// postMessage: (property) chrome.runtime.Port.postMessage: (message: any) => void  
// postMessage（發送訊息）  
port.postMessage({  
    action: 'ans',  
    portName: port.name,  
    result: mes,  
})  
  
// in foreground:  
// onmessage（接收訊息）  
  
port.onMessage.addListener(async (message: any) => {  
    if (message.action === 'ans' && message.portName === currentPortName) {  
        // 將 message.result 的內容 update to popup  
    }  
})

And that’s a wrap! Enjoy. 🎆

👏

My Linkedin: [https://www.linkedin.com/in/%E7%AB%A3%E7%BF%94-%E8%A8%B1-3188a41a1/](https://www.linkedin.com/in/%E7%AB%A3%E7%BF%94-%E8%A8%B1-3188a41a1/)