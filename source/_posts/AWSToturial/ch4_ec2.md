---
title: 大話 AWS 雲端架構 - Ch4 - EC2
date: 2021-08-18 15:00:38
tags:
- 大話 AWS 雲端架構
- AWS
- 讀書會報告
categories: 
- 讀書會報告
cover: https://picsum.photos/id/137/800/600
---

# 大話 AWS 雲端架構 - Ch4 - EC2

## 4.1 添購伺服器的流程

添購實體 Server 的流程

```
|  軟體市場
|  作業系統 
|  硬體規格 
|  系統調校＋軟體安裝 
|  硬碟擴充 
|  標記資產
|  防火牆建立 
|  連線金鑰確認
∇
```

## 4.2 AWS 的 EC2 虛擬主機服務

在 AWS 的雲端服務中有一款 EC2 服務，能夠動態租用虛擬主機(Instance)，流程上也跟購買實體主機的概念大致相同

```
|  軟體市場 ◄---------------► AMI Marketplace (Amazon 系統映像市集)
|  作業系統 ◄---------------► AMI (Amazon 系統映像)
|  硬體規格 ◄---------------► Instance (虛擬主機)
|  系統調校＋軟體安裝 ◄-------► Userdata (使用者資料)
|  硬碟擴充 ◄---------------► EBS ()
|  標記資產 ◄---------------► Tag (標籤)
|  防火牆建立 ◄--------------► Security Group (安全群組)
|  連線金鑰確認 ◄------------► Key Pair (金鑰對)
∇
```

## 4.3 在專案使用 EC2 時的應用考量

### 4.3.1 購買方案 - Purchase Option

- 機器的數量有限額（[20 個 instance](https://aws.amazon.com/tw/ec2/faqs/))
- 付費方案分為幾種
    - On Demand (隨需)
    - Reserved (保留)
    - Schedule Reserved (時程保留)
    - Spot (競標)
    - Delicate (專用主機)



| 付費方案                     | 應用場境 |
| -------------------------- | -------- |
| On Demand (隨需)            | 適用於臨時需求且機器需要穩定表現，用多少、付多少、效能穩定 |
| Reserved (保留)             | 適用於需求明確，且確定是長期需求，可低價購買長期保留機器 |
| Schedule Reserved (時程保留) | 適用於需求明確，但僅有特定時間使用 |
| Spot (競標)                 | 適用於運算力高的需求，且運算任務可被中斷，但之後可接續進行（會依照競標高價者，取得使用權）|
| Delicate (專用主機)          | 適用於特殊軟體使用場景，會依照主機規格，進行授權使用|

### 4.3.2 虛擬映像檔 Amazon Machine Image

挑選完付費方案之後，接下來我們要挑選作業系統，也就是挑選 AMI

- AMI 屬於 Regin 級別的服務，不能跨區使用
- 如果想要跨區使用，可以製作快照(snapshot)，再透過 snapshot 複製出一個新的 AMI

### 4.3.3 機器 Instance

接下來要選擇硬體規格， EC2 的 instance 有許多不同的規格，例如下表



| 機器規格   | 虛擬處理器 | 記憶體 | 網路效能 |
| --------- | -------- | ----- | ------ |
| t2.micro  | 1        | 1     | 低     |
| t2.small  | 1        | 2     | 中低    |
| t2.medium | 2        | 4     | 中低    |

底下解析一下規格的英文數字組合所代表的意義


```

t2.micro
||    |
||    ∇
||   micro:規格大小
||
|∇
|2:虛擬化技術版本
|
∇
t:應用場景
```

應用場景的部分還有其他許多代號，每個代號代表不同的意義，如下表



| 常見代號 | 應用場景     |
| ------- | ---------- |
| A       | 一般用途     |
| T       | 一般用途     |
| M       | 一般用途     |
| C       | CPU 運算優化 |
| R       | 記憶體優化   |
| X       | 記憶體優化   |
| Z       | 記憶體優化   |
| U       | 記憶體優化   |
| P       | GPU 運算使用 |
| Inf     | GPU 運算使用 |
| G       | GPU 運算使用 |
| F       | GPU 運算使用 |
| I       | 儲存優化     |
| D       | 儲存優化     |
| H       | 儲存優化     |

### 4.3.4 網路與內部環境配置

有了有了主機與作業系統之後，難免會需要對作業系統做一些軟體安裝及設定，而這些動作可以透過 Userdata 來完成， Userdata 可以讓你使用你編寫的腳本，在 Instance 建立的時候 AWS 會主動幫你執行並完成你想要的環境建置

### 4.3.5 硬碟 - Elastic Block Store and Instance Store

EC2 為我們提供了兩種硬碟方案

1. EBS Volume
    - 與 Instance 位於不同的伺服器上
    - 速度較慢
    - 不愧因為 Instance 關機刪除資料
3. Instance store
    - 與 Instance 位於相同同的伺服器上
    - 速度較快
    - 關機後資料會刪除

### 4.3.6 EBS 的種類

如書本 p.62 的圖或者[官網表格](https://aws.amazon.com/tw/ebs/features/)

### 4.3.7 Snapshot 快照

EBS Volume 提供了備份相關的功能，叫做 Snapshot ，另外還推出了一個 Lifecycle Manager ，協助我們定期做資料備份

我們可以用 Snapshot 建立一個 AMI 出來，其場景大致上有兩大類

1. 把現有的 Instance 的內容與軟體做一個封裝，未來可用此封裝的 AMI 新建 Instance
2. 如果我們想對機器做轉移，就可以透過 Snapshot 的方式，以創造建立的方式來進行

### 4.3.8 防火牆 - Security Group (安全群組)

EC2 採用了外掛式的防火牆，又稱 Security Group (安全群組) ，我們可以自己設定 Inbound rule 及 Outbound rule ，在沒有調整的狀況下， Inbound rule 只會預設開啟提供給 SSH 用的 22 Port

如果我們想要在 EC2 Instance 之間連線做存取，我們也可以在填寫 Security Group Id 來允許指定的 Security Group 存取

### 4.3.9 網路卡 Elastic Network Interface (ENI)

EC2 的 Instance 也會幫主機配置網路卡，每一台 EC2 Instance 都至少有一張 ENI

預設的情況下，Public IP 在重新開機後會改變，但如果想要固定 IP ，可以使用 Elastic IP (彈性 IP)功能，把彈性 IP 綁定在 Instance 之後， Public IP 就不會異動了

### 4.3.10 連線金鑰 - KeyPair 金鑰對

在管理 Instance 的議題上， AWS 提供了兩種思維

1. 在 Instance 裡面放入公鑰，然後使用私鑰連入 Instance
2. 不在 Instance 裡面埋入金鑰

在 AWS 雲服務理論中，關於不放金鑰至 Instance 裡面的管理方法有兩種論點：

1. 希望我們盡可能不要連回 Instance 做操作，部署與應用啟動都透過 Userdata 來完成，後續如果 Instance 異常了，直接按先前的設定開一台或多台新的 Instance
2. 書上說有兩種可是他只寫了一種＠＠？

### 4.3.11 監控

Instance 啟動後，為了要確認 Instance 是否正常運作， AWS 會每五分鐘對 Instance 進行一次監控，如果連續偵測到三次異常數據，則會寄信給管理員

但如果覺得五分鐘太長也可以經過設定將時間縮短，可以得到比較即時的回覆

