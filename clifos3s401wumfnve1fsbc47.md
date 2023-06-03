---
title: "CVPR2018 — 可解釋的CNN（spotlight）"
datePublished: Sun Aug 02 2020 06:22:09 GMT+0000 (Coordinated Universal Time)
cuid: clifos3s401wumfnve1fsbc47
slug: cvpr2018-e58fafe8a7a3e9878be79a84cnn-spotlight-6a38361eece5
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685778140218/36a5c94c-cfd6-45dc-beaf-1825399c683c.png

---

#### Interpretable Convolutional Neural Networks

paper link: [https://openreview.net/pdf?id=BJxWx0NYPr](https://openaccess.thecvf.com/content_cvpr_2018/papers/Zhang_Interpretable_Convolutional_Neural_CVPR_2018_paper.pdf)

Spotlight in CVPR 2018

**可解釋人工智慧**（Explainable AI，或縮寫為 XAI）這個研究領域所關心的是，如何讓人類瞭解人工智慧下判斷的理由。在 2016 年 AlphaGo 與李世乭對戰五番棋的過程中，AlphaGo 多次下出人類圍棋專家未曾想過的棋步。當時觀戰的樊麾曾經評論「這不是人類會下的棋步，我未曾看過人類下出這一步（It’s not a human move. I’ve never seen a human play this move.）」。我們對於 AlphaGo 的棋力可能會有出於好奇心與競賽需要的研究，不過如果人工智慧相關技術要推廣到更多的領域，例如在法院裡協助量刑、在醫療上協助診斷、在保險與金融上判斷一個投資策略的優劣，或是在社會福利政策裡主導資源的分配，我們都會更迫切需要知道模型到底怎麼得出結論的。\[1\]

但目前的CNN based的架構過度複雜，一個filter可能身兼數職，如下圖：

上面是這篇paper的CNN filter的效果，一個filter專注檢測某項object-part（例如熊貓頭），給出了判斷最終輸出的合理依據；下面是一般CNN的效果，一個filter可能用了不同的side information（有時可能是nonsense）來判斷最終輸出

基於上述的觀察，作者希望能讓每個filter專注檢測某項object-part就好（例如熊貓頭），針對不同input，都有對應負責的filter可以做決策，期待model can **learn a better representation**

上述以程式開發舉例就是模組化的概念；以現實生活來說，就是招募人才是HR部門負責、產品研發是RD部門負責…，而不用什麼事都要董事長來參一咖，因此，這篇paper的貢獻一言以蔽之就是：

> **Designed a filter loss to disentangle each high conv-layer filters, without** any annotations of object parts or textures.

### Know how

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778122958/76ae11b6-1c3c-40b4-afac-e5adc0480745.png)

上圖是傳統CNN與這篇的CNN的差別，這篇其實就是把Conv layer的loss部分做改良，除了原任務的loss（例如cross-entropy）外，額外加上一個mutual information的loss（接下來會解釋），所以上右圖的綠色部分不是額外加了什麼layer，只是作者的畫圖表達方式不同而已

除了上述loss外，還有額外加了一個mask，主要概念是抑制分散的activation（如下圖），讓feature map能focus在某一區塊（object-part）

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778124863/bc290d89-207f-473d-9c6c-b3231a741a58.png)

我們無法確保傳統的CNN會學到什麼，因此他的檢測結果很容易受到dataset的不同而影響performance

接下來產生了幾個問題：  
1\. mask是要有ground-truth的label，我才能指導model去學習物體的哪些部分是有意義的嗎？  
2\. 我怎麼知道要mask哪裏，才能抑制noisy activation & 留下預測所需的關鍵資訊？（information bottleneck theory \[2\]）  
3\. 那個mutual information是誰跟誰的mutual information，為什麼能讓filter們各司其職？

第一個問題的答案是**NO**，不然也不用在這邊講這篇paper了xD

要回答第二個問題，首先我們先介紹paper中如何定義mask的，假設一張輸入圖片經過conv和relu後得到feature map（下稱為x）的shape是3\*3，那我們就會有3\*3張masks（對應每個x的pixel），每張mask的shape也都是3\*3，如下圖，每張mask的能量分別會以一個點為中心像四周衰減（L1-norm）

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778126472/94814752-670f-49f0-ac51-22ff76e845e0.png)

paper中的做法是會去看x的pixel值在哪個座標最大（relu後的值最大，有一點pooling的味道），就去用哪張mask做x ◦ mask，用意是確保x只會激活某個region，以此來filter out noisy activation，可以把以上概念想成我今天要檢測貓，然後filter可能會同時用部分貓腳、貓耳朵、貓臉來聯合檢測，但我希望一個filter只要foucs在一個object-part就好，所以會用mask的方式只留下x上最明顯的區域（下圖）

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778128867/6b362860-8b94-4790-ac9a-b594fa4fabde.png)

而如何”留下預測所需的關鍵資訊”呢，filter的優化目標除了原任務的loss外，加上了針對不同的x，我們如何選擇一個合適的mask，paper中的做法是最大化所有x（下稱X）與所有masks（下稱T）之間的mutual information，代表當我今天拿到一個x，我能最大程度的降低要選擇哪個mask的不確定性（i.e. 如果我選擇每個mask的機率是一樣的，那我等於不知道哪個mask對x合適，所以我的目標是要確定選某一個mask，這樣我的不確定性就是0），上述formulate成數學式的話，如下圖：

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778130434/3351b751-bd5a-4256-8566-dd8bc7c22f3e.png)

最後一項為原任務loss

其中

![](https://cdn-images-1.medium.com/max/800/1*Tg4DE1KazGC0MgutwYoGOA.png)

目標：最大化mutual information（MI），因此loss function前面有負號

值得注意的是，filter loss這件事在此paper只有考慮在high-layer的conv，原因是low-layer的conv通常會學到一些common senses，例如圖片的形狀、紋理、顏色…，而high-layer的conv會開始學到task-specific的資訊，因此只需對high-layer的conv增加限制、希望高層的conv can **learn a better representation**即可

最後回答第三個問題：為什麼能讓filter們各司其職。還記得剛提到我們希望每個filter只負責一個object-part嗎（例如當鳥頭出現時，對應的filter會做事，即根據其x最大的pixel值的座標，從T裡（更精確來說，是T+）挑一個對應位置的mask指定給它；不相關的filter就保持安靜，即指定給它一個全黑的mask，下稱T-），首先，我們需要在training process determines the target category for each filter，paper中是依據filter對於某類型image set得到的x的pixel值平均後來決定filter對哪個類別特別有興趣，就讓該filter負責該類型圖片的檢測（如下圖）

![](https://cdn-images-1.medium.com/max/800/1*00oIBt23tz5JxiGl5I4j7w.png)

c為圖片類別，例如狗、貓之類的

定義好filter的類別後，我們來改寫一下上面提到的MI（數學推導見\[3\]）

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778137219/dfd90bf0-8259-40d9-9a1b-8844f483e032.png)

紅框部分為常數，代表prior entropy of masks   
藍框部分目標為Low inter-category entropy，T’ ∈ {T-,T+}，When we know x, we need to assure T- or T+ (category c or not) to minimize the conditional entropy  
棕框部分目標為Low spatial entropy, T+為 all positive masks. When we know x, we need to assure the one mask to minimize the conditional entropy. 而不是選所有mask的機率都一樣（這樣entropy會高）

以上，優化過程中藍框保證了filter should only be activated by a certain category c and keep silent (i.e. fit to negative mask) on other categories；棕框保證了filter should only be activated by a single region of the feature map x

### Experiments

paper寫得很清楚了，懶人包的話可以直接參考我的報告ppt \[4\]

### Conclusion

*   Proposed a general and unsupervised method to disentangle high conv-layer to enhance their interpretability
*   Each filter is more semantically meaningful than traditional CNN
*   CNN’s classification accuracy may decrease a bit.

### My rethink

*   One filter is encoded to only one object-part. It sounds low flexible.
*   Disentangled high conv-layer filter to focus one object-part, making it hard to catch some common senses between objects.
*   The way to decide filter class and selected mask seems to be naive.
*   I speculate the design may make the model easier to know what it doesn’t know.

And that’s a wrap! Enjoy. 🎆

👏

### Reference

\[1\] [Explainable AI 是什麼？為什麼 AI 下判斷要可以解釋？](https://medium.com/trustableai/%E5%AE%83%E6%98%AF%E6%80%8E%E9%BA%BC%E7%9F%A5%E9%81%93%E7%9A%84-%E8%A7%A3%E9%87%8B%E6%B7%B1%E5%BA%A6%E5%AD%B8%E7%BF%92%E6%A8%A1%E5%9E%8B-f18f57d18d4f)

\[2\] [Information bottleneck theory](https://zhuanlan.zhihu.com/p/29579424)

\[3\] [原paper arxiv版（證明見第十一頁）](https://arxiv.org/pdf/1710.00935.pdf)

\[4\] [自製的google slide](https://docs.google.com/presentation/d/14GskOYX3xQCQgal_23YrAurEK1FgY8u8fsC9Q_zw84A/edit?usp=sharing)