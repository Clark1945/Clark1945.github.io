---
title: Spring Bean延遲初始化
category: Development
date: 2024-10-17 21:13:59
index_img: img/Java_logo.png
 
tags:
  - SpringBoot
  - Bean
---

當你啟動一個專案，Spring Framework會將所有被定義為Bean的物件交給Spring 容器控制，Spring Bean 的一個主要特性是採用了Singleton(單例)模式，也就是Bean的生命週期不是由使用者掌控，而是交給框架，這也是IOC的精神。

但是你有沒有想過，假設你擁有一個開銷昂貴的服務，它的功能並不經常被使用，但是卻佔據了大量的效能，而且又是一個Bean。你可能會想，有沒有一個方法可以讓開發者自己去掌控這個物件的生命週期？

有，就是@Lazy這個功能。

```java
@Component
@Lazy
public class XXService {
   public XXService() {
     System.out.print("XXService Init!");
   }
}
```

@Lazy提供了延遲初始化的功能，使Bean在專案啟動時，被建立的是一個Bean的代理而不是Bean本身，實際上Bean會在被注入到其他類別時才被載入並初始化。而定義方法除了定義在Class上之外，還有以下兩種

```java
@Component
public class MyService {

    private final MyBean myBean;
    
    @Autowired
    public MyService(@Lazy MyBean myBean) {
        this.myBean = myBean;
    }
    
}

與

@SpringBootApplication
public class MyApp {

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(MyApp.class);
        app.setLazyInitialization(true); // 启用全局延迟初始化
        app.run(args);
    }
}
```

你可以定義@Lazy在依賴注入的位置，這會讓實際使用到功能時才初始化Bean，另外也可以像下方直接定義在SpringBoot的設定中(但不建議)

這種方法的優缺點如下：

優點：

1. 減少SpringBoot初始啟動的服務數量，資源的節省，重啟速度也會比較快。
2. 只在需要時建立，避免冗餘占用。

缺點：

1. 延遲，因為被延遲初始化建立的Bean會在被實際使用時才被建立，可能會發生建立花的時間超出預期導致不可預料的情況發生，也有可能初始化出錯，但因為不是由Spring容器掌控，沒有第一時間掌握到錯誤發生。

除了Spring Bean之外，Hibernate的資料庫物件也同樣可以設定延遲初始化，Hibernate的延遲初始化通常是用來避免一次性載入到不需要的Entity，通常載入模式被分為兩種

1. Eager Loading 立即載入
2. Lazy Loading 延遲載入

而它們被使用的方式如下：

```java
@Entity
class Character {
 private String name;
 private int age;
 private long unic_id;
 
 @OneToMany(fetch = FetchType.LAZY, mappedBy = "skillTree")
 private Set<SkillTree> skillTreeSet;
 
 @OneToOne(fetch = FetchType.EAGER)
 @JoinColumn(name = "user_id")
 private User user
}
```

以上定義了一個角色類別(Character)，他對應到多組的技能樹(SkillTree)，但資料庫讀取Character資料時，不會預設一併讀取skillTree，減少不必要的資源浪費。另外針對User則是採用EAGER_LOADING。預設值是 toMany的採用LAZY，toOne的採用EAGER。

然而，俗話說得好，有一好就沒有兩好，但是有三好夏凜是我的 ( 誤

Hibernate使用延遲載入會有一個潛在問題，就是LazyInitializationException，如果試圖在Session已經關閉的時候訪問被關聯的資料時，因為無法載入，所以會出現此錯誤。想來也合理，你是不可能訪問得到你一開始沒有載入的實體的，比方說以下程式

```java
@GetMapping("/getMyCharacter")
public Character getMyCharacter(@Reqeustparam long id) {

Character myCharacter = charaService.findCharacterByUnicId(id);

SkillTree myCharacterSkillSet = myCharacter.getSkillTreeSet(); // 出錯
}
```

解決方法如下：

1. 在Transactional中就把屬性載入。
2. 使用JOIN語法，一開始就載入完整的實體，從而避免無法存取的問題

以上就是今天的分享，@Lazy我在工作上很少看到，但一次看過後就一直很好奇他是什麼，看著看著就產出這篇記錄文章了，希望能對其他有類似困惑的朋友帶來一點幫助，那我們明天見吧