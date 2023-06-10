---
title: "Gpt3 — 耗資1200萬美元訓練的通用nlp模型的商業價值"
datePublished: Wed Sep 16 2020 02:12:45 GMT+0000 (Coordinated Universal Time)
cuid: clifong7g04lsyhnv6wg4dqv6
slug: gpt3
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685777922747/0c314690-4fa4-4473-b996-679b82aabd90.jpeg
tags: gpt-3

---

全文連結：[https://techcrunch.com/2020/08/28/what-does-gpt-3-mean-for-the-future-of-the-legal-profession/](https://techcrunch.com/2020/08/28/what-does-gpt-3-mean-for-the-future-of-the-legal-profession/)

> GPT-3, an one-fits-all NLP model without need of re-training

**What is GPT-3 ?**

GPT-3 是一款OpenAI發布的通用自然語言處理模型，只需要在 GPT-3 中的輸入框裡用一般使用的語言進行描述，就可以幫你生成出你想要的東西，不管是撰寫網頁code、文章、答題、統計、翻譯通通可以，其基礎原理是: 當你輸入一個或一段文字後，GPT-3 會這種字跟哪些字有高度相關，例如你輸入「fire」後，它會依照權重判斷「truck」和「alarm」會比「lucid」或「elvish」更常跟 fire 一起出現，然後一串一串接著下去自動產生文章。

**What difference between GPT-3 and other general NLP model (e.g. Bert)?**

能用更少的領域數據、**且不需經過**微調（**fine-tune）步驟去解決問題**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777919362/4bcb25a4-3b39-403e-a4b8-8c33a27ccd2c.jpeg align="left")

這裡的Zero-shot（不給例子）、One-shot（只給一個例子）、Few-shot（只給少數例子）experimental setting都是完全不需要fine-tune的

**Why the above mentioned is valuable?**

對於業界使用深度AI來說，標註資料量的多寡是個大門檻，在NLP領域中即使有Bert這種預訓練（pre-trained）模型的出現，但還是少不了一定量的 task-specific 標註數據去fine-tune模型，但 task-specific 的數據又有限，模型只能從少量的數據去學習如何調整，很可能造成過擬合（over-fitting），使模型產生偏頗（bias）。例如用AI來協助法律判決，如果AI看過的case不夠多，就會沒辦法做出合理的判決，而用來訓練AI的資料（例如法律文件、判決書）的收集和整理過程是很耗時間和金錢的。

因此，GPT-3靠著輾壓式的1750億參數、45TB訓練數據，只要用文字給出任務描述，GPT-3就會去理解任務並執行，讓業界能更專注在思考新奇的點子與用法，而不是90%的時間都在數據收集與清理（data clean）

### Reference

\[1\] [https://www.inside.com.tw/article/20684-A-college-student-used-GPT-3-to-write-fake-blog-posts-and-ended-up-at-the-top-of-Hacker-News](https://www.inside.com.tw/article/20684-A-college-student-used-GPT-3-to-write-fake-blog-posts-and-ended-up-at-the-top-of-Hacker-News)

\[2\] [https://buzzorange.com/techorange/2020/08/11/gpt-3-helps-coding-and-news-editing/](https://buzzorange.com/techorange/2020/08/11/gpt-3-helps-coding-and-news-editing/)

\[3\] [https://www.zhihu.com/question/398114261](https://www.zhihu.com/question/398114261)

And that’s a wrap! Enjoy. 🎆

👏