---
title: 提升程式設計師的面試力 - Ch 11 - 測試
date: 2022-07-13 15:00:38
tags:
  - 提升程式設計師的面試力
  - Test
  - 讀書會報告
categories:
  - 讀書會報告
cover: https://picsum.photos/id/137/800/600
---

# 提升程式設計師的面試力 - Ch 11 - 測試

面試所遇到的與測試相關的問題大致上分為四大類


1. 測試真實物體
2. 測試軟體
3. 為一個功能撰寫測試程式碼
4. 解決已知問題

## 面試官想看到什麼

### 整體的理解程度

主要在測驗你是不是一個真正了解軟體的人，能不能適當的排出測試案例的優先順序。

### 知道如何化零為整

主要在測驗你是不是真的理解軟體如何運作，還有軟體是否適合其所在的生態圈。

### 組織能力

主要在測驗你在處理問題的方式是否是有條理的。

### 實際性

主要在測驗你是否可以實際制定合理的測試計畫。

## 測試真實的物體

題目：如何測試一個迴紋針

1. 誰將使用它？為什麼？
    - 使用該產品的人是誰？如何使用？
2. 使用情境是什麼？
    - 盡可能列出使用的情境
3. 使用限制是什麼？
    - 如標題，就是探討關於使用方面的限制
4. 壓力、故障條件是什麼？
    - 探討故障條件是否是該產品可以接受的
5. 你會怎麼做測試？
    - 測試方面的細節

## 測試軟體

軟體測試跟測試實際的物體很類似，不過會更著重測試的細節

而軟體測試有兩個核心的方面：

- 手動測試和自動測試
- 黑箱測試與白箱測試

底下描述一下測試軟體的脈絡

0. 我們要做的是黑箱測試還是白箱測試？
    - 可以詢問面試官他想做的是黑箱或是白箱，這樣在你之後步驟地回答方向就比較好掌握關於考官的偏好
2. 誰將使用它？為什麼？
3. 使用情境是什麼？
4. 使用限制是什麼？
5. 壓力、故障條件是什麼？
7. 你會怎麼做測試？
    - 這部分就可以針對黑箱或是白箱去擬定測試的細節，其中還可以提及是不是有部分的測試可以自動化，而關於自動化的優缺點也可以提及

## 測試一個函式

舉例來說，如果考官出了一個題目，請你測試一個排序的 function，那麼我們可以照下面的步驟去進行

### 1. 定義測試案例

可以分為底下幾種狀況去考慮
1. 正常狀況
2. 極端狀況
3. null 與「不合法的輸入」
4. 奇怪的輸入

### 2. 定義期望結果

各種測試案例下期望獲得的回傳結果或是造成的效果

### 3. 撰寫測試程式碼

## 除錯

還有一類的面試問題是在考驗你如何除錯或排除現有問題，而遇到這樣的問題的時候，可以依照以下的脈絡去回答

1. 瞭解場景
    - 盡可能了解造成問題的當下發生了什麼事情，如何復刻問題
2. 分解問題
    - 將流程分割為可測試的單元，試圖去找出問題發生的細節
3. 建立具體的、可行的測試
    - 對於問題建立測試的程式碼

## 題目

### 11.1

請找出以下程式碼中的錯誤
```cpp=
unsigned int i
for(i = 100; i >= 0; --i)
    printf("%d\n", i);
```

#### ans

這段程式碼中存在兩個錯誤

第一個是因為我們指定了 ```unsigned int```
所以在 ```for``` 裡面的條件設定應該改為 ```i > 0```

第二個是使用 ```%u``` 代替 ```%d```，因為我們要印出的是不帶正負號的整數([```%u``` 與 ```%d``` 的差異](https://www.796t.com/content/1545497120.html))

因此，正確程式碼修正為

```cpp=
unsigned int i
for(i = 100; i > 0; --i)
    printf("%u\n", i);
```

這樣我們就可以順利輸出 100 到 1 的數字
