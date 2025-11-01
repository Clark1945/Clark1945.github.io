---
title: 使用Swagger撰寫你的 Rest API document 吧!(適用Java 17)
category: Development
date: 2024-04-22 11:20:19
tags: Spring
---

作為一個後端工程師，除了基本的API開發，如何撰寫出讓人好理解的文件至關重要。一份好的文件不僅可以幫助RD了解API，也能提供給PM等部門去了解自己的產品。
<!-- more -->Swagger就是因此而生，Swagger的特色是利用Annonation撰寫文件，另外透過OpenAPI，將撰寫的內容轉成靜態網頁，文件只需要提供靜態網頁就可以讓前端人員、PM去參考API的格式與規範，相當方便。
本文以Java 17舉例。

### 安裝
在Maven的pom.xml檔案中添加以下依賴，並reload：
```
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.2.0</version>
</dependency>

```
接著補上API(已經有API的這部分可以不看)
```
@RestController
@RequestMapping("/api")
public class MemberController {
    @PutMapping("/member")
    public String login(@RequestBody HashMap reqMap) {
        System.out.println("Start login~");
        String response = memberService.login(reqMap);
        System.out.println("Response = " + response);
        return response;
    }
    
}
```

接著訪問以下路徑
http://localhost:8080/swagger-ui/index.html
就可以看到一個靜態網頁，裡面應該有所有Project的API，簡單的建立就完成了。
OpenAPI的特色是可以建立一個網頁，讓你瀏覽專案中所有在偵測範圍內的API。

你可以試著為API添加一些資訊，像是針對系統的名稱、描述或是開發者的資料等等。像這樣
```

@Configuration
public class OpenAPIConfiguration {

   @Bean
   public OpenAPI defineOpenApi() {
       Server server = new Server();
       server.setUrl("http://localhost:8080");
       server.setDescription("Development");

       Contact myContact = new Contact();
       myContact.setName("Jane Doe");
       myContact.setEmail("your.email@gmail.com");

       Info information = new Info()
               .title("Employee Management System API")
               .version("1.0")
               .description("This API exposes endpoints to manage employees.")
               .contact(myContact);
       return new OpenAPI().info(information).servers(List.of(server));
   }
}

```

如果想要為個別的API添加描述，可以參考以下程式。
```
@Tag(name = "get", description = "GET methods of Employee APIs")
@GetMapping("/employees")
public List<Employee> findAllEmployees() {
   return repository.findAll();
}

@Tag(name = "get", description = "GET methods of Employee APIs")
@GetMapping("/employees/{employeeId}")
public Employee getEmployee(@PathVariable int employeeId) {
   Employee employee = repository.findById(employeeId)
           .orElseThrow(() -> new RuntimeException("Employee id not found - " + employeeId));
   return employee;
}
```
@Tag中的name代表了分類，description代表針對這個分類的描述。

如果要建立Response的文件規範的話，可以參考以下：
```
@ApiResponses({
       @ApiResponse(responseCode = "200", content = { @Content(mediaType = "application/json",
               schema = @Schema(implementation = Employee.class)) }),
       @ApiResponse(responseCode = "404", description = "Employee not found",
               content = @Content) })
@DeleteMapping("/employees/{employeeId}")
public String deleteEmployee(@PathVariable int employeeId) {
   Employee employee = repository.findById(employeeId)
           .orElseThrow(() -> new RuntimeException("Employee id not found - " + employeeId));
   repository.delete(employee);
   return "Deleted employee with id: " + employeeId;
}
```
@ApiResponse是一個回傳格式，可以撰寫不同HTTP狀態碼的回傳內容，以及更多描述。
有了以上功能，你應該就可以為你的API寫出足夠描述的文件了。

![这是图片](/images/swagger-demo.png)

### 參考資料
1. https://medium.com/@berktorun.dev/swagger-like-a-pro-with-spring-boot-3-and-java-17-49eed0ce1d2f
2. https://bell-sw.com/blog/documenting-rest-api-with-swagger-in-spring-boot-3/E

以上