---
title: 用Github Page + Hexo + Vercel 建立你的部落格
category: Development
date: 2024-12-22 23:34:44
tags: 
  - Git 
  - GithubPage 
  - Hexo 
  - Vercel
---

作為一名開發者，Github已經是每天都無法脫離的工具，除此之外，Github Page也是許多新手工程師的好朋友，使用簡單的指令就可以讓新手工程師建立一個屬於自己的部落格，以下就是簡單介紹要怎麼搭建一個自己的Github Blog
我們主要使用Hexo作為框架，他主要以Node.js作為編譯工具。
### 環境安裝
1. Git(https://git-scm.com/downloads) 版本控制工具
2. Node.js(https://nodejs.org/zh-tw) 執行Hexo框架必備Node.js的套件管理工具。

安裝完後，打開CMD，輸入 "node -v" 與 "git -v" 看看兩者是不是都有安裝成功，有的話就繼續下去。

步驟解說
===

#### 本機設定
1. 建立你的工作資料夾。
2. 在你建立的工作資料夾目錄下，點選右鍵 "Open Git Bash"，開啟Git cmd
3. 安裝Hexo 輸入"npm install -g hexo-cli" npm 是 node.js的套件管理工具，-g 代表在資料夾外也可以使用(global)，hexo-cli是我們要安裝的node.js套件
4. 安裝完後輸入"hexo -v" 確認hexo 是否有成功安裝，成功的話繼續。
5. 在你的工作目錄下初始化hexo資訊 CMD輸入"hexo init" 與 "npm install"
6. 安裝完後輸入 "hexo s" 啟用hexo運作的伺服器範例
7. 你應該會看到訊息 "Hexo is running at port 4000"，這時可以開啟URL確認看看。

這樣一來基本的安裝就完成了！

#### GitHub
1. 註冊Github
2. 以"你的名字.github.io"建立Repository。

#### 部署至GithubPage
1. 在Hexo工作資料夾下安裝套件 "npm install hexo-deployer-git --save"
2. 打開_config.yml文件，修改以下設定
```
deploy:
type: git
repository: <你的GithubPage路徑>
branch: <你要推的分支>
```
修改完後，我們要重新打包，在cmd 輸入 "hexo g" 產生靜態文件。
接著輸入 "hexo d"，把你的Hexo Page推到Github上。
接著打開你的Repository，並觀察Actions中，是否WorkFlow成功執行完。如果正常結束，那恭喜你，你已經完成了。

感謝觀看，有任何問題歡迎下方留言，就這樣掰掰。
