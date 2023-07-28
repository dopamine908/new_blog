---
title: 部落格復活之路 pt.1 - 自動更新
date: 2023-07-28 20:23:03
tags:
  - CI/CD
  - Blog
categories:
  - 日常開發筆記
cover: https://picsum.photos/id/137/800/600
---

# 部落格復活之路 pt.1 - 自動更新

## 荒廢兩年終於復活的部落格

筆者我的 [部落格](https://dopamine908.github.io/) 自從兩年前換了工作之後，一路荒廢更新到前陣子，在生活中心有所轉移後，決定要來直面我荒廢已久的部落格了。

這兩年間雖然時不時也會寫點筆記、製作些讀書會有關的報告，但卻毫無更新動力。於是就把這些文章暗自存在 hackmd.io 上沒有發佈。

兩年的時間不長不短，但久到我可以忘記我的部落格專案是存在哪一個 Github Repository 了，基本也忘記了部落格框架的使用方法，檔案架構... etc。

簡單來說，在悠久且漫長的歲月中，我什麼都不記得了！

### 復活之路

腦袋一片空白要怎麼復活部落格？這時候有一個愛寫部落格的好友就很重要了，筆者在這邊鼓勵大家多多結交愛寫部落格的朋友，你有什麼問題直接投靠就對了。

**所以本文致謝 [Bear](https://github.com/YNCBearz) 在筆者最徬徨無助的時候猶如雪中送炭般的給予筆者人生前進道路中的一盞明光**

實際上呢，也真的是辛苦 [Bear](https://github.com/YNCBearz) 了，一步一步引領著我，帶我回想起來悠久流長的歲月中，部落格的使用、建立、部署方式。

## 自動更新部落格？

筆者的部落格是由 [Hexo](https://hexo.io/zh-tw/) 搭配 [Markdown](https://markdown.tw/) 語法及 [Github Page](https://pages.github.com/) 建立的，寫部落格的方式就是寫寫 markdown 然後使用些指令去預覽，並且在確定要 po 文的時候下部署指令。

#### 預覽部落格
```
hexo server
```

#### 部署部落格
```
hexo clean && hexo deploy
```

至於關於 Hexo 的操作、部落格運作方式、如何建立...之類的事情，我們把這些議題留到之後的文章記錄。（工程師的 TODO，有緣就會實作了）

### 關於更新部落格

荒廢更新部落格的這兩年，筆者也是有所成長，變得更<span style="color:red">**懶惰**</span>了！

> 懶惰是本部落格三大核心精神之一

所以呢，每次寫完部落格要更新到部落格的 Github Repository 之後還要部署，這要兩步驟欸，簡直太麻煩了吧！

好啦，其實是最近弄了一些 CI/CD 有關的東西，然後單純的想說：「如果我可以只要 push ，然後其他事情自動化腳本都會幫我搞定那不是很好嗎？」

於是乎我找到了最近很常打交道的 [Github Action](https://docs.github.com/en/actions)，開始尋找有沒懶人包可以讓我更懶一點的完成這件事情。

## 官方推薦

在東翻西找的過程中，在 Hexo 的官方文件裡找到了[在 Github Page 部署的 Github Action Workflow yml 寫法](https://hexo.io/zh-tw/docs/github-pages)。

Hexo 果然沒有忽落廣大的懶人們，連這一步都幫我們想好了，那麼我們就來嘗試使用它給的 yml 檔吧！

## 跟想像中不一樣

結果在一番嘗試後，Github Action 的狀態都顯示成功，但我看頁面都沒有更新，所以我又仔細了繼續看文件（理論上應該是一開始就要先看完比較對，但我看到可以直接貼的 yml 就見獵心喜...），結果發現文件下方有這麼一段說明

> 5. 當部屬作業完成後，產生的頁面會放在儲存庫中的 gh-pages 分支。
> 6. 在儲存庫中前往 Settings > Pages > Source，並將 branch 改為 gh-pages。
> 
> #### 專案頁面
> 如果你希望網站部署在 <你的 GitHub 用戶名>.github.io 的子目錄中：
> 
> 1. 開啟你在 GitHub 的儲存庫，並前往 Settings 頁面。更改你的 Repository name 使你的部落格網址變成 username.github.io/repository

於是我查看了分支，的確這個腳本幫我部署到了 ```gh-pages``` 分支，並且查看了 https://dopamine908.github.io/new_blog 頁面（沒錯我的 repository 名稱就是那麼粗暴的叫做 new_blog），結果發現這邊有我最近的更新內容，看起來好像一點也沒錯。

但這不是我要的啊！我原本期望是更新完之後可以在 https://dopamine908.github.io 就可以簡單地看到最新的內容了，但是這個腳本的行為卻不是那樣，於是乎我又在東翻西找有沒有什麼設定可以做調整，但都搜尋無果。

而這個腳本中的 [Github Action Package : peaceiris/actions-gh-pages](https://github.com/marketplace/actions/github-pages-action) 裡面似乎也沒有相關設定可以調整。（又或許有只是筆者我沒能悉知這些內容，畢竟筆者沒有耐心慢慢看文件...如果您知道還懇請開示）

## 自幹吧

不就是一行部署指令嗎？我自己寫也可以（吧）！

所以又到了發揮工程師黃金價值的時刻了，來！自幹一個吧！

於是乎經過了一番冒險，為您呈現的是最後的腳本本人。

#### [.github/workflows/auto-deploy-blog.yml](https://github.com/dopamine908/new_blog/blob/master/.github/workflows/auto-deploy-blog.yml)
```yaml=
name: Auto Deploy Blog

on:
  push:
    branches:
      - master

jobs:
  pages:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v3
        with:
          token: ${{ secrets._GITHUB_TOKEN }}
          submodules: recursive
      - uses: webfactory/ssh-agent@v0.7.0
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
      - name: Use Node.js 16.x
        uses: actions/setup-node@v2
        with:
          node-version: '16'
      - name: Cache NPM dependencies
        uses: actions/cache@v2
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Install hexo-cli
        run: |
          npm install -g hexo-cli
          hexo version
      - name: Set Git Information
        run: |
          git config --global user.email "dopamine908@gmail.com"
          git config --global user.name "Mobius"
      - name: Deploy
        run: |
          hexo clean 
          hexo deploy
```

接下來挑幾個重點稍微說明下好了

### on

```yaml=
on:
  push:
    branches:
      - master
```

這裡主要是在指定，當我的 ```master``` 分支被 push 之後，即觸發這個 workflow，會特別提這個是因為現行大家的 git 版本不太一樣，舊版本的主分支會使用 ```master``` ，新版本的主分支會使用 ```mian```，如果您是新版本的，別忘記將這個值修改為 ```main```。

### secrets._GITHUB_TOKEN

```yaml=
- uses: actions/checkout@v3
  with:
    token: ${{ secrets._GITHUB_TOKEN }}
    submodules: recursive
```

這邊因為會需要取用個人的專案權限，所以這邊會使用到 ```secrets._GITHUB_TOKEN``` 這個值，這邊會需要到個人的 

Settings > Developer Settings > Personal Access Token 去新增一把新的 token

![](https://hackmd.io/_uploads/HJFbVybin.png)

並且將這把 token 新增到

Github Repository > Settings > Secret and Variable > Actions

![](https://hackmd.io/_uploads/BJkSSkbs2.png)

### secrets.SSH_PRIVATE_KEY

```yaml=
- uses: webfactory/ssh-agent@v0.7.0
  with:
    ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
```

這邊我們最終想要將部落格專案打包好的東西發佈 Github Page Repository 中，你可以想像，我們要在 Github Action Runner 的主機中去 push code 到另一個自己的 Github Repository 中，乍看之下挺完美的，但是！

Github Action Runner 每次被呼叫都是一個新的空白的環境，一個空白的環境裝了 git 就可以推 code 到你的任何專案嗎？

即便是 pubilc 的專案也不行吧？這樣太恐怖了，我們總是需要些身份驗證的機制。

忘記從什麼時候開始的，github 在你 push code 的時候已經強制需要換成 ssh-angent 的驗證機制了，這中間可是有一些設定流程要做的，於是乎在這邊筆者找到了一個專門在幹照這件事情的 [Github Action Package : webfactory/ssh-agent](https://github.com/marketplace/actions/webfactory-ssh-agent)，所以我們只要簡單的設定下 ssh-angent 相關的內容就可以了

首先參照[這邊](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)的指令生產公鑰、私鑰。

```
ssh-keygen -t ed25519 -C "your_email@example.com"
```
再來將你的私鑰內容新增到

Github Repository > Settings > Secret and Variable > Actions

![](https://hackmd.io/_uploads/BkkjOy-jh.png)

並且將公鑰新增到 Settings > SSH and GPG Keys

![](https://hackmd.io/_uploads/BkXztyZo3.png)

如此一來就完成了設定可以 push 到別的專案的權限了～

### Install hexo-cli

在一連串基礎環境設定(如：安裝 nodejs, 設定快取)之後，最重要的 Hexo 打包工具還沒有安裝，所以這邊我們就安裝一下 [hexo-cli](https://github.com/hexojs/hexo-cli)

```yaml=
- name: Install hexo-cli
  run: |
    npm install -g hexo-cli
    hexo version
```

### Set Git Information

在最後要 push 之前我們還缺少了一點 git 相關的設定，這裡將設定補上。

```yaml=
- name: Set Git Information
  run: |
    git config --global user.email "dopamine908@gmail.com"
    git config --global user.name "Mobius"
```

### Deploy

最後一哩路，將 Hexo 的部署指令新增上去！

```yaml=
- name: Deploy
  run: |
    hexo clean 
    hexo deploy
```

## 最終成果

經過了一連串的努力我們終於可以快樂的在 push 部落格文章之後

在部落格的 Github Repository > Actions 中看到我們自動被觸發的 Action

![](https://hackmd.io/_uploads/H1PthJWjh.png)

點進去就可以看到腳本執行的過程跟 logs

![](https://hackmd.io/_uploads/S1Kepkbin.png)

於是乎，寫部落格這件事情的阻礙又更少了，皆大歡喜～皆大歡喜～

## Reference

- [Hexo](https://hexo.io/zh-tw/)
- [Markdown](https://markdown.tw/)
- [Github Page](https://pages.github.com/)
- [Github Action](https://docs.github.com/en/actions)
- [Github Action Package : peaceiris/actions-gh-pages](https://github.com/marketplace/actions/github-pages-action) 
- [Github Action Package : webfactory/ssh-agent](https://github.com/marketplace/actions/webfactory-ssh-agent)
- [Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
- [hexo-cli](https://github.com/hexojs/hexo-cli)
- https://hackmd.io/@CynthiaChuang/Generating-a-Ssh-Key-and-Adding-It-to-the-Github
