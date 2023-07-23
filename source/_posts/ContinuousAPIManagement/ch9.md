---
title: 持續 API 管理 - ch9 API 園林
date: 2023-02-01 17:45:38
tags:
  - API
  - 讀書會報告
categories:
  - 讀書會報告
cover: https://picsum.photos/id/137/800/600
---

# 持續 API 管理 - ch9 API 園林

### API 園林的定義
API 園林就是一個組織發表的所有 API 。
園林內的 API 可能處於不同發展階段，也可能有不同的風格與實踐方法。

人們可能會用 API portfolio、API catalog、API surface area 來稱呼 API 園林。

## API 考古

### API 園林的定義
API 考古就是尋找整合處，研究他們為何會被做出來、怎麼做出來，用以了解複雜的 IT 系統歷史與結構

### proto-API
讓組件互動的「非」API 機制都可以稱作 proto-API （你可以把它想成尚未發展成真的 API，還停留在原型階段的意味）

### 總之
API 考古可以幫助你更了解 API 園林，即使組件大部分都還是 proto-API，他是了解過往整合需求的起點。

## 大規模的 API 管理
管理大規模的 API 就是在「實施某種園林規模設計規則」和「將個別 API 等級的設計自由度最大化」之間進行平衡。這種平衡型是經常出現在典型的複雜系統裡：「為了實現一制性和潛在的優化而將整合中心化」 vs. 「為了實現敏捷性與可發展性而去中心化」

- 整合中心化
    - 這種作法的主要驅動力是將功能的交付標準化，以便用優化且具有成本效益的方式提供那些功能。高水準的整合可以帶來更多優化的潛力，但也會影響系統的可變性與可發展性。
- 去中心化
    - 採取這種作法的主要動力是為了將功能的可訪問性標準化，使得功能可以用大量的、不斷發展的方式提供。去中心化的目標是為了改善鬆耦合，也可以讓園林更容易個別做變動，而不影響到其他部分。

### 平台原則
很多人在討論 API 及一般商業目標時都會提到「平台」，但商業上關於平台的想法，可能跟技術面會有所出入。商業上，平台提供一個基礎，將各方聚在一起，以便交換價值。

平台的吸引力通常會被兩個因素所影響

1. 平台接觸範圍有多大？
    - 也就是當我們加入了那個平台後，可以接觸多少用戶？
2. 平台的功能是什麼？
    - 如果我在平台上建立產品，平台會如何支援或是限制我產生價值？

在技術層面常見兩種平台

1. 網路 APP
    - 可以被所有人，以及支援基本網路標準的任何東西使用，例如瀏覽器
2. 原生 APP
    - 通常在中心化的 APP 商店裡下載，有可能只能在特定裝置被使用，商店擁有者有「決定可以安裝什麼」的權利


### 原則、協定與模式
精心設計的平台是圍繞著原則、協定與模式設計的，我們可以用 web 平台來說明這些概念，而 web 平台也同時具有驚人的穩定性與靈活性。

在過去的幾十年裡，web 的基礎結構沒什麼改變，但本身卻有很大的發展，其中的原因在於，web 平台沒有任何規則指定服務該如何實作及被使用，web 架構把重心放在「決定資訊該如何識別、交換與表示」的介面上，這使得他處理增長及變化的能力勝過其他任何的 IT 架構

#### 原則
原則是建立在這個平台中的骨幹中的基本概念，web 平台有一條原則是「用統一資源識別碼(URI)來識別資源」

#### 協定
協定定義了基於原則的具體互動機制，如 HTTP(s)、FTP、WebSocket...等等。

#### 模式
模式是在原則和協定所形成的解決方案空間中解決問題的共同實踐法，說明如何結合協定中的互動來實現 APP 的目標，例如 OAuth 機制。

### 將 API 園林當成語言園林

> 在本節中的「語言」一詞，指的是「與 API 之間的互動」，而不是程式語言的實作

- API 風格決定了基本的對話模式（例如：同步、非同步請求）以及主要的對話規範
- API 協定決定了基本語言機制，例如基於 HTTP 的 API 中 request header
- 在 API 協定中，通常有許多技術「子語言」為核心技術的擴充，例如原始 HTTP header 指定義了部分的欄位，API 可以根據他們的規範去選擇擴充的 header
- API 的某些層面可能是跨領域的，而且很容易在各種 API 中重複使用。（詳情可見後數的「詞彙」小節內容）

### API the APIs

「關於 API 的一切應該由 API 說出來」，可以為 API 提供狀態資訊，或是要求 API 文件也成為 API 的一部分或者透過 API 本身來管理 API 的資安層面，使 API 成為自助產品，提供資訊來幫助別人了解與使用它

## 了解園林

API 園林與其他產品園林並沒有什麼不同，我們的目標都是讓他可以輕鬆的發展。通常我們一定會在「優化單一目標」與「優化可變性」之間進行權衡。

## API 園林的八個 V

管理 API 園林是一個困難的工作，我們需要在「產品的速度與獨立性」與「長期的一致性與穩健性」之間取得平衡。

以下會介紹管理 API 園林的 8 個重要面向：**多樣性、詞彙、數量、速度、脆弱性、能見度、版本管理、波動性**

### 多樣性

API 園林必須保持多樣性，同時也要限制多樣性，避免 API 用戶端學習與太多不同風格的 API 互動。因此，管理 API 園林的多樣性是一種平衡行為，要避免大量無益的 API 風格，同時又要對於擴充多樣醒保持開放態度。重點在於，要避免在園林內建立那種讓你難以長期支援的東西。

### 詞彙

每一種 API 都是一種語言（如前述），他定義開發者如何與服務互動。
我們要盡量避免在詞彙上重新造輪子，最簡單的例子就是 HTTP 狀態碼，這種已經默默成為共識的東西。

採取復用這樣的語言有幾個好處：
1. 開發團隊不需要重新發明詞彙
2. 使用者也可以降低學習成本

如果真的要定義詞彙的話，以下有幾種管理方式
1. 表達與 API 的互動
    - XML 或 JSON 的 Schema
2. 當成表示法的基本元素
    - XML 的名稱空間
3. 共享的資料類型

有效管理詞彙的另一種可行方法就是使用業界標準，雖然不一定完美也不一定適合我們的 API ，但其中肯定有可以復用的基本元素。

### 數量

園林內的 API 數量會隨著時間增加，我們的目標是可以輕易處理自己的園林 API 數量規模。關於園林中的 API 數量，有的組織會盡量減少數量，有的組織的重點是讓數量問題更容易處理。

### 速度

更安全且快速地進行設計、發表、測試與變動，去應應市場的變化、激烈的競爭。在 API 園林中，速度在很大的程度上是藉由給予開發團隊更大的自由來提昇。

前面提到接受去中心化的概念、放棄整合是避免耦合以及耦合造成的速度下降的方法之一，這種作法的代價是，你必須要調整交付與營運方法來適應新園林，並確保整體的生態系統符合組織所需的標準與穩健性。

### 脆弱性

我們要把對外部的依賴視為脆弱的關係，並在這些接合點上讓服務保持韌性，雖然實作韌性有時候可能不是小事情，有些第三方的依賴項目非常重要，當他失效時做任何事情都無法挽回，但即便如此，我們的設計方向應該朝向防止服務崩潰當機、讓服務明確的回報狀況，以便後續修復。

另外值得注意的是，當 API 越多，API 也會成為攻擊的對象。

總之處理脆弱性的方法是，以安全(safely & securely)的方式管理 API。

### 能見度

相對小的規模中，大多數事情都是可見的，但在大型且分散的環境中，能見度是一個很難實現的目標。大規模能見度的典型模式往往結合兩個層面（以 web 為例）

1. 發表東西必須以一種可以被發現的方式進行
    - 例如 sitemap、schema.org
2. 搜尋東西比起僅僅只是發現目標更依賴背景
    - 例如 Google 的 page rank ，為搜尋結果的受歡迎程度來計算相關性

### 版本管理

簡單來說，使用[語意化版本系統](https://semver.org/lang/zh-TW/)。

### 波動性

服務可能變動、停止運作甚至消失，我們的目標是盡量將 APP 設計的有「防禦性」，讓他更有機會去適應執行環境的變化，在 API 園林中，應用程式的終極目標是：盡量穩健的營運，以及不要依靠其他組建的妥善性。

## 結論
