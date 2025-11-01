---
title: Clean-Code-筆記-2
category: Development
date: 2024-12-29 11:53:08
index_img: /img/clean_code.jpg
tags:
  - Clean Code
  - Java
  - Programming
---

本次是繼續上回的Clean Code筆記，廢話不多說，Let's us begin.

## TDD

#### Principle

1. 撰寫一個單元測試前不應該撰寫產品程式碼
2. 只撰寫剛好能通過的產品程式碼
3. 只撰寫剛好能通過當前測試失敗的產品程式碼

#### 注意要點
* 撰寫測試的好處可以讓你更動程式碼時不那麼恐懼， 需要注意的是，測試程式碼同樣需要與時俱進，若沒有跟著調整容易造成程式碼的腐敗，最終測試程式碼的品質同樣影響產品程式碼。
* 測試程式碼要求的是可讀性 闡明性(Clarity)，而不是效能。
* 盡量維持一個測試一個斷言(Assert)，讓一個測試只會顯示一個結果，方便開發者評估功能。(一個測試只測試一個概念)

#### TDD FIRST原則
* Fast 測試應該要夠快
* Independent 測試應該要能夠獨立運行，不依賴其他測試程式
* Repeatable 測試應該要能夠重複使用
* Self-Validating 測試應該要能夠自我驗證，顯示要測試的概念是否成功運作
* Timely 要在產品程式之前被撰寫，若是之後才撰寫，經常會發現撰寫出來的產品程式碼並不好被測試

## Class-Level

#### 單一職責原則(Single Responsibility Principle, SRP)
單一職責原則主張，一個類別或一個模組應該只能有一個，而且只能有一個被修改的理由。
個人對這段的理解是，一個類別的職責應該是很明確的，他只該負責一件事，而需要修改類別的原因，
必須得是當初認為要做的那件事。 比方說有一個專門處理報表Excel輸出的類別ExcelObject，若今天需要做輸出Excel之外的事 ，比方說：判斷格式、字串拼接、欄位驗證、給定預設值，若因為這些理由而修改了ExcelObject，就是違反了SRP。

#### 凝聚力
類別中的方法，盡量讓每個變數都被使用在每一個方法中。
當類別失去凝聚力時，就應該將它拆開

#### 開放閉合原則(Open-Closed Principle)
類別應該要對擴充具備開放性，但對修改具有閉合性。

#### 隔離修改
類別之間不應該實體依賴，而是依賴抽象，透過一層抽象隔離變化，責任分明且易於測試。

## System

#### 依賴注入(Dependency Injection, DI)
將物件的建立過程從使用中分離出來，這是控制反轉(Inversion of Control, IOC)在相依性管理裡的一種應用手段。控制反轉是將某個物件的第二個職責，移至其他專注於該職責的物件裡，也因此支援了SRP原則。
在相依性管理的範疇中，一個物件不應該負責實體化對本身的相依。而是將責任交給外部。建議可以透過抽象工廠進行。

#### 抽象工廠(Abstract Factory) Example
```java
// 抽象產品：Button
interface Button {
    void render();
}

// 抽象產品：TextBox
interface TextBox {
    void render();
}

// 具體產品：WindowsButton
class WindowsButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering a Windows-style Button");
    }
}

// 具體產品：WindowsTextBox
class WindowsTextBox implements TextBox {
    @Override
    public void render() {
        System.out.println("Rendering a Windows-style TextBox");
    }
}

// 具體產品：MacOSButton
class MacOSButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering a MacOS-style Button");
    }
}

// 具體產品：MacOSTextBox
class MacOSTextBox implements TextBox {
    @Override
    public void render() {
        System.out.println("Rendering a MacOS-style TextBox");
    }
}

// 抽象工廠：UIFactory
interface UIFactory {
    Button createButton();
    TextBox createTextBox();
}

// 具體工廠：WindowsFactory
class WindowsFactory implements UIFactory {
    @Override
    public Button createButton() {
        return new WindowsButton();
    }

    @Override
    public TextBox createTextBox() {
        return new WindowsTextBox();
    }
}

// 具體工廠：MacOSFactory
class MacOSFactory implements UIFactory {
    @Override
    public Button createButton() {
        return new MacOSButton();
    }

    @Override
    public TextBox createTextBox() {
        return new MacOSTextBox();
    }
}

// 客戶端
public class AbstractFactoryExample {
    public static void main(String[] args) {
        // 客戶端選擇一個工廠
        UIFactory factory = getFactory("Windows"); // 可切換為 "MacOS"

        // 使用工廠創建產品
        Button button = factory.createButton();
        TextBox textBox = factory.createTextBox();

        // 渲染產品
        button.render();
        textBox.render();
    }

    // 工廠選擇邏輯
    private static UIFactory getFactory(String osType) {
        return switch (osType) {
            case "Windows" -> new WindowsFactory();
            case "MacOS" -> new MacOSFactory();
            default -> throw new IllegalArgumentException("Unknown OS type");
        };
    }
}
```
▲ 這樣的設計可以靈活切換不同風格的 UI，且對於新的 UI 主題（如 Linux），只需添加新的工廠和產品類即可，無需修改現有代碼。

### 橫切關注點
假設我們有一個交易系統，流程如下：
1. 登入-> 身分驗證  -> 付款
2. 登入-> 身分驗證  -> 退款

嘗試以這樣的角度去理解系統的流程，你會發現「登入」與「身分驗證」並不是真正核心的功能，且兩者重複存在。為了使我們可以更專注在真正重要的商業流程上，我們可以將那些次要的細節作為一個個切片(Aspects)從主流程切割出來，這一個步驟我們稱之為關注點分離。
而實現這樣的構想的技術就是Java Proxy機制，或是AOP(又名切面導向編程)。
關於AOP，可以參考我的另一篇文章：[Spring AOP](https://medium.com/@ziegler7359/spring-boot-aop-%E5%AD%B8%E7%BF%92%E5%BF%83%E5%BE%97-392e2b83cd54)

大約整理到這邊，後面Clean Code的相關討論幾乎都是案例題目了，這邊就不拿出來討論了。

