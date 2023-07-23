---
title: 金魚腦記不住想要 ssh 的 ip ?
date: 2021-06-25 14:25:44
tags:
  - SSH
categories:
  - 日常開發筆記
cover: https://picsum.photos/id/660/800/600
---

# 金魚腦記不住想要 ssh 的 ip ?

筆者開發的情境時常會需要去連線不同的主機，所以必須要去記憶很多組主機的 ip ，這樣在使用 ssh 的時候才有辦法知道要連線的目標，但是 ip 實在是太難記憶了

這時候我們可以透過設定 ssh config 來減輕我們腦袋的負擔

```
cd ~/.ssh
vim config
```

我們可以新增設定的形式

**config**
```
Host dev
Hostname 127.0.0.1
identityfile ~/.ssh/id_rsa
User mobius
```

- **Host**:你想要設定的主機名稱
- **Hostname**: ip
- **identityfile**: 連線主機用的 key 存放位置
- **User**: 用來登入的 user name

如果想要設定多組的話就是往下持續遞增這樣的區塊就可以了

**config**
```
Host dev
Hostname 127.0.0.1
identityfile ~/.ssh/id_rsa
User mobius

Host production
Hostname 123.123.123.123
identityfile ~/.ssh/id_rsa
User mobius
```

如上設定，以往我們想要連進去 production 我們可能要輸入下列指令

```
ssh mobius@123.123.123.123
```

設定完之後我們可以用較為好記憶的方式去連線主機

```
ssh production
```
