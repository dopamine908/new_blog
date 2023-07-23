---
title: 金魚腦記不住團隊 git 規範？ 那就用 git template 吧
date: 2021-05-20 14:25:44
tags:
  - Git
categories:
  - Git
cover: https://picsum.photos/id/660/800/600
---

# 金魚腦記不住團隊 git 規範？ 那就用 git template 吧

常常我們在進行團隊協作開發的時候都需要遵循一定的 git commit 規範，而規範通常不是那麼容易被記住，這時候如果在 commit 的時候有人可以提醒我關於規範的大小事就好了

**git template** 正好可以拿來做這樣的事情，以下稍微介紹一下。

這邊假設我們的 commit 規範是使用 [Angular Commit Guidelines](https://github.com/angular/angular/blob/22b96b9/CONTRIBUTING.md#-commit-message-guidelines) 來做範例

多人協作下免不了的是我們必須要將 git commit 去做分類，已達到管理的目的，但你看看像是 [Angular Commit Guidelines](https://github.com/angular/angular/blob/22b96b9/CONTRIBUTING.md#-commit-message-guidelines) 的規範光是他的 commit type 就有非常多種

> * **build**: Changes that affect the build system or external dependencies (example scopes: gulp, broccoli, npm)
> * **ci**: Changes to our CI configuration files and scripts (example scopes: Travis, Circle, BrowserStack, SauceLabs)
> * **docs**: Documentation only changes
> * **feat**: A new feature
> * **fix**: A bug fix
> * **perf**: A code change that improves performance
> * **refactor**: A code change that neither fixes a bug nor adds a feature
> * **style**: Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc)
> * **test**: Adding missing tests or correcting existing tests

而在提交的時候也有個固定格式

```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

也就是說我提交的時候應該要打得像是這樣

```
feat: 新增了某個功能

我這邊新增了某個功能

issue: #1234
```

所以底下介紹如何新增一個 git template

## 設定 git template

首先我們先創一個存放模板的檔案

```
vim ~/.git-template
```

接著將我們期望的模板內容貼上

```
[type]: [subject]

build: Changes that affect the build system or external dependencies (example scopes: gulp, broccoli, npm)
ci: Changes to our CI configuration files and scripts (example scopes: Travis, Circle, BrowserStack, SauceLabs)
docs: Documentation only changes
feat: A new feature
fix: A bug fix
perf: A code change that improves performance
refactor: A code change that neither fixes a bug nor adds a feature
style: Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc)
test: Adding missing tests or correcting existing tests

[body]

issue: #[issue_number]
```

最後我們將這個模板加入到 git 的社定中

```
git config --global commit.template ~/.git-template
```

之後我們下 commit 就會看到預設的模板可以讓我們參考跟編輯了

```
git add .
git commit
```

```
[type]: [subject]

build: Changes that affect the build system or external dependencies (example scopes: gulp, broccoli, npm)
ci: Changes to our CI configuration files and scripts (example scopes: Travis, Circle, BrowserStack, SauceLabs)
docs: Documentation only changes
feat: A new feature
fix: A bug fix
perf: A code change that improves performance
refactor: A code change that neither fixes a bug nor adds a feature
style: Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc)
test: Adding missing tests or correcting existing tests

[body]

issue: #[issue_number]

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# On branch master
# Changes to be committed:
#       new file:   app/Console/Commands/SendEmailCommand.php
#       modified:   app/Console/Kernel.php
#       new file:   app/Jobs/SendEmailJob.php
#       new file:   app/Models/Demo.php
#       modified:   config/database.php
#       modified:   config/queue.php
#       new file:   database/migrations/2020_12_07_053909_create_demo_table.php
#       modified:   routes/web.php
#
~
```
