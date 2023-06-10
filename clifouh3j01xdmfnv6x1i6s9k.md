---
title: "探討graph attention機制有效性 — Understanding Attention and Generalization in Graph Neural Networks"
datePublished: Mon Mar 09 2020 15:08:16 GMT+0000 (Coordinated Universal Time)
cuid: clifouh3j01xdmfnv6x1i6s9k
slug: understanding-attention-and-generalization-in-graph-neural-networks
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685778250122/306309b4-82e6-4fab-a5de-cbcae49dfb09.png
tags: ai, graph

---

paper link: [https://arxiv.org/abs/1905.02850](https://arxiv.org/abs/1905.02850)

發表於NIPS 2019

### 前景提要

自從graph attention network (GAT)提出以來(不熟悉GAT的話可以參考下面台大李宏毅教授的助教課影片)，attention的概念變成訓練graph相關的神經網路時，一個很powerful的工具，attention物理意義上就是某一節點對其他不同節點應有不同關注度，例如你喜愛的偶像或家人說的話對你的影響力，相比一個普通朋友說的高出許多

<iframe src="https://www.youtube.com/embed/eybCCtNKwzA?start=2071&feature=oembed&start=2071" width="640" height="480"></iframe>

但是，要量化或學習attention的大小並不容易，更不用說可以事先知道attention的值（使supervised式的訓練方法窒礙難行），甚至可能弄巧成拙，使model的效能下降（因為過度關注了不重要的node），例如當你只看中\*電視台（node），你對於事物的判斷就會有所偏頗，甚至覺得自己可以Fa da tsai 💰

> 因此，本篇paper藉由不同實驗來探討graph attention機制有效性，並提出一個weakly-supervised的訓練方法來近似supervised的結果，在real dataset上也得到了性能提升

### 本文開始

在graph neural networks (GNN)中，attention可以被定義在edge上（i.e. edge weight），也可以在node上（i.e. node weight），本文的分析主要focus在node weight上

> Attention meets pooling in graph neural networks

> 首先，作者將attention的概念融入pooling，用attention來做graph pooling

借鑒Top-k pooling，對於那些不重要的node（weight低於某threshold，paper內實驗的取值範圍 from 0.0001 to 0.1），不是給他們一個較低的weight，而是直接drop他們，強迫模型更focus在那些剩下的重要節點

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778211544/af5f1297-b575-4d7a-a962-774268963a94.png align="left")

本文提出的attention-based pooling的方式

Top-k pooling不是uniformly sampled，所以這方法的好處是可以select到不同的局部特徵點，但很明顯的，這種直接丟掉節點的方法會破壞graph structure，甚至產生isolated nodes，只是在本文的實驗中，一個點的周圍節點通常會有相似大小的attention，換句話說就是你跟你朋友們會同時被選上或落選，因此isolated nodes的問題於本文中並沒這麼嚴重

當attention layer應用在input層時，具有很好的interpretability（since the network makes a decision only based on pooled nodes），而本文也是這麼設計的，概念如下圖

原圖經過trainable attention-based pooling layer萃取出重要的節點，其餘捨棄

> How to train an attention-based pooling layer

法1: train一個projection vector **p（在後面的討論會知道在unsupervised時p的初始化很重要；supervised則沒差）**, then

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778213704/d83658b8-a447-4957-b5d6-31bed50a582d.png align="left")

X為node feature map

法2: train一個GNN

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778215727/ebc41c5c-af5a-4a64-b0a6-d27f8d366a59.png align="left")

A is the adjacency matrix of a graph, X為node feature map

並經過softmax來provides more interpretable results and encourages sparse outputs

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778217546/84bb2f26-c0e0-4139-87b0-85bb4e458f49.png align="left")

為了在supervised or weakly-supervised的情況都能計算loss，作者使用了KL散度（衡量兩個分佈的差異，在概率論與統計中，我們經常會將一個複雜的分佈用一個簡單的近似分佈來代替。KL 散度可以幫助我們測量在選擇一個近似分佈時丟失的信息量。），因此training的total loss為（後面會詳細解釋）

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778219814/d13d199f-baca-425f-ad8f-e49d9e155101.png align="left")

參數意義可參考paper section 3.3

> Another model

在本文的一些實驗中，Graph Convolutional Networks (GCN) 和 Graph Isomorphism Networks (GIN，運用了sum aggregator聚合完節點特徵後加上更多的fully-connected layers，在判別graph structures時有更好的性能）的表現並不好，因此本文也結合GIN和ChebyNet，提出一個更stronger的model - ChebyGIN

### 實驗部份

> Datasets說明

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778222977/0b6488cd-0439-4ac6-b3de-215134041d11.png align="left")

1\. Color counting task (上圖a）

把原圖各節點著色，計算有幾個綠色節點，因為範例有兩個綠色，所以其Ground truth attention各為1/2 = 0.5，在這個task中， the graph structure is unimportant and edges of graphs act like a medium to exchange node features

2\. Counting the number of triangles（上圖b）

數出原圖中有幾個三角形，這任務各個node的Ground truth attention可以透過下式來計算，比如範例圖共有2個三角形，由圖可知四個跟三角形有關係的node的Ti分別為1 2 2 1，1+2+2+1 = 6，因此attention為0.2的節點是由1/6四捨五入得來；0.3是由2/6四捨五入得來

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778225517/2c9ebc0c-f5d1-42d5-a423-c0e46483c37d.png align="left")

3\. MNIST-75SP（上圖c）

為了測試GNN在irregular grids上的能力，作者根據前人的論文改良了MNIST dataset，將image在避免遺失essential class-specific information的前提下，以a small set of superpixels 來建出一個graph代表原本的image，這個graph的node features為average intensity value to all pixels within a superpixel和超像素的重心座標；edges為超像素中心之間的空間距離， Ground truth attention被定義為

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778228579/fc226f78-8425-4da8-8b42-688c25c3f745.png align="left")

作者對每張image取N ≤ 75個superpixels, 因此這dataset被稱為MNIST-75SP

4\. Molecule and social datasets - more practical cases

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778230898/7f54a491-bb0f-410c-84e8-5aa40c6868ea.png align="left")

在practical cases中，作者用了graph classification任務的benchmark datasets，如蛋白質結構dataset: PROTEINS and D&D；以及scientific collaboration dataset: COLLAB，實驗中，作者想探討attention-based model在inductive任務的能力，因此根據節點數量分割graph dataset，例如PROTEINS中，we train on graphs with N ≤ 25 nodes and test on graphs with 6 ≤ N ≤ 620 nodes

下圖為前三個任務的train&test data examples

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778233766/331bbba1-c2ff-42b6-aced-b66f1f72b83a.png align="left")

> Generalization to larger and noisy graphs

attention的一個能力是generalize to unseen, potentially more complex and/or noisy, inputs，藉由減少關注這些爛nodes來更好的訓練，因此這篇paper在test case上把原本的data做些變換，例如上圖color任務的TEST-LARGE增加不是答案的節點（非綠色）或TEST-LARGEC增加不同unseen顏色的nodes，這些以下統稱為雜訊，作者想藉由with and without attention的GNN來探討the limits of GNNs with attention，和attention在哪些任務會work，哪些任務反而有害（focus不重要的nodes or drop重要的nodes）

> Loss function and training

COLORS and TRIANGLES任務中要minimize the regression loss (MSE)，其他任務則minimize cross entropy (CE)，後項的KL散度要衡量的分佈是ground truth attention和predicted attention

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778236014/1628c10a-176f-491b-8363-6f21261b563b.png align="left")

參數意義可參考paper section 3.3

> Weakly-supervised attention supervision

現實情況中，預先知道ground truth attention幾乎是不可能的，因此本文提出了一種弱監督的訓練方法來估算attention，藉此optimize上面的total loss

首先我們想train一個model A，但我們沒有ground truth attention，所以我們可以先train一個和model A長的一樣的model B，只是model B沒有attention/pooling機制，只有最後的global pooling (e.g. readout）來做分類機率輸出，因此model B只有optimize MSE or CE的loss，訓練好之後，我們可以用以下式子來藉由model B估算attention：

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778238051/228a61b4-66e5-4b59-9142-bc92a45df9ac.png align="left")

y是原圖分類的prediction；yi是原圖去除node i的分類prediction

概念就是我藉由觀察有node i跟沒有node i，機率分佈的結果會差多少，如果沒什麼差，代表有沒有這node根本不重要，其attention自然就低，因此藉由上式&model B，可以得到估算的attention給model A訓練用

### 個人認為這方法只能知道node對整體graph的影響，不能知道是好的影響還是壞的影響，相信這也是之後可以改進的部分

### 結果分析

![](https://cdn-images-1.medium.com/max/800/1*inaggtq98_6xqqJKWu5MKQ.png align="left")

從上表可看出，運用attention over nodes可以讓model generalize to more complex or noisy graphs at test time，尤其可以從COLORS的結果看出，GIN運用global pooling和運用attention-based pooling的結果，差了60幾%

從實驗結果也可以看出本文提出的weakly-supervised方法shows performance, robustness and relatively low variation (i.e. sensitivity to initialization) similar to supervised models and much better than the unsupervised model

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778242362/dc6dd55b-fe09-4ddd-91f1-404929e679e2.png align="left")

在真實dataset中，supervised attention是不可行的，而本文提出的弱監督訓練方法也取得了比無監督更好的結果

> What are the factors influencing the performance of GNNs with attention?

1. **initialization of the attention model (i.e. vector p or GNN)**
    
2. strength of the main GNN model (i.e. the model that actually performs classification)
    
3. other hyper-parameters of the attention and GNN models.
    

第一點是最重要的，我們可以從下面兩張圖探討

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778245452/237f9e1d-1f55-4df0-91e6-b0aaff92ac8b.png align="left")

無監督attention的結果（容易卡在local optimal）

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778248052/01132bd7-890d-4da9-a1d4-557f66d3c0b9.png align="left")

監督attention的結果

> Why is the variance of some results so high?

Caused by the initialization of other trainable parameters of a GNN, but we show that once the attention model is perfect, other parameters can recover from a bad initialization leading to better results. The opposite, however, is not true: we never observed the recovery of a model with poorly initialized attention

### 結論

attention機制在GNN是非常powerful的，但前提是有個好的**initialization of the attention model（still an open issue），反之則會損害模型**，paper也透過實驗證明，attention can make GNNs more robust to larger and noisy graphs，本文提出的attention-based weakly-supervised訓練方法也能在real dataset中work

And that’s a wrap! Enjoy. 🎆

👏