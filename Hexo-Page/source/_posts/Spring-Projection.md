---
title: Spring Projection
date: 2024-10-19 21:16:02
index_img: /img/Java_logo.png
category: Software
tags:
  - Java
  - SpringBoot
---

我們在前幾次的單元中提到了延遲初始化的概念，但延遲初始化其實並不是解決問題的根本方法，當這個議題產生時，通常想要解決的問題就是──我們一次抓出太多不必要的資料了，

那有沒有一個方法，可以讓我們選擇只抓出部分的資料而不是全部呢？有的！去找吧！我把所有的資源都放在那裡了！那我們今天的分享就到這裡──

嘖！不行嗎？好吧。

Spring Projection 主要想要解決的問題，就是避免撈出不必要的資料，透過在Repository 回傳一個介面而不是物件的形式，比方說以下程式碼：

```java
//Part1.
@Entity
@Getter
@Setter
public class Address {
    @Id
    private Long id;
    @OneToOne
    private Person person;
    private String state;
    private String city;
    private String street;
    private String zipCode;
}

public interface AddressView {
    String getZipCode();
    @Value("#{target.city + ' ' + target.state}")
    String getCityAndState();
}

public interface AddressRepository extends Repository<Address, Long> {
    List<AddressView> getAddressByState(String state);
}

// Test
AddressView addressView = addressRepository.getAddressByState("CA").get(0);
// Hibernate: select address0_.zip_code as col_0_0_ from address address0_ where address0_.state=?
assertThat(addressView.getZipCode()).isEqualTo("90001");
assertThat(addressView.getCityAndState()).isEqualTo("Los Angeles CA");
```

在上面的範例中，於JPA的Repository中建立了getAddressByState()方法，並指定回傳物件為List的AddressView這個自訂物件，這樣的查詢好處是只會選擇到被使用的欄位，避免選擇到不需要的物件造成資源的浪費。

接著我們再看一個例子。

```java
@Entity
@Getter
@Setter
public class Person {
    @Id
    private Long id;
    private String firstName;
    private String lastName;
    @OneToOne(mappedBy = "person")
    private Address address;
    // getters and setters
}

public interface PersonView {
    String getFirstName();
    String getLastName();

    @Value("#{target.firstName + ' ' + target.lastName}")
    String getFullName();
}

public interface AddressView {
    PersonView getPerson();
}

// Test
AddressView addressView = addressRepository.getAddressByState("CA").get(0);
PersonView personView = addressView.getPerson();
assertThat(personView.getFirstName()).isEqualTo("John");
assertThat(personView.getLastName()).isEqualTo("Doe");
```

我們修改了AddressView，使他可以回傳關聯物件(Person)的資料，但回傳的形式使用了另外一個Interface接，透過PesonView介面，我就可以訪問Person類別的firstName與lastName。

此外，Projection還有提供 動態投影的效果，比方說可以在上面的Repository加入這一行。

```java
public class PersonDto {
    private String firstName;
    private String lastName;
    public PersonDto(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}

public interface PersonView {
    String getFirstName();
    String getLastName();

    @Value("#{target.firstName + ' ' + target.lastName}")
    String getFullName();
}

public interface PersonRepository extends Repository<Person, Long> {
    <T> T findByLastName(String lastName, Class<T> type);
}
//Test
Person person = personRepository.findByLastName("Doe", Person.class);
PersonView personView = personRepository.findByLastName("Doe", PersonView.class);
PersonDto personDto = personRepository.findByLastName("Doe", PersonDto.class);

assertThat(person.getFirstName()).isEqualTo("John");
assertThat(personView.getFirstName()).isEqualTo("John");
assertThat(personDto.getFirstName()).isEqualTo("John");
```

你可以使用各種類別或介面去封裝回傳的物件。當然，要確保屬性的命名與SpringDataJPA的風格一致。

那今天的部分就先到這邊吧，明天見了。

參考資料：

https://docs.spring.io/spring-data/jpa/reference/repositories/projections.html