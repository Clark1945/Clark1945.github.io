---
title: JDBC to Hibernate to JPA
category: Development
date: 2022-10-12 20:58:10
index_img: /img/Java_logo.png
tags:
  - Java
  - Hibernate
  - Spring Data JPA
---

雖然我們前面已經快速地介紹了基本的三層式架構，(Controller、Service與Repository)，但我今天想要再詳細一點地介紹當中的Repository。Repository掌管與資料庫之間互動的細節。

Spring與資料庫溝通的方式有很多種，你也許聽過JDBC、Spring JPA、Hibernate ORM 等等，今天就會一一介紹他們。

### JDBC

Spring Framework 本身提供了jdbcTemplate讓開發者可以以執行SQL的形式向資料庫溝通，要使用前，首先你會需要引入相關的依賴。

```java
// 如果是maven
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    
// 如果是gradle

implementation 'org.springframework:spring-jdbc:6.1.12'

```

引入之後，就可以開始開發JDBC的應用程式。注意，為了避免有人忘記，我要提醒一下下，除了引入依賴外，SpringBoot的application.properties也必須要添加對應的設定，比方說如下

```yml
spring.datasource.url=jdbc:mysql://localhost:3306/school
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

從上到下分別是，資料庫的位址、資料庫的帳號與密碼、以及資料庫的datasource驅動程式。實際上，根據你使用的資料庫資訊會有些微差異

jdbcTemplate封裝了JDBC，並提供了一系列的實作，假設我想要建立一筆學生資料，JDBC會長這樣子。

```java
String sql = "INSERT INTO students (name, age, gender, class) VALUES (?, ?, ?, ?)";
jdbcTemplate.update(sql, student.getName(), student.getAge(), student.getGender(), student.getStudentClass());
```

假設我們想要查詢單筆學生的資料、或是單一個值

```java

String sql = "SELECT * FROM students WHERE id = ?";
jdbcTemplate.queryForObject(sql, new Object[]{id}, (rs, rowNum) -> {

    Student student = new Student();
    student.setId(rs.getInt("id"));
    student.setName(rs.getString("name"));
    student.setAge(rs.getInt("age"));
    student.setGender(rs.getString("gender"));
    student.setStudentClass(rs.getString("class"));
    return student;
});
    
String sql = "INSERT INTO students (name, age, gender, class) VALUES (?, ?, ?, ?)";
return jdbcTemplate.update(sql, student.getName(), student.getAge(), student.getGender(), student.getStudentClass());
```

這樣的方法，你會發現，其實就只是Java上面下SQL Statement而已，而且物件必須要被重新處理，因此後面為了克服這一點，才有ORM的出現。

ORM──全名為ObjectRelatedModel，就是以Java的物件(Object)直接對應至資料庫的表格(Model)，省去繁複的物件比對功能。Java來說，Hibernate是最常見的ORM應用。主要有以下好處

● 生產力提升（Productivity）：減少撰寫資料存取元件程式碼，聚焦在商業邏輯。

● 容易維護（Maintainability）：程式碼較少，語法上來說比純SQL更容易維護。

● 效能調校（Performance）：節省SQL語法除錯時間，多花時間調校SQL邏輯效能。

● 資料庫廠商的獨立性（Vendor independence）：免除受限於特定廠商，開發者可於輕量資料庫上開發程式，部署時再安裝至正式的大型資料庫。

在Hibernate的定義中，與資料庫的一次對話被定義為一個Session，在一個Session中可以執行多個CRUD等等操作，舉例來說，一段Hibernate的程式碼會像這樣：

```java
 Student student = new Stduent(); 
 student.setName("Johnson"); 
 student.setAge(18); 

 // 開啟Session，相當於開啟JDBC的Connection
 Session session = HibernateUtil.getSessionFactory().openSession(); 
 // Transaction表示一組會話操作
 Transaction tx= session.beginTransaction(); 
 // 將物件映射至資料庫表格中儲存
 session.save(user);
 tx.commit(); 
 session.close(); 
```

所以，Hibernate中一個與資料庫的溝通基本流程是這樣的：

1. Hibernate對SQL建立一個對話(連線) Session。
2. 在Session中建立一個Transaction。
3. 在Transaction中執行對SQL的操作 ( CRUD )
4. 將改動儲存 .save() 或是 .execute()
5. 事務提交  tx.commit() ( 也會同步觸發對資料與資料庫之間的同步 .flush() )
6. 結束這次對話，session.close()

Hibernate成功實現了以Java物件操作資料庫物件的可能性，但ORM 現在更多人使用的是SpringJPA。

Spring Data JPA 封裝了 JDBC 與 Hibernate ORM，並使用方法名稱來Mapping資料庫。所以讓程式碼更加的簡單。就好像之前的例子。

```java
@Repository
public interface StudentRepository extends CrudRepository<Student, Long> {
    Student findByName(String name);
}
```

在這個方法中，你完全不必要撰寫SQL語法，就能夠透過方法名稱去取得資料庫中對應的資料，比方說用姓名找出學生資料等等。

以上就是今天的介紹，感謝各位。