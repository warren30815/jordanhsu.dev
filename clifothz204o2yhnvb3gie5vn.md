---
title: "Self-Attention Graph Pooling"
datePublished: Fri Mar 13 2020 08:36:22 GMT+0000 (Coordinated Universal Time)
cuid: clifothz204o2yhnvb3gie5vn
slug: self-attention-graph-pooling
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685778205478/2c1ba442-001b-4a6e-87cb-06e90a889c7a.png
tags: ai, graph

---

#### Self-Attention Graph Pooling

paper link: [https://arxiv.org/abs/1904.08082](https://arxiv.org/abs/1904.08082)

發表於 ICML 2019

> 一言以敝之，這篇paper藉由在GCN公式加上一trainable attention參數（attention mask），來做到同時考慮node features and graph topology的graph pooling方法，如下圖

### 前景提要

在CNN的常規操作中常搭配pooling，用來避免overfitting和降維，擴展到graph中，近年來graph convolution的研究遍地開花，也取得了很好的成績，但graph pooling的研究卻遠少於graph convolution，因此，本文提出了基於Self-Attention的graph pooling方法，並經實驗證實在合理的時間複雜度下，於小dataset和大dataset都取得目前最好的成果

### Graph pooling方法overview

目前的graph pooling可分為三種：topology based, global, and hierarchical pooling.

簡單來說，topology based的方法劣勢是沒很好利用到graph features的資訊量；global的方法劣勢是沒很好利用到graph structure的資訊量（例如hierarchical的資訊）；而hierarchical的方法則是feature and topology-based的折衷

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778178737/57e79b86-f321-4e6d-a025-8d98ed0cc6db.jpeg align="left")

左邊為global，右邊為hierarchical，readout下面會解釋

> hierarchical方法在大dataset上有更好的效果，但在小dataset（結構資訊不複雜）的情況下，用global的方法反而較合適，因為考慮了全局的資訊量，本文也藉由實驗證實這件事

### 本文開始

self-attention又稱為intra-attention，原因是他讓input features本身做為attention大小的判斷準則，舉個例子，如果我們的GNN是用[Kipf & Welling](https://arxiv.org/abs/1609.02907)所提出的一階近似表達式，那我們的self-attention score可以寫成

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778180981/39f33f32-d47f-46d8-b16d-2190f0288cd1.png align="left")

theta att是本文方法的唯一trainable的參數

可以看出，藉由在GCN公式加上一trainable attention參數，來做到同時考慮node features and graph topology的效果（因為式子裡面有A又有X，分別代表圖的adjacency matrix和feature map）

然後根據上面的分數，在pooling時drop掉一定比例的node，例如pooling ratio = 60%時，只有self-attention score排名前60%的node會被留下進入下一層（所以A跟X的大小會縮減）

而算這個Z的算式中的GNN也可以做一點變化，例如add the square of an adjacency matrix creates edges between two-hop neighbors（如下A²）

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778182272/de0bc3dc-20fd-45cd-ab87-488cd23371d4.png align="left")

或者疊兩層（經實驗證明效果好一點點）

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778184609/e7efb2cf-2140-43e5-b1bb-cda217e0639a.png align="left")

或者用multi-head attention的想法，平均多個GNN結果

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778186805/57206092-5d5b-409e-adc5-dbb60e29dd4a.png align="left")

本文中M = 2 or 4

![](https://cdn-images-1.medium.com/max/800/1*rLPSt1VOXONhl47Seh2lMg.png align="left")

但就實驗結果來說，以上和式3其實差不到1%的準確率而已

> 模型架構

如下圖，readout用途是aggregate node features to make a fixed size representation

Global pooling結構包含三個卷積層，將它們的輸出串接起來得到的 features送入pooling layer & readout，最後得到的features送到MLP（a.k.a fully-connected layer）做分類；Hierarchical pooling結構則為做一次卷積，做一次pooling，最後將三次readout的結果加起來使用MLP來分類。

![](https://cdn-images-1.medium.com/max/800/0*deFwQTFrwVAxf9cv.jpg align="left")

readout式子如下

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778193394/30e83d50-1bcb-4b61-849f-847b0e01d8e0.png align="left")

串接符號左半部是global average pooling概念；右半部是global max pooling概念，經[Cangea et al.](https://arxiv.org/abs/1811.01287)的實驗證實這種readout方法效果更好

### 實驗部分

> Datasets

前兩個為較大的dataset，後三個為較小的dataset

![](https://cdn-images-1.medium.com/max/800/0*Otl7w6ZGVZ0i9XN2.jpg align="left")

> 實驗結果

可以看出在前兩個較大dataset，hierarchical的方法表現較好；後三個較小dataset，global方法表現較好，原因分析如前面章節 ”Graph pooling方法overview” 所述

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778197079/12db97a7-8251-4e87-af5b-9a037d81e51e.jpeg align="left")

裡面的其他model可以參考[Taki](https://zhuanlan.zhihu.com/p/104837556)的介紹

下圖為不同GNN結構或輸入（如式7,8,9）帶來的表現差異，就實驗結果來說，其實差不到1%的準確率而已

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778200619/ee1c9ac6-bf0f-40da-b065-4461ca5c8391.jpeg align="left")

> Scalability

可看出本文提出的model並不受輸入大小影響，具有scalability

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778202913/7d3fc073-5c6b-4260-8778-699d14670eec.png align="left")

k為pooling ratio

> Limitations

model中使用了pooling ratio k，來處理various size的input graphs。在SAGPool，很難parameterize the pooling ratios to find optimal values **for each graph**，為了解決這個問題，我們使用二分類來決定node的保留與否，但是這沒有完全解決問題

### Open issue

How to improve the effectiveness and computational complexity of pooling is an open question for investigation

And that’s a wrap! Enjoy. 🎆

👏