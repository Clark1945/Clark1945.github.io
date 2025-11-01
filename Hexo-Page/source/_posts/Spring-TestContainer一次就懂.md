---
title: Spring TestContainer一次就懂
category: Development
date: 2024-10-12 20:44:24
index_img: /img/Java_logo.png
 
tags:
  - Test Container
  - Java
  - SpringBoot
---
當今天你想要進行Web Application的測試時，除了程式開發，外部的環境配置也是必不可少的，假設你想要開發一個MySQL的應用程式以及一個PostgreSQL的應用程式，你就會必須要在本地安裝兩份資料庫。這是過去的作法。

然而，僅僅因為測試就得安裝一個資料庫的新環境非常麻煩，所以後來在進行開發測試時，許多開發人員都會使用Docker，先在本機下載Image之後在本地執行。如此一來就不必要煩惱各SQL的安裝了。

TestContiner依據這樣的需求誕生，整合了Docker與SpringBootTest的功能，實作也很簡單。比方說今天想要測試我的API

```java
@RestController
class StudentController {
    private final StudentRepository repo;
    StudentController(StudentRepository repo) {
        this.repo = repo;
    }

    @GetMapping("/api/student")
    List<Student> getAll() {
        return repo.findAll();
    }
}

interface StudentRepository extends JpaRepository<Student, Long> {
}
```

首先是安裝環境，安裝基本的Spring Web、取用資料的Spring JPA、資料庫的依賴(以下使用PostgreSQL舉例)、SpringBootTest、TestContainer與Rest Assured ( 提供整合測試使用的方法 )，如果你使用Maven的話，安裝應該會如下：

```yaml
/// ...
	<properties>
		<java.version>17</java.version>
		<testcontainer.version>1.19.8</testcontainer.version>
	</properties>
/// ...

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
	<groupId>org.postgresql</groupId>
	<artifactId>postgresql</artifactId>
	<scope>runtime</scope>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-testcontainers</artifactId>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>org.testcontainers</groupId>
	<artifactId>junit-jupiter</artifactId>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>org.testcontainers</groupId>
	<artifactId>postgresql</artifactId>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>io.rest-assured</groupId>
	<artifactId>rest-assured</artifactId>
	<scope>test</scope>
</dependency>
```

環境安裝後便可以著手撰寫測試程式了

```java

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class StudentControllerTest {

    @LocalServerPort
    private Integer port;

    /**
     * 利用docker Image建立一個PostgreSQL的容器，容器內的位址是預設的5432
     */
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>(
            "postgres:16-alpine"
    );

    @BeforeAll
    static void beforeAll() {
        System.out.println("PostgreSQL 開始建立");
        postgres.start();
    }

    @AfterAll
    static void afterAll() {
        postgres.stop();
        System.out.println("PostgreSQL 已終止");
    }

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    StudentRepository studentRepository;

    @BeforeEach
    void setUp() {
        RestAssured.baseURI = "http://localhost:" + port;
        studentRepository.deleteAll();
    }

    @Test
    void shouldGetAllStudents() {
        List<Student> students = List.of(
                new Student(null, "John", "john@mail.com"),
                new Student(null, "Dennis", "dennis@mail.com")
        );
        studentRepository.saveAll(students);

        given()
                .contentType(ContentType.JSON) // 請求時的Header
                .when()  
                .get("/api/students") // 請求路徑
                .then()
                .statusCode(200) // 預計接受回傳statusCode
                .body("name", everyItem(not(containsString("Clark")))); // 回傳中的每一個name都不可以包含Clark
    }
}
```

這段建構了一個PostgreSQL的Instance，並實際存入值後進行整合測試與驗證結果。我簡單講一下運作

- @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
  這段代表了SpringBootTest 會在隨機的Port執行 用來模擬外部的HTTP請求測試
- @LocalServerPort
  這段宣告了SpringBootTest的Port
- @BeforeAll是一個SpringBootTest的方法，會在啟動之後，任何@Test執行之前被執行，且只會執行一次，可以看出這一段啟動了PostgreSQL容器。
- @AfterAll在應用程式結束後把PostgreSQL容器停止。
- @DynamicPropertySource
  SpringBoot本來讀取資料庫相關位址資訊的是透過application.properties這個檔案，該註解的作用是動態配置application.properties的內容。這邊是取得容器的路徑與帳號密碼並覆蓋掉開發環境的application.properties
- @BeforeEach
  與BeforeAll不同，該方法會在每一個Test方法執行之前被執行，比方說這段定義了整合測試的路徑，並移除所有存放在Container的資料，避免資料重複存入。

以上是Spring Container 的基本使用，個人覺得這個方法可以隔離開發環境驗證API，對於撰寫整合測試來說相當理想，另外如果本地沒有Docker Image，還可以協助安裝，推薦各位試試看。

那麼今天的文章就到這邊，感恩。