---
title: Java Stream用法、MapStruct、Lombok語法
category: Development
date: 2024-05-01 14:16:37
tags: Java
---

Java 8 加入Stream語法，提供了很多針對陣列物件的便利功能，以下介紹一些常見的用法。
<!-- more -->
### Java Stream 常見用法

Java Stream 主要功能如下

1. Filter 透過設定的條件過濾元素

```
// 找出非Empty數量
long memberCount = gameDevMember
.stream()
.filter(string -> !string.isEmpty())
.count();

// 找出非重複的第一位成員
String firstMember = gameDevMember
.stream()
.filter(string -> !string.isEmpty())
.distinct()
.findFirst()
.get();

// 找出 陣列中以AL1S開頭的任何一個，若沒有符合的回傳"IS NOT EXIST"
String findQueen = gameDevMember
.stream()
.filter(string -> string.startsWith("AL1S"))
.distinct()
.findAny()
.orElse("IS NOT EXIST");

// 找出前三位成員
String findThreeMember = gameDevMember
.stream()
.filter(string -> !string.isEmpty())
.distinct()
.limit(3)
.collect(Collectors.joining());

```

2. Map 映射每个元素到對應的結果

```
List<Food> foodObj = Arrays.asList(
    new Food("酸辣湯"),
    new Food("餛飩湯"),
    new Food("肉羹湯"),
    new Food("牛肉湯"),
    new Food("貢丸湯"), 
    new Food("下水湯"));

// 取得物件的字串
List<String> foodList = foodObj.stream().map( food -> food.getFoodName()).toList();
System.out.println("食物有"+foodList);
```

3. forEach 迭代Stream中的每個元素 ( 同樣可以修改 但是不會回傳)

```

List<Food> foodObj = Arrays.asList(
    new Food("酸辣湯"), 
    new Food("餛飩湯"), 
    new Food("肉羹湯"), 
    new Food("牛肉湯"), 
    new Food("貢丸湯"), 
    new Food("下水湯"));

List<String> foodList = foodObj.stream()
.map( food -> food.getFoodName())
.toList();
System.out.println("食物有"+foodList);

// 將所有FoodName 加上 NT$
foodObj
.stream()
.forEach(food -> {
			food.foodName = "NT$"+food.getFoodName();
		});

```

4. reduce (元素匯總成單一結果。)

```
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);
//將列表元素加總
Integer sum = integers.stream().reduce(0, (a, b) -> a + b);
```


### MapStruct 用法，MapStruct可以將兩個類別互相轉換

```
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class UserPo {
    private Long id;
    private Date gmtCreate;
    private Date createTime;
    private Long buyerId;
    private Long age;
    private String userNick;
    private String userVerified;
}


@Data
public class UserEntity {
    private Long id;
    private Date gmtCreate;
    private Date createTime;
    private Long buyerId;
    private Long age;
    private String userNick1;
    private String userVerified;
}

// 轉換器
@Mapper
public interface IPersonMapper {
    IPersonMapper INSTANCT = Mappers.getMapper(IPersonMapper.class);
    @Mapping(target = "userNick1", source = "userNick") // 如果名稱不同無法對應 就使用 這個做Mapping
    @Mapping(target = "age", source = "age", numberFormat = "#0.00")

    UserEntity poToentity(UserPo userPo);
}

// 測試PO 與 Entity 轉換
public static void testNormal() {
    System.out.println("-----------testNormal-----start------");
    UserPo userPo = UserPo.builder()
            .id(1L)
            .gmtCreate(new Date())
           .buyerId(666L)
           .userNick("測試mapstruct")
           .userVerified("ok")
           .age(18L)
           .build();

    System.out.println("USERPO = " + userPo);
    UserEntity userEntity = IPersonMapper.INSTANCT.poToentity(userPo);
    System.out.println("ENTITY = " + userEntity);
    System.out.println("-----------testNormal-----ent------");
    }

```

### @Lombok 的Annonation

* @Data 注解：
@Data 是 Lombok 提供的最常用的注解之一，它结合了 @ToString、@EqualsAndHashCode、@Getter、@Setter 以及 @RequiredArgsConstructor 的功能。它会自动生成这些方法，并为类的每个非静态字段创建 Getter 和 Setter 方法，还会生成 toString 方法和 equals、hashCode 方法。
* @Builder 注解：
@Builder 是 Lombok 提供的注解，用于实现建造者模式。它生成一个嵌套的静态内部类 Builder，使得创建对象时可以链式调用，更加清晰和易读。使用 @Builder 注解时，您可以通过生成器模式设置实例的字段。
* @AllArgsConstructor 注解：
@AllArgsConstructor 是 Lombok 提供的注解，用于生成一个包含所有参数的构造函数。它会为类的每个字段生成构造函数参数，方便创建对象时传入所有字段的值。
* @NoArgsConstructor 注解：
@NoArgsConstructor 是 Lombok 提供的注解，用于生成一个无参数的默认构造函数。它可以在创建对象时使用，特别是当您在使用 @Builder 注解时，因为默认情况下，@Builder 注解需要一个无参构造函数。