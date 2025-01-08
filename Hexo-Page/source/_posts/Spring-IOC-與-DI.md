---
title: Spring IOC 與 DI
date: 2024-10-19 20:44:51
index_img: /img/Java_logo.png
category: Software
tags:
  - SpringBoot
  - Inversion Of Control
  - Dependency Injection
---

當初剛學Spring的時候，看著教學大談Spring的設計概念，一直都沒有什麼感覺，直到後來真正開始看大型專案時，才慢慢了解到Spring IOC 與 DI 的概念。 因為這部分 其實並不太好描述，我會參考其他人的描述加上自己的體悟慢慢說明它。

### IOC (Inversion of Control) 是什麼？

IOC，中文稱作控制反轉，是Spring設計上最為核心的一點。以往整個應用程式的生命週期是由開發者所開發的程式碼定義的，開發者會根據需求建立當下所需要的服務，比方說new一個Scanner去偵測使用者的輸入等等。

那控制反轉簡單來說，就是把新建立物件這一件事，交給框架去處理，而不必自己撰寫。物件的宣告、使用與回收都交給Spring框架去執行，讓使用者可以更專注在開發的程式碼上面。

之所以會這樣做有很多原因，比方說前幾次談過的，物件在被new 之後，就會在記憶體中佔走一塊位子，不好好處理的話可能會造成記憶體資源的浪費或無法被釋放等等。那與其如此就交給框架幫你煩惱這部分。

那DI 是什麼呢？ DI 就是指IOC是怎麼被實現的，其實跟Spring沒啥關係，比方說以下程式：

```jsx
class Student{
   private int id;
   private String name;
   private int score;
}
```

假設你想要引入Exam這個物件去協助計算整學期的平均，你可以怎麼做？

1. Attribute Injection 就是直接引入，簡單暴力

```jsx
class Student{
   private int id;
   private String name;
   private int score;
   
   @Autowired
   private ExamService exam;
}
```

1. Constructor Injection 透過物件的建構式去注入你想使用的服務。

```jsx
class Student{
   private int id;
   private String name;
   private int score;
   private ExamService exam;
   
   @Autowired
   public Student(int id,String name,ExamService exam) {
     this.id=id;
     this.name=name;
     this.exam=exam;
   }
}
```

就這樣？對，就這樣。

另外，你們肯定會注意到 @Autowired這個註解吧，這個註解是代表了它被注入的對象是一個Spring Bean 物件，而不是一般物件，這代表的是框架會幫你在專案啟動階段時，就處理好物件的初始化。

但可以被Spring 容器管理的物件，本身必須得是一個Spring Bean。這點很重要，請筆記。以上面的例子來說，你必須將Exam物件宣告為一個Spring Bean，才能夠作用。同時Student也必須是一個SpringBean

Spring Bean 可以是一個屬性、可以是一個方法、也可以是一個物件，用途包山包海，在此不多做贅述。至於宣告物件成為Bean的方法呢…

說到這裡，你有玩過天歲之咲稻姬嗎？也許沒有，但沒關係，這不重要。

將物件宣告為Spring Bean的方法有很多種，也根據你的Java版本不同有差異。常見的部分有

@RestController、@Service、@Repository、@Component、@Configuration、@Bean等等。

你可以根據實際商業場景宣告符合需求的註解，他們各自有不同的用途，可以去了解看看。

今天的部分就到此為止了，下周見吧。