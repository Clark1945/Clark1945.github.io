---
title: Java Optional API 紀錄
date: 2024-05-23 10:29:06
category: Programming
tags: 
  - Java
  - API
---

之前我在寫Java時，總是覺得一直寫 if (obj != null) {} 這樣的處理會讓整體的程式碼變得不好看，我希望有一種API可以更好地協助我處理NLP問題，於是我後續就開始使用Optional這個套件
，覺得它挺好用的。它好用的點如下：
* 提供兩種對Null的處理方式，使你更安全地操作你的變數。例如說：Optional.of() 與 Optional.ofNullable()
```java
Object object = Optional.ofNullable(object); 
boolean isExist = object.isPresent(); // Null 顯示為 false
if(isExist) {
    System.out.println("String = " + object.get());
}

Object object = Optional.of(object); // 若Object為Null會立即爆錯
```

* 支援Lambda function
```java
Object object = Optional.ofNullable(object);
object.ifPresent(name -> System.out.println(name.length()));
```

* 例外處理可以賦值、執行Lambda、拋錯誤
```java
// 若InputName存在顯示InputName否則回傳orElse Value
String inputName = "Kevin";
String name = Optional.ofNullable(inputName).orElse("John"); 

// 如果為Null 可以使用orElse() 與 orElseGet() 處理Null問題
String text = null;
String defaultText = Optional.ofNullable(text).orElseGet(this::getMyDefault);
String defaultText = Optional.ofNullable(text).orElse(getMyDefault());
public String getMyDefault() {
    return "Default Value";
}

// 如果為null 拋出Exception
String insertName = "Kelvin";
name = Optional.ofNullable(insertName).orElseThrow(IllegalArgumentException::new);
```

* 支援Java Stream API，提供更多資料處理的協助。
```java
// 使用filter可以篩選判斷
Integer year = 2016;
Optional<Integer> yearOptional = Optional.of(year);
boolean is2016 = yearOptional.filter(y -> y == 2016).isPresent();

// 可以使用 function進行值的檢測判斷
Modem modem2 = new Modem(13.10d);
System.out.println("Result = " + priceIsInRange2(modem2));
boolean priceIsInRange2(Modem modem2) {
    return Optional.ofNullable(modem2)
            .map(Modem::getPrice).
            filter(p -> p >= 10).
            filter(p -> p <= 15).
            isPresent();
}
class Modem {
      private Double price;

      Modem(Double price) {
          this.price = price;
      }
      // standard getters and setters

      void setPrice(Double price) {
          this.price = price;
      }

      Double getPrice() {
          return price;
      }
}


// 利用map進行映射，再用filter處理值
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
* 對Collection API的處理
```java
// 使用map 處理陣列
List<String> companyNames = Arrays.asList("paypal", "oracle", "", "microsoft", "", "apple");
Optional<List<String>> listOptional = Optional.of(companyNames);
int size = listOptional.map(List::size).orElse(0);
System.out.println("SIZE = " + size);

Person person = new Person("john", 26);
Optional<Person> personOptional = Optional.of(person);
Optional<String> nameOptional = personOptional
                                    .map(Person::getName)
                                    .orElseThrow(IllegalArgumentException::new);
Optional<String> namesOptional = personOptional.flatMap(Person::getName);

@Test
public void testflatMap() throws Exception {
    List<Integer> together = Stream.of(asList(1, 2), asList(3, 4)) // Stream of List<Integer>
            .flatMap(List::stream)
            .map(integer -> integer + 1)
            .collect(Collectors.toList());
    assertEquals(asList(2, 3, 4, 5), together);
}

public class Person {
    private String name;
    private int age;
    private String password;

    public Person(String name, int i) {
        this.age = i;
        this.name = name;
    }

    public Optional<String> getName() {
        return Optional.ofNullable(name);
    }

    public Optional<Integer> getAge() {
        return Optional.ofNullable(age);
    }

    public Optional<String> getPassword() {
        return Optional.ofNullable(password);
    }

    // normal constructors and setters
    }
```
* 更多參考
```java
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
