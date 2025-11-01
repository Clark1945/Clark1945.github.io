---
title: Git 常用指令與說明
category: Development
date: 2024-12-24 22:01:08
index_img: /img/git.jpg
tags:
  - Git
---

我們常用的指令有很多，對資訊科技的學生來說，Git的使用已經是基礎了，更何況是一名資訊工作者。本次是要整理那些常用的指令，以及針對一些細節去做釐清學習。
常用的指令整理如下：
```shell
git init #初始化Git Repo
git status #查看 Git 目錄更動狀態
git log # 查看commit紀錄
git add  # 將tracked的資料，添加到add區域
git commit # 將add區域的資料，提交一筆紀錄
git push # 把本地的commit提交至遠端伺服器
git reset -soft # 用來拆commit用的 會把HEAD重製到指定位置
git reset -mixed # 用來拆commit用的 除了把HEAD重製到指定位置，還會把資料變成untracked狀態
git reset -hard # 最少用，重製到指定位置，並刪除之前位置到重製點之前的commit，危險!
git checkout: 用來切換分支或恢復檔案，切換到分支時，使HEAD 指向該分支。
git merge: 將另一個分支的更改合併到當前分支中，會產生合併提交。
git rebase: 用來將一個分支的提交移到另一個分支上，通常用來保持提交歷史簡潔，可以取代 git merge 的方式。
git branch: 列出、創建或刪除分支。
```

#### Detach HEAD: 表示當前不在任何分支上，HEAD 直接指向某個提交，通常需要使用 git checkout 返回分支。

#### Fast-forward: 當目標分支可以直接向前推進而不需要合併提交，會自動進行快進合併，保留簡潔的歷史。

大致整理到這邊，後來其實我越來越少用了，主要是IDE內建Git功能很多都比Git Bash shell 的更易於使用了，也有更便捷的一些做法，所以以上僅作個人紀錄之用。


