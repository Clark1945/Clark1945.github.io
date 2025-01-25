---
title: HATEOAS 是什麼？
date: 2025-01-25 20:53:40
index_img: img/Java_logo.png
category: Software
tags:
 - HATEOAS
 - REST
 - SpringBoot
---

`Spring HATEOAS` 是開發 `REST` 應用程式中常出現的概念，這個名詞在我工作中並沒有出現過，但往往自學就是要接觸自己平時不熟悉的事物。在哭完 “我已經學不動啦！”之後，就讓我們一起來看看 `Spring HATEOAS`吧。

`HATEOAS`是種規範，與我們常用的`JSON`概念相近， `HATEOAS` 的目的是，除了已簡潔的JSON格式回應客戶端請求外，另外告知客戶端如何取得下一步的資源，與 `JSON` 相似但不完全一致。為了好好的說明它，我們先介紹兩種http header 的 Content-Type

- application/json
- application/hal+json

第一種是傳統的 `JSON`格式，另一種則是傳統的 `JSON`加上 **`HAL`** 的支援。 `HAL` 就是實作 `HATEOAS` 的方法，不多說，直接SHOW YOU THE CODE。

```java
// 這是基本的JSON結構
{
  "transactionNumber": "Fool001",
  "transactionResult": "SUCCESS"
}

// 這是HATEOAS的結構
{
  "transactionConfigId": "013",
  "transactionNumber": "Fool001",
  "transactionId": "12673",
  "transactionResult": "SUCCESS",
  "_links": {
    "self": {
      "href": "http://localhost:8080/transaction/12673"
    },
    "transactionConfig": {
      "href": "http://localhost:8080/transactionConfig/013"
    }
  }
}
```

已上方的例子來說， `JSON`只會回應交易單號與交易結果，但使用 `HATEOAS` 不僅可以得到結果，還可以得知下一步想要取得資源的方式，這樣的做法有不少好處。

- 幫助客戶端動態發現資源，若是 `JSON`客戶端必須要透過其他方式(比如說文件)，才能了解更多回應的資訊。
- 除了連結以外，也可以包含一些操作，比方說更新、刪除等等。
- 寫在response裡面，減少了API更新必須同步外部的困擾。

如果是 `Spring` ，會使用 `Spring HATEOAS` 這個套件來實現，想要使用首先須引入套件

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
```

引入之後就可以開始使用，以下是一個簡單的範例：

```java
@RestController
@RequestMapping("/payment")
public class TransactionController {

    @GetMapping("query/{transactionId}")
    public Transaction getTransaction(@PathVariable String transactionId) {
        // 創建 Transaction 物件
        Transaction transaction = new Transaction();
        transaction.setTransactionConfigId("013");
        transaction.setTransactionNumber("Fool001");
        transaction.setTransactionId(transactionId);
        transaction.setTransactionResult("SUCCESS");

        // 添加 self 鏈接
        transaction.add(linkTo(methodOn(TransactionController.class).getTransaction(transactionId))
                .withSelfRel());

        // 添加 鏈接
        transaction.add(Link.of("http://localhost:8080/payment/pay/" + transactionId)
                .withRel("pay"));

        transaction.add(Link.of("http://localhost:8080/payment/refund/" + transactionId)
                .withRel("refund"));

        transaction.add(Link.of("http://localhost:8080/payment/reverse/" + transactionId)
                .withRel("reverse"));

        transaction.add(Link.of("http://localhost:8080/payment/settle/" + transactionId)
                .withRel("settle"));

        return transaction;
    }
}

//
public class Transaction extends RepresentationModel<Transaction> {
    private String transactionConfigId;
    private String transactionNumber;
    private String transactionId;
    private String transactionResult;

    public String getTransactionConfigId() {
        return transactionConfigId;
    }
    public void setTransactionConfigId(String transactionConfigId) {
        this.transactionConfigId = transactionConfigId;
    }
    public String getTransactionNumber() {
        return transactionNumber;
    }
    public void setTransactionNumber(String transactionNumber) {
        this.transactionNumber = transactionNumber;
    }
    public String getTransactionId() {
        return transactionId;
    }
    public void setTransactionId(String transactionId) {
        this.transactionId = transactionId;
    }
    public String getTransactionResult() {
        return transactionResult;
    }
    public void setTransactionResult(String transactionResult) {
        this.transactionResult = transactionResult;
    }
}
```

Transaction是一個物件，我建立了一個查詢交易的API，當客戶端查詢或是工程師們使用Postman進行測試時，會收到以下回應：

```java
GET http://localhost:8080/payment/query/Test001

{
    "transactionConfigId": "013",
    "transactionNumber": "Fool001",
    "transactionId": ${你輸入的訂單編號},
    "transactionResult": "SUCCESS",
    "_links": {
        "self": {
            "href": "http://localhost:8080/payment/query/${你輸入的訂單編號}"
        },
        "pay": {
            "href": "http://localhost:8080/payment/pay/${你輸入的訂單編號}"
        },
        "refund": {
            "href": "http://localhost:8080/payment/refund/${你輸入的訂單編號}"
        },
        "reverse": {
            "href": "http://localhost:8080/payment/reverse/${你輸入的訂單編號}"
        },
        "settle": {
            "href": "http://localhost:8080/payment/settle/${你輸入的訂單編號}"
        }
    }
}
```

用戶使用訂單查詢後可以順便帶入他的下一步可進行的動作，這麼做有以下好處：

1. 靈活度 客戶端的API不必寫死，也可以不必要事先知道有哪些API路徑。
2. 當API有標註版本時，還可以進行靈活的版本控制，比方說原本是1.0版推出了1.1版。
3. 更好的描述性，客戶端可以很明確地知道自己的下一部可以怎麼做。