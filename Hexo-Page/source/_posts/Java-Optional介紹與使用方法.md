---
title: Java Optional介紹與使用方法
date: 2024-04-24 13:08:13
categories: Coding
tags: Java
---

Optional是Java 8推出時加入的新功能，Optional被發明出來的初衷是為了解決Java常見的NullPointException。
<!-- more -->
為沒寫過Java的人科普一下，NullPointException大多數的發生原因都是針對value為Null進行操作而導致的例外狀況，Optional為此出現。

#### Java 8 Optional 使用方式
使用方式很簡單，可以理解為將值外部再包了一層類別，比方說以下程式：
```
Optional<Student> student = Optional.ofNullable(studentRepo.getAllStudent())
```
Student類別的外部包了一個Optional類別，可以想像為程式外部又包了一個try catch的處理。

#### Optioal常用的方法如下：

* isPresent() **判斷是否為Null，是則回傳True，否則回傳False**
* get() **取得Optional內的值，需要先使用isPresent()確認是否為Null**
* Optional.of() **建立Optional的方法，若為Null會直接報錯**
* Optional.ofNullable() **建立Optional的方法，可以放入Null與非Null物件**
* ifPresent() **如果值為非Null就執行某方法，沒有回傳值**
* orElse() **如果值為Null就用參數代替**
* orElseGet() **如果值為Null就執行該方法，與orElse()相差在可接受傳入參數不同**
* orElseThrow() **如果值為Null就拋出某些Exception**

#### 以下介紹一些常用的方法

```
Optional<String> opt = Optional.ofNullable(InputName);
if(opt.isPresent()){ // 判斷 是否為Null
    System.out.println("String = " + opt.get()); // 非Null才取值
}

opt.ifPresent(name -> System.out.println(name)); // 同一個意思，只是寫得比較簡潔

String name = Optional.ofNullable(InputName).orElse("John"); //若inputName 是 null 回傳"John"

opNull.orElseGet(() -> "John"); // 同個意思 只是orElseGet() 接受的是方法

name = Optional.ofNullable(insertName).orElseThrow(IllegalArgumentException::new); // null　就拋出IllegalArgumentException
```

#### 更多實際應用
使用Optional還有一個好處，就是可以套用Java 8 的API

1. 判斷是否為2016年

```
Optional<Integer> yearOptional = Optional.of(year);
boolean is2016 = yearOptional
                    .filter(y -> y == 2016)
                    .isPresent(); // 判斷值是否為2016年
```

2. 判斷物件價格範圍是否介於15~20之間

```
// 可以使用 function進行值的檢測判斷
Modem modem2 = new Modem(13.10d);
System.out.println("Result = " + priceIsInRange2(modem2));

public class Modem {
      private Double price;

      public Modem(Double price) {
          this.price = price;
      }
      // standard getters and setters

      public void setPrice(Double price) {
          this.price = price;
      }

      public Double getPrice() {
          return price;
      }
}

public boolean priceIsInRange2(Modem modem2) {
    return Optional.ofNullable(modem2)
            .map(Modem::getPrice).
            filter(p -> p >= 10).
            filter(p -> p <= 15).
            isPresent();
}
```

3. 先處理過再判斷。

```
String password = " password ";
Optional<String> passOpt = Optional.of(password);
boolean correctPassword = passOpt
	.filter(pass -> pass.equals("password"))
	.isPresent();
assertFalse(correctPassword);

correctPassword = passOpt
	.map(String::trim)
	.filter(pass -> pass.equals("password"))
	.isPresent();
assertTrue(correctPassword);
```

4. 取得陣列長度，null為0

```
List<String> companyNames = Arrays.asList("paypal", "oracle", "", "microsoft", "", "apple");
Optional<List<String>> listOptional = Optional.ofNullable(companyNames);
int size = listOptional.map(List::size).orElse(0);
System.out.println("SIZE = " + size);
```
5. map 與 flatmap 的差異 
```
        // 假設我們有一個包含字串的列表
        List<String> words = Arrays.asList("Java", "Stream", "API");

        // 使用map操作，將每個字串轉換為它的長度
        List<Integer> wordLengths = words.stream()
                .map(String::length)
                .collect(Collectors.toList());
        System.out.println("使用 map 操作得到的長度列表: " + wordLengths);

        // 使用flatMap操作，將每個字串拆分為單個字符並平坦化為單獨的流
        List<Character> characters = words.stream()
                .flatMap(word -> word.chars().mapToObj(c -> (char) c))
                .collect(Collectors.toList());
        System.out.println("使用 flatMap 操作得到的字符列表: " + characters);

        / /使用 map 操作得到的長度列表: [4, 6, 3]
        // 使用 flatMap 操作得到的字符列表: [J, a, v, a, S, t, r, e, a, m, A, P, I]
         
```
6. 參考用法
```
//  取得第一個非null的值，Optional::get 是為了取得<Stream<String>>
//  方法會全數執行
String found = Stream.of(getEmpty(), getHello(), getBye())
        .filter(Optional::isPresent)
        .map(Optional::get)
        .findFirst()
        .get();

//  同樣意思，但方法不會全數執行
Optional<String> found2 =
      Stream.<Supplier<Optional<String>>>of(this::getEmpty, this::getHello, this::getBye)
              .map(Supplier::get)
              .filter(Optional::isPresent)
              .map(Optional::get)
              .findFirst();

// 若function想要帶上參數 需使用lambda
Optional<String> found3 = Stream.<Supplier<Optional<String>>>of(
              () -> createOptional("empty"),
              () -> createOptional("hello")
      )
      .map(Supplier::get)
      .filter(Optional::isPresent)
      .map(Optional::get)
      .findFirst();


//        判斷是否全數為null
String found4 = Stream.<Supplier<Optional<String>>>of(
                () -> createOptional("empty"),
                () -> createOptional("E")
        )
        .map(Supplier::get)
        .filter(Optional::isPresent)
        .map(Optional::get)
        .findFirst()
        .orElseGet(() -> "default");

assertEquals("E",found4);

private Optional<String> getEmpty() {
        return Optional.empty();
    }

    private Optional<String> getHello() {
        return Optional.of("hello");
    }

    private Optional<String> getBye() {
        return Optional.of("bye");
    }

    private Optional<String> createOptional(String input) {
        if (input == null || "".equals(input) || "empty".equals(input)) {
            return Optional.empty();
        }
        return Optional.of(input);
    }

```
7. map與flatmap
```
public static String testOptionalFlatMap(Student student) {
			
// 這段是使用Optional 檢測 Null
        var score = student.getScore();
        if(score.isPresent()) {
            var word = score.get().getWord();
            if(word .isPresent()) {
                return word.get();
            }
        }
        return "NA";

// 這段是使用Optional Flatmap 檢測 Null
        return student.getScore()
                .flatMap(Score::getWord)
                .orElse("n.a.");
}
```
以上就是常見的基本Optional用法，也許不需要全部都會，只需要基本了解就很夠用了也說不定。

以上