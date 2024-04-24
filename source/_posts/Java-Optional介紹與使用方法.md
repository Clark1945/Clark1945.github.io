---
title: Java Optional介紹與使用方法
date: 2024-04-24 13:08:13
tags: Java
---

Optional是Java 8推出時加入的新功能，Optional被發明出來的理由是為了解決Java常見的NullPointException，為沒寫過Java的人科普一下，NullPointException大多數的發生原因都是針對Null進行操作而導致的例外狀況，Optional就是為此出現的。

#### Java 8 Optional 使用方式
使用方式很簡單，他其實是將你的值外部再包了一層類別，比方說以下程式：
```
Optional<Student> student = Optional.ofNullable(studentRepo.getAllStudent())
```
可以看到，Student類別的外部包了一個Optional類別，可以想像就像是程式外部又包了一個try catch一樣。
Optioal常用的方法如下：
```
Optional<String> opt = Optional.ofNullable("Baeldung");
if(opt.isPresent()){ // 判斷 是否為Null
    System.out.println("String = " + opt.get()); // 非Null才取值
}

opt.ifPresent(name -> System.out.println(name)); // 同一個意思，只是寫得比較簡潔
```
針對Null的處理
```
    String name = Optional.ofNullable(InputName).orElse("John"); //若inputName 是 null 回傳"John"

    opNull.orElseGet(() -> "Not Exist"); // 同個意思 只是orElseGet() 接受的是方法

    name = Optional.ofNullable(insertName).orElseThrow(IllegalArgumentException::new); // null　就拋出IllegalArgumentException
```
另外，使用Optional還有一個好處，就是可以套用Java 8 的API
```
Optional<Integer> yearOptional = Optional.of(year);
boolean is2016 = yearOptional.filter(y -> y == 2016).isPresent(); // 判斷值是否為2016年


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

### 其他更多使用方法
