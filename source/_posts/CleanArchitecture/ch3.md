---
title: 整潔架構 - ch3 範式概述
date: 2022-08-24 20:30:17
tags:
- Clean Architecture
- 讀書會報告
categories:
- 讀書會報告
cover: https://picsum.photos/id/137/800/600
---

# 整潔架構 - ch3 範式概述

這一章會概述這個章節會提到的三種程式設計範式(programming paradigms)

> 範式([paradigms](https://zh.wikipedia.org/zh-tw/范式))是指程式設計的方式，與語言無關。
> 範式告訴我們有哪些程式設計結構可以使用，以及何時該使用它們


- ch4. 結構化程式設計(structured programming)
- ch5. 物件導向程式設計(object-orient programming)
- ch6. 函數程式設計(functional programming)

## 結構化程式設計

第一個被採納的範式（並不是第一個被發明）是由 **Edsger Wybe Dijkstra** 於 1968 年發表的**結構化程式設計**，他證明了使用無限制地跳耀(goto 語句)對於程式的結構是有害的。比起這樣的跳耀他更推薦使用 ``` if/then/else ``` 和 ``` do/while/until ``` 來建構程式。

我們可以將結構化程式設計範式總結：**結構化程式設計在直接的控制移轉上加上規範**

### 關於 goto 語句

可以參考以下兩篇文章來了解

- [你所不知道的 C 語言: goto 和流程控制篇](https://hackmd.io/@sysprog/c-control-flow?type=view)
- [使用流程控制：GOTO](https://ithelp.ithome.com.tw/articles/10009329)

## 物件導向程式設計

到二個被採用的範式，實際上比第一個早了兩年，是由 **Ole Johan Dahl** 及 **Kristen Nygaard** 在 1966 年察覺到的。

我們可以將物件導向程式設計範式總結：**物件導向程式在間接控制移轉上加上規範**

## 函數程式設計

最近才開始被採納的第三種範式其實是第一個被發明的範式，函數程式設計是 **Alonzo Church** 工作的直接結果。

我們可以將函數程式設計範式總結：**函數程式設計在賦值上加上規範**

## 沈思

請特別注意到，這三種範式並沒有新增什麼新的功能，反而是告訴程式設計師「你**不該**做什麼」。
這三種範式總共刪除的是 goto 語句函式指標和賦值。

## 總結

說明範式和架構有什麼關係？

1. 多型可以作為跨越結構邊界的機制
2. 函數程式設計對資料的位置和存取進行了規範
3. 結構化程式設計可當作模組的演算法基礎

這三點與架構的三大關注點（函式、元件分離及資料管理）的關係匪淺
