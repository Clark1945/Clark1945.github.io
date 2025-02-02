---
title: 說明Java @Functionalinterface 的功用
date: 2025-02-02 10:02:32
category: Software
tags:
  - Java
  - Functional Programming
---

Java──眾所皆知，這是一個非常OOP的語言，Java的萬物皆物件的概念使得Java無法像Python一樣寫出不依賴物件的方法，簡言之，Python可以寫出這樣的程式碼而Java不行
```python
def printHello():
   return "Hello"
print(printHello() + "Clark)
```
因此，`Functional Programming`被不少人評價使用Java撰寫總是有點彆扭，也有一部分是這個原因。那在提到@Functionalinterface這個註解前，我們先來提一提Java的FP實作，也就是大家常用的Lambda吧。
Lambda可以將Function以參數的形式傳遞，恰好符合FP將Function作為一等公民的概念，在Java 8 引入Lambda之後，就可以在不使用物件的情形下使用方法，也就是匿名函式(anonymous function)。我們來看看幾個例子
```java
Map<String, Integer> nameMap = new HashMap<>();
nameMap.put("John", 100);
Integer value = nameMap.computeIfAbsent("John", s -> s.length());
```
行3的作用是，假設nameMap中沒有名為John的Key的話，就執行方法中的匿名函式，也就是說將會回傳4這個值。寫法也可以改成方法參照
```java
Integer value = nameMap.computeIfAbsent("John", String::length);
```
或是將方法提取出來，也就是說上面的方法可以改寫成以下
```java
Map<String, Integer> nameMap = new HashMap<>();
nameMap.put("John", 100);
Function<String,Integer> compute = String::length;
Integer value = nameMap.computeIfAbsent("John", compute); // Function介面 接收一個輸入與輸出參數
```
另外Function也接受組合(Composition)，也就是說Function接續執行
```java
Function<String, String> firstExecute = s -> s + "就只會";
Function<String, String> secondExecute = s ->  s + "騙人";
Function<String, String> quoteIntToString = secondExecute.compose(firstExecute);
assertEquals("初華就只會騙人", quoteIntToString.apply("初華"));
```
透過以上程式碼，傳入參數會先執行firstExecute，再執行secondExecute，最後我們使用斷言得出一個結論，那就是：初華就只會騙人。Q.E.D. 證明完畢

Function這個介面目前已經有不少Java得實作，以下介紹部分：
1. Suppliers
2. Consumers
3. Predicates
4. Operators

#### Suppliers
不接受傳入，只會傳出一個值，通常用來延遲產生某個運算值，比方說以下程式碼，直到呼叫.get()之前Lambda都不會被執行。
```java
public String squareLazy(Supplier<String> lazyValue) {
    return lazyValue.get();
}

Supplier<String> soyoriinWord = () -> "就由我來將它結束";
String valueSquared = squareLazy(soyoriinWord);
```

#### Consumers 
與Supplier相反，接受一個或多個傳入，但不回傳任何數，下方案例我們可以印出
**`從來不覺得, 玩樂團, 開心過,`**
```java
Consumer<String> template = name -> System.out.println(name + ",");
List<String> sentenceList = Arrays.asList("從來不覺得", "玩樂團", "開心過");
sentenceList.forEach(template);
```

#### Predicates
Predicate的特性是接收值，但只回傳boolean，常用於 .stream()的 filter()當中。以下案例我們可以找出初華
```java
Predicate<String> findLiar = name -> name.equals("初華");
List<String> aveMujicaList = Arrays.asList("睦", "海鈴", "初華", "小祥", "破貓");
List<String> liarMemberList = aveMujicaList.stream()
        .filter(findLiar)
        .toList();
System.out.println(liarMemberList);
```

#### Operators
Operator是用來處理值的變化，比方說以下程式碼會將每一個Array中的值轉為大寫。
```java
UnaryOperator<String> changeUppercase = String::toUpperCase;
List<String> names = Arrays.asList("soyorin", "tomorin", "rikki");
names.replaceAll(changeUppercase);
```

最後@Functionalinteface這個註解是用來標示Function的，假設我們有多個lambda都需要實作一個功能，而你可以使用介面去掌控它，但這個介面只是單純用來做Functional的定義的，所以為了把他與一般的類別作區隔，於是就可以加入@Functionalinterface做為區隔。

以上就是解釋了，不知道看的人了不了解，希望各位除了知道初華就只會騙人之外，還可以學到更多Java的相關知識，連價最後一天，我要去繼續補Ave Mujica了，就這樣。