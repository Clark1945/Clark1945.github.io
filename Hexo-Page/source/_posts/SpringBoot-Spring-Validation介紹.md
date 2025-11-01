---
title: SpringBoot Spring Validation介紹
category: Development
date: 2025-01-07 20:47:16
index_img: img/Java_logo.png
 
tags:
  - Java
  - Hibernate
---

今天要介紹的是一個簡單卻實用的小工具──SpringBoot Validator，它可以協助我們做物件的驗證，減少物件建立後繁瑣的驗證邏輯，以下會介紹一些常用的方法。首先要使用的話我們要先引入依賴，如果是使用Maven的話可以參考以下程式碼：

```jsx
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

假設我們今天有一個交易的Object，我們希望在實際進入Controller前先驗證傳入值是否正確的話，可以看看以下舉例

```java
public class TransactionObject {
    @Size(min = 10, max = 28, message = "長度必須介於10~28之間")
    private String transactionId;
    @Email(regexp = "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$", message = "Email格式不符合")
    private String email;
    @NotBlank(message = "Type is not exist")
    private String transactionType;
    @NotNull
    @PositiveOrZero
    private BigDecimal amount; // 交易金额
    @PastOrPresent
    private LocalDateTime transactionDate;

// ... getter setter constructor
}
```

在TransactionObject中，
@Size會在物件建立時判斷transactionId是否在長度10~28之間，若不符合會報Excpetion，訊息為 長度必須介於10~28之間。
@Email 利用正規表達式驗證傳入email格式是否符合，若不符合就跳出錯誤訊息。

@NotBlank 類似於撰寫StringUtils.isNotBlank()，@NotNull則用來判斷是否不為Null

@PositiveOrZero 宣告了數值必須是正數或者0

@PastOrPresent 宣告了交易時間，必須是現在或者現在以前，因為不會有未來的交易。

以上，透過物件在建立的同時便完成驗證就是Spring Validator的特色。

通常他會在API Controller就被使用，如下：

```java
@RestController
public class TransactionController {

    private TransactionService transactionService;
    public TransactionController(TransactionService transactionService) {
        this.transactionService = transactionService;
    }
    @PostMapping("/payment")
    public TransactionObject payment(@RequestBody @Valid TransactionObject transactionObject) {

        boolean success = transactionService.payment(transactionObject);
        return success ? transactionObject : null;
    }
}
```

其中@Valid 代表了啟動物件驗證的方法，如果不符合就會直接拋出org.springframework.web.bind.MethodArgumentNotValidException，避免讓不合法的物件進入程式內部。

然而，在Controller作物件驗證有一個缺點，就是無法撰寫try catch，這部分要怎麼解決呢，我們明天在說明吧，感恩感謝。

參考資源：

[Spring Boot中validation-api和hibernate-validator详解及快速应用实践，@Valid BindingResult实现接口入参自动检验，Java实体字段校验-CSDN博客](https://blog.csdn.net/Hello_World_QWP/article/details/116129788)

https://www.baeldung.com/spring-boot-bean-validation