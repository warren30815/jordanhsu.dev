---
title: "如何下載串流的影片或音訊們-前端工程師實戰思路"
datePublished: Sat Jul 30 2022 07:57:01 GMT+0000 (Coordinated Universal Time)
cuid: clifomxhu01vamfnv3i6v43uh
slug: download-streaming-video-from-frontend-engineer-perspective
cover: https://cdn-images-1.medium.com/max/800/1*cqgCjickMOSRjT4XGgXHnQ.jpeg
tags: streaming, shell, frontend-development

---

### 前言

今早接到了一個在私人投資基金工作的朋友的訊息，因工作需要把聯發科的法說會整理成逐字稿，所以需要把法說會影片下載下來，然後把影片丟到逐字稿軟體再修改一些翻錯的地方，找到了[法說會的連結](https://webpage-ott2b.cdn.hinet.net/webpage/watch?contentProvider=mediatek&v=55627)進去後，一個樸實無華的網頁映入眼簾

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777863118/dbc96dad-b663-40b4-b298-e3e4cf012bd2.png align="left")

大大的iframe影片加上一個聯發科橘的背景，跟簡簡單單的header / footer商標，原本她想說開開心心的按下影片右鍵，就可以準備去享受假日鬆一下了，殊不知….

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777865083/737f2ef0-6aab-4da6-b978-aa97d0b37616.png align="left")

Ok聯發科工程師太狠了，直接鎖右鍵的下載功能🥵

也沒辦法透過網路上那些丟網址然後幫你載影片的連結來載，因此來請我幫忙看看有什麼方法🆘

### **正文開始**

這時身為前端工程師第一步就是報價（x  
按右鍵打開原始碼（o

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777867415/3cfe09fb-3c28-4056-ae24-b37883c581b6.png align="left")

打開後，可以從html DOM tree很容易看出他用iframe來顯示影片

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777869980/1862f065-685c-4ab4-ad3e-1d69667772b8.png align="left")

找到下方javascript下載資料的部分，可以發現最關鍵的一行是在最後loadIframe(“player”, playerLink + vodRssUrlVo\[0\].url);這個部分

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777872468/6a45dd04-b092-4fd3-a3ad-a95c1bec8005.png align="left")

然後我們可以知道，playerLink是定義在上方的一個字串，vodRssUrlVo\[0\].url是vodRssUrlVo那個list of object裡面的url參數，也是一個字串而已  
所以我們把playerLink跟vodRssUrlVo\[0\].url合起來會變成如下的網址（自行去除跳脫字元，\\u0026為&的意思）：  
https://webplayer-cds.cdn.hinet.net/ottplayer/VideoJS/index4.jsp?cp\_name=mcc&playerid=15&ch\_name=ch01&VideoURL=https://mediatekvod-ott2b.cdn.hinet.net/vod\_mediatek/\_definst\_/smil:mediatek/mediatek01\_20220729160302/consult-hls-cl-pc.smil/playlist.m3u8

將上面那個網址貼到瀏覽器後，原本以為這樣找到影片的源頭就可以下載了，殊不知只出現一個全屏的影片，一樣被鎖定右鍵下載了

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777875347/ae31f7bd-6884-4106-8aae-7b89c290cbe3.png align="left")

於是我們換個思路，直接從瀏覽器的開發者工具來監聽網路封包，打開Network視窗後按下重新整理，去看看chrome下載了什麼東西，我們可以發現右下角的initiator有一些從video.js來的檔案

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777878243/fd1b397e-a7c5-423c-8cbe-03e449f5d5f2.png align="left")

點下其中的media\_b2000001\_0.ts後，可以知道他request的url是什麼

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777879800/62ec80d5-96cb-42d5-b808-699b184c17ae.png align="left")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777881626/27a0afb7-c1c6-4023-bc4f-d1edb7ec4da3.png align="left")

把上面的   
[https://mediatekvod-ott2b.cdn.hinet.net/vod\_mediatek/\_definst\_/smil:mediatek/mediatek01\_20220729160302/consult-hls-cl-pc.smil/media\_b2000001\_0.ts](https://mediatekvod-ott2b.cdn.hinet.net/vod_mediatek/_definst_/smil:mediatek/mediatek01_20220729160302/consult-hls-cl-pc.smil/media_b2000001_0.ts)  
貼到瀏覽器後，瀏覽器幫我們下載了一個.ts檔案，打開後正是我們要的影片檔！只是只有前十秒

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777883892/e11687b5-62c8-4320-b11c-7d741c61c0e7.png align="left")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777885532/3605cacb-5d4e-40f6-8758-41d8f16c63c9.png align="left")

這裡的ts不是我們熟悉的Typescript XD，而是一種影片的格式

以此類推，底下的編號\_1, \_2, \_3…分別就是11~20秒, 21~30秒, 31~40秒的影片檔案，於是我們可以簡單寫的for迴圈來自動下載這些檔案

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777887725/22c00351-6eff-465a-9445-51082042f049.png align="left")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777890882/587181c8-055d-4894-b18d-c651b9545214.png align="left")

我們直接將影片拉到最後，得到最後一個片段的編號是377，然後我們可以把上面的code直接貼到開發者工具的視窗，按下enter後瀏覽器就會開始幫我們不斷下載資料，**這邊要注意的細節是每個10秒片段的下載都要間隔一段時間（這邊設2秒）**，原因是聯發科網頁的後端如果短時間收到大量請求，只會接受最後一個，比如我如果直接for迴圈0~377，他只會給我377的資料而已，這樣其實是很合理的設計，因為正常的user不可能同時這麼高頻的切換影片片段，正常情況後端只要知道你現在看的片段的開始跟結束時間，然後給你那一小段附近的資料就好，不然網路頻寬會爆開，老闆收到帳單會直接讓你飛起來🤩（p.s. 我公司的網頁只是租個aws最便宜的機器，然後流量也不高，一個月就要付6000了，雲端的生意還是好賺na，三大雲端設備公司的財報增速也都普遍30% up）

抓完了一堆影片片段後，剩下的就是合起來輸出成一個mp4或mp3音檔了  
這邊我採用的是ffmpeg套件的mac版，可以直接從[官網](https://ffmpeg.org/)下載，然後用來合併與轉檔我們剛剛載的一堆資料，下方直接提供shell script的code供參考：

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777896602/68ee086b-63b3-4b7b-9cf0-1623fa4b6af8.png align="left")

這邊需求因為只是要音檔，所以我轉成mp3，如要影片，可以把最後那行#註解拿掉就可以了，然後只要將上面的code寫成一個檔案，比如檔名叫merge\_video.sh，接著打開terminal，輸入sh ./merge\_video.sh就可以跑起來了，大功告成！

### **總結**

懶的寫總結拉，下班下班🤪

And that’s a wrap! Enjoy. 🎆

👏