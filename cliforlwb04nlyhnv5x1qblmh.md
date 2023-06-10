---
title: "Adversarial Attack on Community Detection by Hiding Individuals"
datePublished: Wed Sep 02 2020 14:30:31 GMT+0000 (Coordinated Universal Time)
cuid: cliforlwb04nlyhnv5x1qblmh
slug: adversarial-attack-on-community-detection-by-hiding-individuals
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685778117347/98b0ca2d-04e0-47be-8d91-5a3e24c563cf.png
tags: ai, graph, adversarial-attack

---

paper link: [https://dl.acm.org/doi/pdf/10.1145/3366423.3380171?casa\_token=N5jr2JHp3KMAAAAA:2BcySv\_APTzji9nikXSFJhWvVbr8xkyFKCqdcESwZxHV9waZPP0MMec-RmSZJzi-WGsRmuJzqet9](https://dl.acm.org/doi/pdf/10.1145/3366423.3380171?casa_token=N5jr2JHp3KMAAAAA:2BcySv_APTzji9nikXSFJhWvVbr8xkyFKCqdcESwZxHV9waZPP0MMec-RmSZJzi-WGsRmuJzqet9)

Published in KDD 2020

此篇為這issue的第一篇paper，故提出的方法實驗效果並沒有很好，只是題目新穎能帶來啟發

### 前言

Community detection (CD) 是什麼？簡單來說就是圖上nodes的聚類，同一類之間會比較dense，類跟類之間會比較sparse

每種顏色為同一類

Adversarial attack in CD是什麼？可以微調（perform small perturbations）圖的nodes、edges或features（例如增刪node, edge），來讓CD的演算法聚類的結果不同

如下圖，原本黃色類別的target nodes，在砍掉一條edge後，被同樣的CD演算法分為紫色類別了

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778074460/6aa9775f-d0ac-41aa-81b4-8624450233a7.png align="left")

攻擊這件事能幹嘛？舉個例子，電商上有許多商店，假設黃色類別代表評價差，紫色類別代表評價優良，透過攻擊電商的CD系統，我就能讓目標商店（target nodes）混入評價優良的類別，得到更多曝光率

**註：在graph上，通常砍edge比加edge會有更多的影響，by 指導教授 in 清大資工**

### 問題定義

給定一個CD演算法，要train一個生成adversarial graph的graph generator，目標是讓給定的一組target nodes **能分散地隱藏進不同的communities中，可以理解成要把一些身份特殊的人混入人群中，讓他們不被發現（如下圖）**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778076629/47b5d056-e616-4d5b-b846-d94100a339b2.png align="left")

此paper的Intuitive example，原本node 8, 9是在同一類別，刪掉0跟15之間的edge後，node 8, 9就被歸類到不同communities了\*\*（分散地隱藏）\*\*

另外此篇paper簡化問題，生成adversarial graph時只考慮edges的增刪，並限制一個budgets，限制改變的數量（例如增刪加起來總共只能改變5條edges）；另外 focus on non-overlapping communities，一個node只會屬於一個community

### 方法

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778078980/ecae837d-c425-4a07-a348-a2662cd7a5d7.png align="left")

整體流程圖

整體分成兩個modules，為constrained graph generator跟surrogate community detector，constrained graph generator部分的輸入是乾淨的graph，輸出是改動過的攻擊graph，constrained的意思是能增刪的edges在一budget數目以下；surrogate community detector代表可替換的CD演算法，會藉由兩個loss指導generator生成的攻擊圖要符合兩個原則：  
(1) 乾淨圖跟攻擊圖的每個node的embedding要相似（i.e. all node embedding之間的KL散度要低）  
(2) target nodes之間要盡量被CD assign到不同的communities（i.e. target nodes之間預測屬於每個communities的機率的向量，兩兩KL散度要越大越好，代表兩兩不同target nodes的communities類別預測的機率分佈長得越不一樣、越不同類）

### 細節

● Surrogate community detection model

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778080455/297abba4-6495-4aae-9b46-38cc0c86accb.png align="left")

輸入一張graph，會經過兩個modules：node embedding module跟graph partition module，node embedding module在本篇透過GCN（local資訊）或Personalized PageRank（global資訊）來學出每個node的向量表示；graph partition module為Fully connected layer + softmax輸出每個node分別屬於K類的機率向量，當然，node embedding module跟graph partition module都是可以替換成不同方法的

這部分的loss為normalized cut，原則是want intra-group connections are denser than the inter-group ones, which mean lower numerator and higher denominator

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778082610/533e947a-a068-447c-a044-b2c442103460.png align="left")

● Imperceptible perturbations

Adversarial attack很重要的事情是，改變前改變後不能太明顯，在image上的例子如下圖

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778084923/a94eaa03-cfa7-4cd6-9323-ce619c8f1203.png align="left")

但在graph上沒辦法透過視覺化直觀地判斷這個改變明不明顯，所以如何定義graph上的imperceptible perturbations還沒有一個標準套路，有人是透過比較圖本身的資訊來判斷（例如node degree distribution），而這篇是透過比較原圖跟攻擊圖學出來的node embedding，兩張圖每個點之間的node embedding的KL散度加起來越小越好

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778086272/f81c6d81-8553-46cf-9464-f8edecff88d2.png align="left")

G為原圖；G hat為攻擊圖，ENC為encode的意思

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778088774/e4b090a1-480d-4fea-a6d6-5cbc9358ff48.png align="left")

● Constrained graph generation

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778091315/9df643e0-2202-4612-b3b3-b0e68bf51915.png align="left")

分成encoder part跟constrained generation part:，輸入為原graph，encoder part follow Variational Graph Auto-Encoder (VGAE)這篇paper的作法（目前graph generation上效果好的方法，參考下圖和reference \[1\], \[2\]），讓編碼出來的每個node的embedding服從標準高斯分佈

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778093287/2707e2bc-f002-4bdf-a4d3-8da6ff96721e.png align="left")

VGAE sample code

Encoder part的loss為：

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778096096/fac76321-49f7-472f-be09-680f860c8510.png align="left")

Constrained generation part:

大原則就是，幫graph裡面的node兩兩算一個相似度分數，如果分數高代表這兩個node之間越可能有edge，依照分數排名，for large-scale graph，就刪掉原本有edge的node pairs裡，排名最後delta位的edges；for small-scale graph，刪掉原本有edge的node pairs裡，排名最後delta / 2位的edges，並新增**原本沒有edge**的node pairs裡，排名前面delta / 2位的edges

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778098494/fca23dc6-6170-4f30-b8da-9dda1e5b830a.png align="left")

from 自製ppt

這邊的缺點就是實驗用的dataset都算很小的（如下圖），以及上述增刪edge的setting太naive，通常都不是optimal的解

![](https://cdn-images-1.medium.com/max/800/1*fCd-IDxsDekxg23wuH7lWw.png align="left")

### **Experiments**

![](https://cdn-images-1.medium.com/max/800/1*xd9QwQoTgiSqppw1GVT_Kw.png align="left")

from 自製ppt

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778105740/524d32d4-b477-4778-919c-f7e9343c789c.png align="left")

from 自製ppt

上圖中右下角的例子紅色部分的C+為target nodes們，藍色為communities

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778109917/d2be3739-f18c-4097-981c-3ebaaae60cec.png align="left")

from 自製ppt

上圖的實驗結果可以看出這攻擊方法效果並沒有很好，M1的指標最高也才13.72%，而且budget的增加也不能帶來多少效果提升，甚至綠色線在10的地方還掉下去

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778112601/3a3ed5ec-41a6-423a-a7fb-56bf68ee2adc.png align="left")

from 自製ppt

上圖的遷移實驗結果也可以看出這攻擊方法效果並沒有很好，M1的指標甚至還在Finance-medium被超過，其他的也沒跟baselines差多少

### **Rethink**

○ 沒有做各個loss的ablation study，不知道各loss的重要程度跟必要性

○ Actor-Critic style的方法，critic部分難收斂，paper中也提到CD部分跟攻擊部分是分開train，但這樣就沒有互相學習的效果了

○ Only report the values of M1 and M2 rather than the gain/changes of these measures，只回報數值感覺跟CD部分用什麼model比較有關係，而不是攻擊的成效

○ 只要點子新穎，即使提出的方法效果再差，還是可以上top conference

And that’s a wrap! Enjoy. 🎆

👏

### Reference

\[1\] [https://zhuanlan.zhihu.com/p/78340397](https://zhuanlan.zhihu.com/p/78340397)

\[2\] [https://jjzhou012.github.io/blog/2020/01/19/GNN-Variational-graph-auto-encoders.html](https://jjzhou012.github.io/blog/2020/01/19/GNN-Variational-graph-auto-encoders.html)

\[3\] 自己 & 和指導教授討論過的理解