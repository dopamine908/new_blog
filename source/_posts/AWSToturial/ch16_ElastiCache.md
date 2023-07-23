---
title: 大話 AWS 雲端架構 - Ch16 - ElastiCache
date: 2021-11-17 15:00:38
tags:
- 大話 AWS 雲端架構
- AWS
- 讀書會報告
categories:
- 讀書會報告
cover: https://picsum.photos/id/137/800/600
---

# 大話 AWS 雲端架構 - Ch16 - ElastiCache

[ElasticCache](https://aws.amazon.com/tw/elasticache/) 簡單來說就是單獨管理快取的一種服務

## 16.3 兩種常見的暫存資料操作手法

### Lazy-Loading

[Lazy-Loading](https://zh.wikipedia.org/wiki/惰性載入) 又稱延遲載入，大意上是只在有人需要資料的時候才將資料做快取，這樣下次有人需要同筆資料的時候就可以快速的回覆資料給使用者


### Write-Through

簡單來說，就是當我們寫入資料的時候也寫一份到快取內，則使用者需要資料的時候就可以即時反應

### Lazy-Loading 與 Lazy-Loading 的比較


| Lazy-Loading | Write-Through |
| -------- | -------- |
| 用戶第一次存取資料的時候，<br>才寫入快取中     | 資料寫入資料庫的時候，<br>同時寫一份到快取中     |
|第一次提取時速度會比較慢|提取速度快|
|被動快取資料|主動快取資料|
|節省記憶體用量|記憶體用量大|
|第一次查詢資料的時候速度會較慢，<br>另外如果資料庫資料有更新而沒有順便更新快取的話，<br>使用者可能會取用到舊的資料|會快取很多沒有要用到的資料，<br>且寫入資料的時候會更花時間|

## 16.4 ElastiCache 的兩種底層引擎 - Memcached 與 Redis

### Redis 與 Memcached 的使用差異



|| Memcached | Redis |
| -------- | -------- | -------- |
| 毫秒等級延遲     | :white_check_mark:     | :white_check_mark:     |
| 資料分割     | :white_check_mark:     | :white_check_mark:     |
| 支援多種程式語言  | :white_check_mark:     | :white_check_mark:     |
| 進階資料結構     | :x:     | :white_check_mark:      |
| 多執行緒架構     | :white_check_mark:      | :x:     |
| 快照備份     | :x:     | :white_check_mark:      |
| 覆寫功能     | :x:     | :white_check_mark:      |
| 交易用途     | :x:     | :white_check_mark:      |
| 消息訂閱系統     | :x:     | :white_check_mark:      |
| 支持 Lua 腳本     | :x:     | :white_check_mark:      |
| 支援地理位置     | :x:     | :white_check_mark:      |

[Lua](https://zh.wikipedia.org/wiki/Lua)

## 16.5 Redis

Redis 隨著需求發顫，已經越來越像在記憶體上的資料庫了。於是 Redis on ElastiCache 的高備援方案，自然也就像是資料庫一樣。首先我們必須定義 Subnet Group (子網路群組)，未來 ElastiCache 會在裡面開啟資料庫的 Node (節點)，並且在節點裡面會用多個 Shard (分片) 來儲存資料，並把
Shard 分散在多個節點，避免節點失效後，資料也一併遺失。如果想要做效能調校，也可以透過 Parameter Group 完成

![](https://i.imgur.com/Bof40XJ.png)


Redis 與 RDS 雷同，必須透過 Parameter Group 調整參數，然後在 subnet group 內開啟節點，並把資料存在各個分片內，並具有 Snapshot 快照功能

## 16.6 Memcached

Memcached 就比較不像是資料庫了，也可以做成叢集。資料會分佈在各個節點上，但不會採 Redis 的方式做分片，也就是說如果有節電壞掉，在那個節點的資料就真的遺失了

![](https://i.imgur.com/QKymtPA.png)


### Auto Discovery

ElastiCache 給 Memcached 的建議是多加節點。節點數量越多，資料遺失的時候，損失的資料筆數就會相對的少。

另外因為節點數量較多，在找資料的時候其實有可能會找錯節點，對此 Memchached 推出了 Auto Discovery (自動探索) 功能，當用戶在尋找資料的時候，發現找錯節點時，能夠自動探勘到正確的節點上。

![](https://i.imgur.com/oNqjAgF.png)

## 16.7 考題解析與思路延伸

### 資料庫與記憶體之間的混搭方式 - Lazy Loading

Application 收到資料後，記憶體不載入資料，直到第一次 Application 存取時，從資料庫取出一份，同時轉存一份回 Cache。

**優點**：確保空間使用效率，相對不會保存讀取頻率較低的資料在快取中
**缺點**：第一次讀取要從資料庫拿取，後續讀取速度才會提升

### 資料庫與記憶體之間的混搭方式 - Write Through

Application 收到資料後，存一份回資料庫，同時也存一份在 Cache 中。

**優點**：確保下次讀取資料時， Cache 裡面已存有資料，能快速回應
**缺點**：有可能會保存讀取頻率低的資料，造成空間浪費

## 16.8 ElastiCache 整體架構圖

![](https://i.imgur.com/onsDoiW.png)
