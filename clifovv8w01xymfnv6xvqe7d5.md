---
title: "Key Opinion Leaders in Recommendation Systems: Opinion Elicitation and Diffusion"
datePublished: Thu Feb 20 2020 00:58:53 GMT+0000 (Coordinated Universal Time)
cuid: clifovv8w01xymfnv6xvqe7d5
slug: key-opinion-leaders-in-recommendation-systems-opinion-elicitation-and-da8b741a2b30
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685778316065/43b839ed-f236-4084-a332-9ce1ec4fb82e.jpeg
tags: neural-networks, graph, recommender-systems, kol

---

Google presentation: [https://docs.google.com/presentation/d/1FLrmmv2Gw23gym-Do3XSFd66wIAZqeXW1W4cMa25Yc8/edit?usp=sharing](https://docs.google.com/presentation/d/1FLrmmv2Gw23gym-Do3XSFd66wIAZqeXW1W4cMa25Yc8/edit?usp=sharing)

***The power of Key Opinion Leaders (KOLs)***

> A [Mediakix](https://mediakix.com/) survey conducted at the end of 2018 found that 49% of consumers depend on KOLs’ recommendations for their purchase decisions.

With important positions in the community (e.g. have a large number of followers), KOLs’ opinions can diffuse to the community and further impact what items we buy, what media we consume, and how we interact with online platforms.

😵 ***Problems***

Despite the importance of investigating the influence of KOLs in recommendation systems, however, it is a non-trivial task due to two major challenges:

1. **Elicitation**: Compared to regular users, KOLs tend to express their opinions on items explicitly (e.g., review, rating or tagging) rather than leave implicit feedback (e.g., views, clicks or purchases). More important, such explicit interactions are inherently multi-relational. On the other hand, the opinions of KOLs could have distinct meanings (e.g., the tag “fantastic” and tag “terrible” are semantically different). ***So how can we extract the elite opinions of KOLs from such multi-relational data?***
    
2. **Diffusion**: For example, users tend to purchase makeup products with the recommendation of Beauty-KOLs they are following. Meanwhile, previous research \[2, 3, 4\] has shown that user preferences on items could diffuse through high-order connectivity (e.g., in Figure 1, the latent preference of user A can diffuse via the transitive path A → q → B → w, to items that he/she hasn’t interacted with). Therefore, the influence of KOLs will also be propagated to those non-direct followers in the community. ***So how can we model this elite opinion diffusion process for improving recommendations?***
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778256983/45237e2a-74a3-4b4f-afec-eeb493777ed6.png align="left")

💡 ***Solutions***

GoRec: a novel end-to-end **G**raph-based neural model to incorporate the influence of K**O**Ls for **Rec**ommendation

1. **Elicitation**: A translation-based embedding method to elicit the opinions of KOLs.
    
2. **Diffusion**: Multiple Graph Neural Network (GNN) layers to model the diffusion process of elite opinions.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778259391/5d2a1b10-8b03-4455-b15b-1bbd6cf77833.jpeg align="left")

> Before scrutinizing this paper’s solution, let’s define the KOLs in real-world datasets and perform some initial analyses to confirm some hypotheses.

In two real-world datasets including Goodreads (a book-sharing community) and Epinions (an e-commerce review-sharing platform). We ranked all the accounts based on their numbers of followers and treat the top accounts as KOLs. After the definition of KOLs, we can start to confirm these hypotheses including:

▶️ **A small number of key opinion leaders (KOLs) can provide sufficient coverage.** In Goodreads, we found that while considering only the top 500 KOLs, there are more than 95% of the users follow at least one of these KOLs. Consequently, *a small number of KOLs can provide sufficient coverage.* Figure 2(a) showed more details.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778261517/6f94dd41-b885-4934-a872-a938813f613b.png align="left")

Figure 2a. Coverage: The percentage of users following at least one of the top (key) opinion leaders. More than 95% of users follow at least one of the top 500 accounts.

▶️ **Users are shifted by the KOLs they are following.**

We represented each user with a simple binary vector over all books, in which a “1” indicates that the user has left implicit feedback on this book, and with five different binary vectors (1 to 5) for each KOL to represent the books’ ratings. We concluded that the explicit opinions of KOLs could directly influence what their followers consume. Figure 2(b) showed more details.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778263941/81e1b9a1-6557-49f3-b59e-eb438adaa9bc.png align="left")

Figure 2b. Books read by users are more similar to books with higher ratings from key opinion leaders they are following.

▶️ **Compared to ordinary users, KOLs tend to express opinions on items explicitly.**

In Figure 2(c), we can see that ordinary users and KOLs leave implicit feedback on a similar number of books, indicating that both are active in their use of Goodreads. Nevertheless, KOLs are more engaged in explicitly sharing their opinions on books.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778266239/c8ede872-0e5a-4434-99e9-a568457cd9c5.png align="left")

Figure 2c. While leaving a similar number of implicit feedback, key opinion leaders prefer to show their opinions on items via explicit interactions (reviews, ratings, self-defined tags).

> With these observations in mind, we can concentrate on solving the two challenges of GoRec.

***Problem Setting and Notation***

In this work, we aimed to provide Top-K recommendation from a candidate set of M items **I** = {i1, i2,…, iM} to a set of N users **U** = {u1, u2,…, uN }.

ℹ️ **User-item Interaction Graph:** a bipartite graph G = (V, W) in which the set of nodes V consists of all the users and items. The edge (u, i) ∈ W denotes that user u has implicit feedback on item i.

✨ **Elite Opinion Graph:** We used **L** = {l1, l2,…, lp } to represent the set  
of key opinion leaders (KOLs) and **O** = {o1, o2,…, oQ} to represent Q different types of explicit opinions. It can produce many opinion triplets (l, o, i) representing kol *l* left opinion *o* on item *i, a*nd by these triplets, we can construct a directed elite opinion bipartite graph **Go.**

💫 **User-KOL Following: F**u ⊂ **L** to represent the set of KOLs followed by user u ∈ **U**. And we let **U** ∩ **L** = ∅ (no overlapping between ordinary users and KOLs).

🏃 Firstly, we start by eliciting the opinions from KOLs toward improving  
the quality of recommendation

***Translation-based Opinion Elicitation***

As analogous to the data structure of the *knowledge graph*, the resulting  
elite opinion graph **G**o consists of many valid opinion triplets. For  
example, a triplet (*l₁, Review: wizard, Harry Potter*) denotes that  
KOL *l*₁ mentions the word *wizard* in a review for item *Harry Potter*. We can also construct these opinion triplets based on ratings or tags provided by KOLs, and get triplets like (*l1, Rate: 5, Harry Potter*) or (*l1, Tag: fiction, Harry Potter*).

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778268368/0bc8e881-8a06-4cca-862a-85c5a5057bd6.jpeg align="left")](https://yashuseth.blog/2019/10/08/introduction-question-answering-knowledge-graphs-kgqa/)

Knowledge Graph: a semantic network obtained by connecting heterogeneous information, providing the ability to analyze problems from a “relationship” perspective.

Our goal is to generate **“effective embedding”** for both items and KOLs in a continuous vector space while preserving the multi-relations (opinions) between them. We will list 3 features of **G**o followed by the corresponding design we propose in the opinion elicitation process:

☝️ **Diverse relationship:** Opinions come with a distinct meanings,  
e.g., tags “fantastic” and “terrible” are semantically different.

***Translation from KOL to Item.*** Adopting a similar idea in multi-relational  
graph embedding \[6, 7, 8, 9\]. Given a valid opinion triplet *(k, o, i)*, we want to ensure that the embedding of the item is close to the embedding of KOL k plus the embedding of opinion o. Let *s(k, o, i)* denote the scoring function for the translation operation, with which a larger value means better translation.

To ensure the graph embedding effectivity, our objective is to maximize the translation score for all the valid triplets while minimizing that for the invalid triplets (k wouldn’t attach opinion o on i′). As the below formulation:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778271208/7946fa6a-5a0e-45a1-ae0b-9b560864c835.png align="left")

in which \[·\]+ ≜ max(0, ·), and γ is a hyper-parameter that denotes the threshold the model used to separate the valid triplets and invalid triplets.

Later we will describe how to calculate scores.

**✌ ️Many-to-Many relations:** On one hand, KOLs and items can be multi-relational (e.g. A KOL rated 5 and left a comment on a book). On the other hand, KOLs can endow opinions with their attitudes certain (e.g. each KOL has his/her criteria for tagging a book with “BestOf2019”).

***Dynamic Mapping Matrix.*** To handle the Many-to-Many relations, a common strategy is to project KOLs and items to an opinion-specific space before the translation operation. We adopt a dynamic mapping matrix \[7\] which is determined by both the opinion and the KOL (or item). Each kol, item, and opinion is represented by two vectors. One vector acts as its embedding, while the other vector is used to transfer KOL (or item) into opinion-specific space (as in Figure 3). Given a triple *(k, o, i)*, we will initialize dense vectors *kᵉ, kᵗ, oᵉ, o****ᵗ***\*, iᵉ, iᵗ\*, with superscript e meaning embedding and t meaning transfer respectively.

![](https://cdn-images-1.medium.com/max/800/1*2fq4BbgU_KnK4KG-fhSnRg.png align="left")

Remember that we aim to calculate the embedding translation score for the triples, we need to transfer those embeddings to the same space. Accordingly, we construct the mapping matrices as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778275397/d19bba71-8edd-4d3f-a7e8-e21435b12c1e.png align="left")

Mapping matrix for transferring kᵉ to the space of o

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778276615/83a8b633-7bce-4835-9b2c-d565b3fc3111.png align="left")

Mapping matrix for transferring iᵉ to the space of o

in which **I** denote the identity matrix for initializing the mapping matrix. Thus we get the projected representation of *k* and *i*:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778278948/b732827d-02a0-4aaf-b4b1-348b5b07c4ad.png align="left")

We can utilize the projected *k* and *i* to evaluate the translation distance for triple *(k, o, i)*. Larger *s(k, o, i)* means *k* and *i* are close to each other with translation *o*, i.e, it is more likely that *k* attaches opinion *o* to *i*:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778280470/56150987-a52d-4507-ba48-e402a5f47e3f.png align="left")

We use L2-norm to calculate the distance empirically.

**👌 Preference Signals:** KOLs have preferences on the items they would interact with, e.g., a romantic book lover may seldom leave any feedback on horror novels.

***Personalized Ranking Model.*** A typical assumption is that the items with feedback from the user are preferred over those without. We also can utilize these preference signals. Following the basic idea in [*matrix factorization*](https://en.wikipedia.org/wiki/Matrix_factorization_%28recommender_systems%29), we use the multiplication between *kᵉ* and *iᵉ*, that is *p(k, i)* = *kᵉ*ᵀ *iᵉ*, to capture the preference of *k* on *i*. Positive pair *(k, i)* representing *k* has left feedback on *i* and negative pair *(k, i′)* meaning *k* has not left feedback on *i*. The objective function to model these preference signals is:

![](https://cdn-images-1.medium.com/max/800/1*IyNUCQ1g-tM05U3dW9k5mw.png align="left")

Use Bayesian Personalized Ranking (BPR) \[10\] to maximize the difference of preference scores between the positive and negative pair, δ(·) denoting the Sigmoid function, and S is the training dataset.

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778284938/8c037b87-1870-47af-9434-ee356e44b152.png align="left")](https://zwindr.blogspot.com/2017/08/ml-matrix-factorization.html)

Example of matrix factorization

👏 Now we finish the opinion elicitation, leading to the following loss function:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778286896/7cb8f2a7-309f-4569-9c58-c876fc71848f.png align="left")

β is used to adjust the weight of pairwise loss in capturing the preference signals.

By minimizing this joint loss, we will get the set of embeddings **K***ᵉ* for KOLs and **I***ᵉ* for items, which inherit both the explicit information and preference signals in the elite opinion graph **G**o.

🏃 Finally, let’s highlight how to enrich the initial user/item embeddings with elite opinions and how to model the elite opinion diffusion process with graph neural networks.

***Fusing Layer (Users)***

Each user is associated with an embedding eᵤ to represent the initial interest, which can be trained from his/her one-hot index with a fully-connected dense layer. Moreover, we know that a user has different attention to each KOLs. We can model the dynamic (personalized) linkage between users and KOLs. Given that **F**u is the set of KOLs that u is following, our objective function of elite opinions influence to user u is as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778290094/bbeaf5b6-6ea6-467e-ab43-fd616f75e317.png align="left")

α is a trainable attentive weight

And α (the weight of KOL p’s influence on user u) can be calculated as:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778291587/53e285eb-56a9-4df4-80f6-df5779d07365.png align="left")

Here | | represents the concatenation operation. W and b is the weight matrix and bias for the attention layer. z is a transformation vector to get embeddings (then all ReLu(‧) in the same space).

👏 By fusing nᵤ (elite opinions influence to user u) with the initial embedding of u, we can obtain the cornerstone for the opinion diffusion:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778293021/f21036e4-120b-4565-98b3-78e46d15d9bc.png align="left")

W is a transformation matrix to get embedding (making the concatenated vectors into the same space)

***Fusing Layer (Items)***

Similarly, each item will start with a trainable dense representation eᵢ, which is associated with its index. **Since the KOLs can influence how the whole community views an item**, we want to complement eᵢ with the KOL-defined features iᵉ. Thus we adopt a similar fusion operation to generate the enriched representation of item i:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778295999/c4f89113-c518-4c5e-84e7-79f27a3a0748.png align="left")

W is a transformation matrix and iᵉis the embedding gained from Dynamic Mapping Matrix for item i.

***Opinion Diffusion with GNNs***

As Figure 1. showed, user preferences on items could diffuse through high-order connectivity, thus the elite opinions from KOLs will also be propagated to those non-direct followers in the community. In this paper, we propose  
to model this opinion diffusion process by virtue of Graph neural networks (GNNs).

The core idea of GNNs is that each layer learns the node embeddings by aggregating the features of neighbors. On the user-item bipartile graph, given the sets of neighbors Nu and Ni which are directly connected with u and i correspondingly, we formulate the message passing on the edge (u, i) from i to u as:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778298089/64b35bbf-18fd-4494-89d4-1677dfc24144.png align="left")

xᵢ is the representation of i with influence from KOLs and Wu(0) denotes a trainable transforming matrix for users at layer 0.

(|Nu | |Ni |)^-1/2 is a normalization constant between u and i. The reason for dividing Nu is to average all “item to current user” messages, and Ni is to calculate item message propagation to each adjacent user (e.g. item A has 4 messages and 2 neighbors, and each neighbor gets 4/2 messages, not 4).

Then we sum up all the messages passed to u to generate its representation xᵤ(1):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778299312/5273dc14-0873-4c22-b114-badbe30b53ae.png align="left")

τ is the activation function and we choose ReLu in this work empirically

Similarly, we can generate the representation of item i at this layer with:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778302444/55a93c75-a446-4ea7-a1f1-706fee457cba.png align="left")

After generating the representation of u and i from the first GNN layer, we can further capture the high-order diffusion by stacking multiple GNN layers. Specifically, at the Lth layer, we will have:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778305127/67765fc6-feda-4a45-9e31-7f78a6e52224.png align="left")

xᵤ(L) at layer L inherits embeddings of users and items from previous layers.

Lastly, we can capture the diffusion of opinions in multiple-order user-item connectivity with GNNs.

> Our final step is to infer the user u’s preference for all the items. 😍

We use a fully-connected layer to decode the output from the GNN layers. That is, for u, we will decode xᵤ(L) to reconstruct his/her feedback vector yᵤ:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778306455/bad893c1-c253-451e-aa2b-c4e054a25798.png align="left")

V and b′ are the weight matrix and bias term correspondingly. And δ(·) represents the Sigmoid function.

The objective is to minimize reconstruction loss:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778307921/7e7f0ed7-fe85-4dff-8850-b40b4bd3e4c2.png align="left")

Su is a binary masking vector. Since the feedback usually is extremely sparse, we only consider all the 1s in yᵤ while calculating the loss.

🎊 Finally, we can write the objective function of our final model (GoRec):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778311332/d32f841c-3c05-4f8c-bce7-75edac969154.png align="left")

Thus we reach our GoRec model (in Figure 4 ) which combines both  
tasks end-to-end with a hyper-parameter λ to balance the tasks.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685778313809/7f842797-5b5a-49d6-9203-390951fbbffa.png align="left")

***Experiment results***

Through experiments on Goodreads and Epinions, the proposed model outperforms state-of-the-art approaches by 10.75% and 9.28% on average in Top-K item recommendations.

And that’s a wrap! Enjoy. 🎆

**References:**

1. Jianling Wang, Kaize Ding, Ziwei Zhu, Yin Zhang, and James Caverlee. 2020. Key Opinion Leaders in Recommendation Systems: Opinion Elicitation and Diffusion. In WSDM.
    
2. Yun He, Haochen Chen, Ziwei Zhu, and James Caverlee. 2018. Pseudo-Implicit Feedback for Alleviating Data Sparsity in Top-K Recommendation. In ICDM.
    
3. Xiang Wang, Xiangnan He, Meng Wang, Fuli Feng, and Tat-Seng Chua. 2019. Neural Graph Collaborative Filtering. In SIGIR.
    
4. Jheng-Hong Yang, Chih-Ming Chen, Chuan-Ju Wang, and Ming-Feng Tsai. 2018. HOP-rec: high-order proximity for implicit recommendation. In RecSys.
    
5. [Introduction to Question Answering over Knowledge Graphs](https://yashuseth.blog/2019/10/08/introduction-question-answering-knowledge-graphs-kgqa/)
    
6. Antoine Bordes, Nicolas Usunier, Alberto Garcia-Duran, Jason Weston, and Oksana Yakhnenko. 2013. Translating embeddings for modeling multi-relational data. In NeurIPS.
    
7. Guoliang Ji, Shizhu He, Liheng Xu, Kang Liu, and Jun Zhao. 2015. Knowledge graph embedding via dynamic mapping matrix. In ACL.
    
8. Yankai Lin, Zhiyuan Liu, Maosong Sun, Yang Liu, and Xuan Zhu. 2015. Learning entity and relation embeddings for knowledge graph completion. In AAAI.
    
9. Zhen Wang, Jianwen Zhang, Jianlin Feng, and Zheng Chen. 2014. Knowledge graph embedding by translating on hyperplanes. In AAAI.
    
10. Steffen Rendle, Christoph Freudenthaler, Zeno Gantner, and Lars Schmidt-Thieme. 2009. BPR: Bayesian personalized ranking from implicit feedback. In UAI.