---
title: 如何建立自己的Github部落格 使用Hexo+Github Page
date: 2024-04-19 11:31:16
categories: 其他
tags: 工具
---


網路上目前已經有很多這類的教學了，所以這篇比較像是自己的學習紀錄。
<!-- more -->
### Hexo是什麼？
Hexo是基於Node.js開發的網誌框架，特色主要如下：
* 一鍵部署
* 簡單操作
* 支援MarkDown語法
* 多種模板可參考，更多模板可以參考https://hexo.io/themes/

### 前置作業
1. 安裝Node.js，Hexo需要透過npm安裝。(https://nodejs.org/en/)
2. 安裝Git，用來將檔案發佈到Github Page。(https://git-scm.com/)
3. 安裝Node.js完成後開啟cmd，輸入**npm version**確保node.js可以成功顯示內容，若Node.js安裝成功，這部分應該會顯示一堆套件的訊息。
4. 安裝Hexo，在cmd輸入以下指令：
```
$ npm install hexo-cli -g
```
5. 安裝好後輸入 hexo version，查看hexo版本，如果有顯示版本名稱就代表安裝成功了。
6. 初始化Hexo，在你想要的路徑下輸入以下指令來建立
```
$ hexo init <資料夾名稱>
```
7. 建立好後將目錄切進去
```
cd <資料夾名稱>
```
8. 安裝npm套件，**這部分重要，不要忘記**
```
$ npm install
```
9. 產生靜態檔案，本地啟動看看
```
$ hexo generate
$ hexo server
```
10. Hexo預設路徑http://localhost:4000/
11. 開啟你的Github帳號，沒有就註冊一個。
12. New一個Repository，並把專案命名為<名字>.github.io，這部分是固定的，不可以自定義。
13. 建立好repository後，先把你的專案網址clone起來。
14. 在cmd輸入以下指令安裝git相關套件
```
$ npm install hexo-deployer-git --save
```
15. 調整Deployment。
```
deploy:
  type: git
  repo: <你剛剛clone的路徑>
  branch: <你專案的branch名稱，通常是master>
```
16. 部署它
```
$ hexo deploy
```
17. 你應該會在Github Action看到 pages build and deployment 的訊息，請耐心等他跑完，應該一分鐘內就會執行完成。你應該可以在https://名稱.github.io的路徑下找到它

18. 之後你的每次修改後都應該要
```
$ hexo clean    // 清除之前建立的靜態檔案
$ hexo generate     // 建立靜態頁面
$ hexo deploy     // 部署至 GitHub
```

以上就是安裝教學。接下來說說要怎麼新增文章。

1. 在cmd輸入以下指令
```
$ hexo new "你的文章標題"
```
2. 新增之後，你應該可以在source路徑下找到你自己剛剛建立的文章
3. 使用Markdown語法輸入你想寫的內容
4. 重複上一篇步驟18的動作，你就成功部署新的文章了


#### 以上。

### 參考資料
1. https://hexo.io/zh-tw/docs/writing
2. https://hackmd.io/@Heidi-Liu/note-hexo-github
3. https://hackmd.io/@Heidi-Liu/hexo-theme
4. https://ithelp.ithome.com.tw/articles/10242172
5. https://hexo.io/zh-tw/docs/github-pages
