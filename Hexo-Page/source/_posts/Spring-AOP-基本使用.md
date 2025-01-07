---
title: Spring AOP 基本概念與使用
date: 2024-05-01 12:26:20
categories: Coding
tags: 
 - Spring
 - AOP
---

到目前為止，在程式開發的發展上已經誕生了許多風格，如物件導向程式設計(Object-Oriented-Programming)、
<!-- more -->
Functional-Programming FP，這些方法都有好有壞，但都有共同的目的，就是簡化重複的程式碼，物件導向程式語言使用物件的封裝、繼承特性去約束程式碼，而FP則是專注在整個流程上，透過將功能建立為一個個微小的Function，進行開發。FP的好處是，通常可讀性可以提升不少。此外因為Function小，各功能也很好共用。
但今天要介紹的並不是這兩者，而是另外一種風格，被稱為切面導向程式設計(Aspect-Oriented-Programming)。
AOP的主要訴求是，將整個程式流程視為一個Flow，假設我有兩隻API，分別提供了付款、退貨兩功能。
付款的流程如下：驗證 > 交易 > 回傳結果
退貨的流程如下：驗證 > 退貨 > 回傳結果
而AOP核心精神就是將兩者相同的「驗證」部分提取出來作為一個切面(Aspect)，讓付款與退貨的API可以集中在真正重要的商業邏輯。也就是所謂的「關注點分離」的概念。
常見的Annotation如下：
* @Aspect - 將類別宣告為一個切面，切面方法放在裡面。
* @Pointcut - 切入點。定義切入位置，通常使用execution切點表達式表達。
* @Before 前置通知，常在切入方法執行前執行。若此方法回報異常，則不會執行方法。而是執行@After與@AfterThrowing
* @After 後置方法 - 再切入方法執行後執行
* @AfterReturn - 在方法正常結束return後執行
* @AfterThrowing - 拋出異常後執行，若無異常則不會執行
* @Around - 環繞通知，可以視為@Before+@After+@AfterThrowing+@AfterReturn
* @Order - 指定切入執行順序

### Spring AOP 為何不生效？原因？
**A:** 當切面在類別上運作時，我們使用的類別實際上是由Spring替我們生成了一個代理對象，故切面必須依賴於類別的呼叫。簡單來說 a.getMessage() 可以套用 AOP 但 getMessage() 是不行的！
解決方法：
1. 注入類別，使用self
```
@Component
public class TestBean {

    @Autowired
    private TestBean self;  // 这个装配的对象是代理对象

    public void hello() {
        System.out.println("hello");
        self.hi(); // 這樣就會成功
    }

    public void hi() {
        System.out.println("hi");
    }
}
```

2. 啟用exposeProxy
```
@EnableAspectJAutoProxy(exposeProxy=true)
@Component
public class TestBean {

    public void hello() {
        System.out.println("hello");
        TestBean self = (TestBean) AopContext.currentProxy();  // 获取当前代理
        self.hi();
    }

    public void hi() {
        System.out.println("hi");
    }
}
```

### 範例
以下是一個範例的切面
```
package com.example.demo;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

@Component
@Aspect
public class StrangeAOP {

    @Pointcut(value = "execution(* com.example.demo.controller..*.*(..))")
    public void pointcut(){

    }

    @Before("pointcut()")
    public void before(JoinPoint joinPoint){
        System.out.println("前置通知");
    }
    @After("pointcut()")
    public void after(JoinPoint joinPoint){
        System.out.println("后置通知");
    }
    @AfterReturning(pointcut="pointcut()",returning = "result")
    public void result(JoinPoint joinPoint,Object result){
        System.out.println("返回通知:"+result);
    }
}

@RestController
public class TestController{

    @GettingMapping("/)
    public String getIndex(){
        System.out.print("Index");
        return "This is Index";
    }
}
```

印出訊息的順序應該是 

前置通知 > This is index > 后置通知 > 返回通知+result


以上