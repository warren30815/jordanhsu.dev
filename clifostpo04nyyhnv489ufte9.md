---
title: "Adaptive Structural Fingerprints for Graph Attention Networks"
datePublished: Mon Mar 30 2020 08:30:38 GMT+0000 (Coordinated Universal Time)
cuid: clifostpo04nyyhnv489ufte9
slug: adaptive-structural-fingerprints-for-graph-attention-networks
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685778174060/a9c8c36a-abf8-4f0e-860d-232313d40d8a.png
tags: ai, graph

---

paper link: [https://openreview.net/pdf?id=BJxWx0NYPr](https://openreview.net/pdf?id=BJxWx0NYPr)

Published as a poster in ICLR 2020

> 一言以敝之，這篇提出一種同時考慮到node feature和結構feature的方法來計算attention score，且不侷限在只能聚合1-hop鄰居

先看結果

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778146640/46864b02-5da7-478e-a9ca-9a4436e84deb.png align="left")

在三個transductive任務（transductive不知道是什麼的話見下圖最下面）中，均展現了不錯的提升，經由ablation實驗證實歸因於加入的結構資訊

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778148808/a4ab47c8-b0d5-4cd7-8899-6c069917864c.png align="left")

transductive的目的是最佳化手上擁有的全部data的預測結果，因此對於預測完全沒看過的data效果通常很差

在看ablation實驗的結果之前，我們先來看看GAT如果將更高hop的鄰居資訊量也考慮進去的結果會如何

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778150709/5e41d4dd-a720-4082-b44b-f595fe261c56.png align="left")

可以看到，將更高hop的鄰居一起考慮對GAT有害無益（即GNN的over-smoothing問題）

上圖可以理解為，雖然加入更多的鄰居可以帶來更多資訊量（i.e. 好朋友），但也會帶來更多noise（i.e. 壞朋友），而GAT這種僅僅考慮鄰居feature（以下以content代替）間的重要程度的方法，不足以對抗這些壞朋友，而效果跟著走下坡，**也反應了GAT並不能很好的利用multi-hop的資訊**

**那這篇會不會也有一樣的問題呢？**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778153123/197be511-1049-4022-a40a-78dca4b8d60a.png align="left")

本篇的實驗結果說明了同時考慮content和結構資訊（比重為train出來的）在2-hop時能提升效果

> 當然，一篇paper不會只有優點沒有缺點，優缺點總結的部分我們放在最後的討論說明（當然缺點的部分paper內都沒提到XD）

接下來我們來正式介紹這篇，將以know what 👉know why 👉know how的順序做為本文的講述方式

### Know what

這篇文章主要是針對GAT來做加強，那GAT（i.e. Content-based attention）最大的缺點是什麼呢？

👎over-smoothing問題，只能聚合1-hop的鄰居資訊，如上上圖ablation實驗數據，**更多hop只會有害無益**

因此本篇的know what就是想要解決GAT沒辦法運用multi-hop資訊的問題，並想探討利用結構資訊能否減輕over-smoothing，並達到提升semi-supervised的節點分類任務accuracy的效果

### Know why

那為什麼要解決這個問題？**利用結構資訊是什麼概念？**

在如下圖case中GAT無法為node A、B計算attention score（因為沒有direct connection），但node A、B都strongly connected到這個dense community，B甚至還直接connect這個community的hub（藍色中心點那個），所以node A、B之間也應該要或多或少影響彼此:

![](https://cdn-images-1.medium.com/max/800/1*lVMCbPz7W-SMv85fI5zPbg.png align="left")

GAT並不會有node A、B之間的attention score

以上可以理解為A、B都加入了同樣的讀書會，在成員間互相分享知識時，即使原本不認識，也或多或少能受益

### Know how

那如何利用結構資訊呢？本篇提出的方法稱為ADaptive Structural Fingerprint (ADSF)，比如我們想知道下圖紅色node i和其他k-hop鄰居的關係強度

首先我們要先算出node i對於每個k-hop鄰居nodes的closeness（以node weight表示），下圖那個3D model顏色越紅代表weight越大 a.k.a 越close，建構完node i的closeness分佈（一個size為(n,1)的向量，n為node數量）後，以此類推我們可以得到graph中的每個node各自的closeness分佈（n\*n的matrix）

結構指紋其實就是k-hop subgraph的概念而已，看接下來的圖會更有感為什麼作者稱closeness分佈為“指紋”

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778157885/e470b7bb-2f7b-4947-857b-e01f81c8e519.png align="left")

上下圖為兩種不同的算closeness分佈的方法，最右邊為node weight的等高線圖表，長的跟指紋87%像（？

在上圖中，左邊為the local subgraph/fingerprint；中間為visualization of the weights；右邊為contours of the weights，可以看出右下的方法能更好擬合結構資訊，接下來主要會介紹這個算closeness分佈的方法 - Random Walk with Restart（RWR）

RWR擴展random walk，增加一個回到原點（restart）的機率，式子如下：

> r = cWr+ (1-c)e

> cWr是random walk部分，(1-c)e是回去起點的部分

c為一\[0,1\]的機率，在本文中設置0.5有最好的結果；W為column-normalized（make each column sum to 1）後的adjacency matrix；r為一 (n,1) 的向量，代表node i對於graph中其他所有點的weight，迭代上式可以收斂；e為一個(n,1)的one-hot向量，唯一那個1代表node i的位置（原點），r的初始值設為e

收斂形式為：

> r = (1-c)( I — cW)⁻¹e

> I為identity matrix，推導過程可[見此](https://medium.com/@chaitanya_bhatia/random-walk-with-restart-and-its-applications-f53d7c98cb9)

藉由對graph每個node都算出上述的r，我們可以得到n個(n,1)向量，即每個node各自的”指紋“

藉由這n個(n,1)向量，我們就可以知道任兩個node i, j間的結構關係強度，以[廣義的Jaccard係數](https://baike.baidu.com/item/Jaccard%E7%B3%BB%E6%95%B0)來衡量兩個指紋之間的相似性：

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778160321/3043e4fc-bb28-4f70-9448-49aae7f0b248.png align="left")

接著我們就能把這個結構關係強度和content的similarity強度結合來計算attention score了，overview如下圖

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778163026/400259b0-2982-4134-b530-b963e07c7e4e.png align="left")

structural interaction即為上述的結構關係強度

其計算content similarity的方法和GAT完全一樣，因此就不復述，上面的structural interaction和content similarity會以一trainable的比例構成attention layer，而本篇也和GAT一樣，用了8-head attention layer

### **實驗部分**

用了三個citation network的dataset，來做semi-supervised的節點分類任務，dataset statistics如下：

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778164664/f1cfbafe-4686-4df8-922a-7fe81cd66730.png align="left")

三個datasets其實格式差不多，以Cora數據為例可參考[這篇文章](https://blog.csdn.net/weixin_39373480/article/details/88742200)

> 實驗流程

因為是transductive的節點分類任務，上表左邊三個代表dataset overview，以Cora為例，意思就是有2708筆labels，右邊的#Training代表training data用了2708筆node features（那些0&1），但每個class各只用了20筆label，20\*7=140，所以有label的資料只有140個nodes，其他2708–140=2568個nodes在train的時候是沒有label的，只有用node features。最後#Testing則是取出2708 nodes其中的1000個nodes的label去驗證accuracy

> 實驗結果：

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778166087/42189232-0a16-44ad-a5a7-a959deefe891.png align="left")

提升了1–2%

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778170453/1044e293-e00f-44bf-b20a-57293b012163.png align="left")

GAT的震盪較多

> Ablation experiments

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778172195/c00df8a8-8e2e-4fb3-a255-e9b5ef1f570e.png align="left")

上圖(a)為Cora dataset的結果，顯示2-hop的鄰居都會在message passing時以不同的attention score被聚合，相較GAT，引入high-order structural details對於over-smoothing的問題有一定的減緩

上圖(b)為RWR的restart probability不同時的結果，可以發現固定在2-hop時，c=0.5效果最好

上圖(c)為不同hop時node weight的衰減，可以看到3-hop之後的node影響力幾乎是0了

> 個人覺得，k-hop的限制本質上等同於一種保護，對你影響最直接的人一定是周邊的人，但並不代表k-hop以外的一定全部都是noise，如現實生活中的電視媒體或知識型youtuber（e.g. [啾啾鞋](https://www.youtube.com/user/chuchushoeTW)、[柴鼠兄弟](https://www.youtube.com/channel/UC45i13dEfEVac2IEJT_Nr5Q)），對你的思考可能也會有很大的影響，但noise肯定也是有的，看看那些常看中\*新聞的人🤫

### 結論

> paper優缺點

優點：

1. The paper is easy to read
    
2. Introduce the combination of structural information to GAT
    
3. Have a few more parameters compared with GAT (for the Cora dataset, about extra 0.03% parameters of GAT), but get 1~2% accuracy enhancement
    

缺點：

1. RWR其實已經是2004年的老方法了，其計算( I-cW)⁻¹的時間複雜度是O(n³) (by Gaussian elimination). The overall complexity is determined by the complexity of matrix inversion as matrix-vector multiplication has the space complexity O(n²)，因此導致本文提出的方法**很難用於large-scale real-world dataset**，甚至GAT中的PPI inductive task也無法在時間內跑出10次平均的結果
    
2. 不能調整聚合content資訊和結構資訊的比例，因為是train出來的，可解釋性比較低
    
3. Bad code readability，變數名稱命名的很差
    

> [ICLR official reviewer comment](https://openreview.net/forum?id=BJxWx0NYPr)

“While I believe the general idea indeed has merit and empirically shows great promise, I believe the paper in its current state is not ready for publication. However, I believe that a more thorough revision can lead to an a publication with potential impact on the applications side.”

And that’s a wrap! Enjoy. 🎆

👏