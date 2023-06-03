---
title: "基礎生醫影像分割 — CT scan"
datePublished: Mon Mar 23 2020 09:41:27 GMT+0000 (Coordinated Universal Time)
cuid: clifooqx804mcyhnv7lae4ts1
slug: e59fbae7a48ee7949fe986abe5bdb1e5838fe58886e589b2-ct-scan-c930d5a85ac1
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685777982729/1456f5d9-615b-42de-8609-f14019370ad9.png

---

作業要求pdf: [https://drive.google.com/file/d/1LL2d6\_oJCJU4qfQk33j9HtD3U0FLKbsG/view?usp=sharing](https://drive.google.com/file/d/1LL2d6_oJCJU4qfQk33j9HtD3U0FLKbsG/view?usp=sharing)

code: [https://gitlab.com/warren30815/nthu-computational-methods-for-biomedical-image-analysis/-/tree/master/lab1](https://gitlab.com/warren30815/nthu-computational-methods-for-biomedical-image-analysis/-/tree/master/lab1)

本文主要會著重在背景知識介紹為主，code的部份了解背景知識就會了～

### 本文開始

> Introduction of CT

CT 是 X 光的 Tomogram斷層片，利用 X 光，加上電腦計算，所做出來的掃描圖，CT 對頭部的掃描，就好像切橘子一樣

我們把橘子當頭顱。從橘子外表看不到裡面有沒有爛掉，所以切片來檢查看看。一片一片切，一顆橘子切成 12 片。當然，首尾的橘子切片跟中間的切片，看起來會是不一樣的。我們一個切片一個切片來看看，這顆橘子有沒有地方爛掉或長蟲了。

從橘子的切片，是不是可以看到很多「裡面」的構造？是不是也好像看到了頭皮，頭骨，腦迴，跟很多內部的構造呢？如果這個橘子有地方爛掉了，仔細檢查這些切片應該檢查得出來。如果我們是用 3mm 的厚度切一片橘子，爛掉的地方是 7mm 那麼大，那麼可能有好幾片橘子切片都會看到爛的地方。當然如果爛掉的地方只有 1mm，那麼就看運氣了，有可能恰好切到，也有可能沒有切到，也就檢查不出來了。其實，CT 也是一樣，小於一公分的病變，有時 CT 也會恰好沒切到。

我們如果是從橘子的尾巴開始切的。如果看到爛的地方是比較後面的那幾片，那麼我們知道是橘子「頭」爛掉了。如果是最先的第 3 或是第 4 片就看到爛掉的地方，那麼我們就知道是橘子「尾」爛掉了。如果我們有三片看到爛掉的地方，那麼爛掉的範圍大概是 3mm 厚度的三倍，就是大概有 9mm 的大小爛掉了。

**如何拼湊成立體**

回到CT，因為掃描的順序如下圖，所以要從一張一張掃描得到的slice排序回原本的3d圖，就需要按照z軸排序，而座標軸的資訊，儲存在dicom格式的檔案中

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777965247/fc6e2725-8baa-492e-8f20-149e8d26888c.png)

CT 一般是以 OM line (Orbito-Meatus line），即眼眶中心點與耳道的連線，做第一張掃瞄

原文： [林啟光醫師](https://www.oocities.org/dr_ericlin/ct_read/01_introduction.html)

> DICOM (Digital Imaging and COmmunications in Medicine)

為一種廣泛使用的醫學影像格式，除了pixel data的紀錄外，也有許多資料欄位，如：

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777967633/b901ec8c-9f6c-4ed4-850e-f139181fdfa6.png)

而剛剛提到的座標，則在Image Position (Patient)那個欄位中，照著z軸排序即可:

slices.sort(key = lambda x: float(x.ImagePositionPatient\[2\]))

排序結果圖（前25 slices）:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777969289/b01c5a94-b417-4645-8afd-811ea1ab0051.png)

> Hounsfield units

X 光對各種物質的穿透力不一樣。鉛板可以完全阻隔 X 光。而空氣則毫無阻擋能力。這種穿透的容易度，跟物質的密度有關，我們稱鉛板的密度很高，而空氣的密度很低。如果要把這種密度「數值化」，我們就有個東西叫做「CT number」，Hounsfield units（HU）即為CT number的單位

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777971939/54d28b23-ca50-4d93-b355-dbeaa3b2a008.png)](https://www.oocities.org/dr_ericlin/ct_read/01_introduction.html)

一般的 CT 片，都把 window level (i.e. 中間色的值，決定顯示要白一點還是黑一點) 訂在大約 40，也就是大腦灰質的 CT number。這樣大腦的灰質就顯示成中間色，也就是灰色。而 CT number 比灰質大的，在 CT 片上就顯示成白色，密度越大就越白；CT number 比大腦灰質小的就顯示成黑色，密度越小就越黑

而比較正式的定義為：

The Hounsfield Units (HU) make up the grayscale in medical CT imaging. It is a scale from black to white of **4096** values (12 bit) and ranges from -1024 HU to 3071 HU (zero is also a value). It is defined by the following:

\-1024 HU is black and represents air (in the lungs). 0 HU represents water (since we consist mostly out of water, there is a large peak here). 3071 HU is white and represents the densest tissue in a human body, tooth enamel.

因此本次作業要先根據公式把pixel value轉成HU，然後normalize的話，可以這樣做:

if normalize:  
 image += 1024 # we want the minimum HU = 0  
 image = image / 4096.0

> Segment

本次作業只要求最基礎的segment方法，用一threshold來做二值化，小於threshold則黑；大於則白，我試了三種計算threshold的方法（skimage library均有實作），兩個local的一個global的，local的意思是每個pixel (HU)會有自己的threshold（根據一給定數量的鄰居們計算統計量，如mean, median，來定為該pixel的threshold），所以得出的threshold value是二維的，global得出的threshold value則是一個數字而已

Segment with local mean threshold結果（block\_size = 21）:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777974198/35cf044d-877b-4b90-9b7e-d9a1322a8d25.png)

紅線為將所有pixel’s threshold平均得到的value

Segment with otsu threshold結果（global）:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777976273/99f84fe3-49f5-4a73-85b9-f5aeb6526553.png)

紅線為otsu threshold value

> 3D viewer

用[這個軟體](https://fiji.sc/)可輕鬆畫出3D圖，[影片demo](https://www.youtube.com/watch?v=2N7i_gGYGLQ)

畫圖時不要從第一張segmented後的圖開始畫，不然會被全白的圖擋住，如：

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777978362/5663804d-5a98-4f7d-8120-6d4d7c5d9bab.jpeg)

可以從中間張數開始畫，就可以看到裡面了:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777980081/ec21c3db-7fb2-4c2c-ac04-34a1f7ec390d.png)

補充: 如果要精確畫出肺的部分，可以用Kmeans等方法去分出foreground (soft tissue / bone) and background (lung/air)部分，如[這個jupyter notebook的**Segmentation**章節](https://www.raddq.com/dicom-processing-segmentation-visualization-in-python/)

And that’s a wrap! Enjoy. 🎆

👏