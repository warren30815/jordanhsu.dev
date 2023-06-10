---
title: "五分鐘學前端系統設計面試（一）- 總覽"
datePublished: Fri Feb 10 2023 15:51:28 GMT+0000 (Coordinated Universal Time)
cuid: clifom0nr04leyhnv2ox7bikx
slug: frontend-system-design-facebook-feed-1
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685777856188/7ca14a41-ecce-46b9-a09a-6f8ba87507f4.png

---

此為兩篇文章的重點整理：[https://www.greatfrontend.com/system-design](https://www.greatfrontend.com/system-design)、[https://www.greatfrontend.com/system-design/types-of-questions](https://www.greatfrontend.com/system-design/types-of-questions)

> 系列文第一篇文章為總覽性質，帶大家認識前端系統設計面試會注重哪些事情，鑑於大部分系統設計文章都是討論後端為主，本系列專注在前端的系統設計

### 前端 vs 後端的系統設計面試

先上一張簡單比較表

舉例來說，一個經典的面試問題是設計一個推特動態牆的應用，在前端&後端系統設計時，著重的討論點會有如下差異：

後端：成本估算、設計資料庫schemas、面對大流量時如何水平擴展、如何產生一個使用者的tweet…

前端：對一則tweet使用者可以做什麼操作（點讚轉發…）、如何實作動態牆的分頁（pagination）、使用者如何建立新的tweet…

本系列文中，我們將專注在前端的部分，前端的系統設計問題主要分為兩類  
***1\. 應用元件  
2\. UI元件***

### **應用元件**

由上述例子可以發現，前端不談如何設計一個分散式系統，而是著重在client端的實作細節 & 應用層面的架構，不同的應用程式有不同著重的探討點，以下列表常見的應用類型、日常生活中的例子以及設計該類型系統著重的點：

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777851602/423e0391-9905-442d-9d31-6d42acb52bf4.png align="left")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685777853510/45076ea2-a704-4a05-a15c-3aea5ff7f322.png align="left")

### **UI元件**

現代的前端開發中，可以直接使用現成的元件庫，如jQuery UI, Bootstrap, Material UI, Chakra UI, Element UI…，但作為前端工程師，能自己刻出UI元件也是需要的能力，一些簡單的元件如text, button, badge…，而複雜的如autocomplete, dropdown, modal…，大多數時候面試被問的是後者

對於設計此類複雜的UI元件，有幾個重點：  
\- 定義子元件（以image carousel來說，即圖片、分頁按鈕、縮圖…）  
\- 定義元件的外部接口（options/props）  
\- 定義元件內部狀態  
\- 子元件間的API  
\- 最佳化效能、無障礙使用、使用者體驗、資安….

### 結語

本篇文章中，我們介紹了前後端系統設計面試著重點的差異，以及前端系統設計面試中會需要考慮的問題

下一篇中我們會介紹系統性的回答這些問題的框架 — RADIO framework

And that’s a wrap! Enjoy. 🎆

👏