---
title: 快速鍵好一個SpringBoot專案
date: 2024-10-07 20:24:15
index_img: /img/Java_logo.png
category: Software
tags: 
  - Java
  - SpringBoot
---
好啦，講了那麼多篇Java了，我想是時候來看看SpringBoot了

過去，還記得我在學生時期時，光是安裝Java JDK 等等的就可以搞上半天，你得要下載JDK，你還要知道自己需要的是Java EE還是Java SE，下載後你還要知道怎麼開啟系統環境變數加入path以及JAVA_HOME等等。舉凡少了一個細節都會讓一個初接觸程式的朋友感到挫折與對外面籃球場的憧憬。 ( 迷之聲：我還沒說Ecplise呢 )

所幸後來，JetBrain的Intellij IDEA 推出了，並最大化簡化了初始環境安裝的複雜程度，讓Java開發者不必在對Java安裝環境焦頭爛額。

首先，要安裝SpringBoot，你只需要安裝Intellij IDEA：https://www.jetbrains.com/idea/

如果你是免費仔的話安裝Community 版本，這個版本一些功能比較侷限，但如果你只是用來練習而非大量使用的話，Community就足夠應付了。

如果你是付費仔、或是學生的話可以安裝Ultimate版本，這個項目比較完整，但基本功能都是不受影響的，請各位自行選擇。

在等待安裝的同時，各位可以先開啟Spring官方提供的專案產生器：[Spring Initializr](https://start.spring.io/;)

在這裡你可以選擇你想要使用的版本及工具，若是懶得想，可以參考我的這張圖。

記得要引入Spring Web，因為我們要撰寫的是一個Web Application。

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/48871ff2-43a1-4294-ac43-f706269b20d2/b27bbcd3-1719-4916-a911-c8f23e86391f/image.png)

下載並解壓縮後，就可以使用你剛剛下載好的Intellij IDEA開啟專案。

接下來你應該會看到這樣的畫面，這就是一個最原始的SpringBoot專案，簡單吧。

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/48871ff2-43a1-4294-ac43-f706269b20d2/6a5286ad-4cdd-464f-b32b-c06c25eceb25/image.png)

再來介紹一下右邊的目錄結構，主要的程式碼都放在src目錄下，src底下有main與test，test主要是提供給寫測試案例時使用，一般開發的程式碼則會聚集在main中。

main中包含了java與resource，resource主要是可以放一些靜態資源，比方說圖片或檔案等等，以及前端的畫面會放在template位置，java就…..java。

另外有一個值得一提的就是application.properties這個檔案。application.properties 是SpringBoot用來簡化配置的一個檔案。假設你想要撰寫Application開放的port、DB的連線URL、是否要套用熱重啟等等都可以放在這裡配置，只要記得遵照SpringBoot的格式，就可以運作，而不必撰寫程式碼。

再來我們來看到DemoApplication這個檔案。這邊是程式的起始點，@SpringBootApplication宣告了這是一個SpringBoot的應用程式。

最後我們來看一下外面的pom.xml檔案。

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/48871ff2-43a1-4294-ac43-f706269b20d2/4d17bc93-d60b-4be7-a2f0-9f2882315a69/image.png)

你可能眼花撩亂，但沒關係，你不必每一行都了解它。先關注在現有的重點就好：

1. name 它反映了你的應用程式的名稱
2. description 它反映了你的應用程式的描述
3. java.version 這部分決定了你要在專案使用的JDK版本
4. <dependecies> 這部分內部就是放置你想要引入的dependency、比方說剛剛引入的Spring Web就會以 spring-boot-starter-web顯示在dependency在裡面，另外還有一些SpringBoot預設引入的dependency，就不一一多做介紹了。
之後若是要引入套件，不必自己寫，直接從Maven官方取得就可以複製貼上了，相當方便。https://mvnrepository.com/artifact/org.springframework/spring-aop/6.1.12

以上是我的極短SpringBoot專案介紹，下一輪我們要進行開發，後續會提到更多有關Spring SpringBoot的事，那麼我們下次見拉。