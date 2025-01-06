---
title: 一些零碎的SpringBoot小知識
date: 2025-01-05 12:05:00
category: Software
index_img: /img/SpringBoot_logo.png
tags:
  - SpringBoot
  - note
---

這邊的是提到許多一個個SpringBoot的小知識，內容沒有多到可以變成一篇文章，所以整理在這裡：

## Spring Bean 模式
Spring Bean 有Singleton、Prototype模式
 - Singleton 只有一個，每次作為Bean被呼叫時，都是使用同一個Bean。生命週期從容器啟動到第一次被請求而實體化開始，只要容器不銷毀或終止，這類的Bean就會一直存活。
 - 原型 可有多個，每次作為Bean被呼叫時，都重新初始化，定義出一個新的Bean，再請求完成後便無法引用，請求方須要自己負責後續生命週期的管理工作，包括銷毀等等。
 - Spring Bean 默認是 單例模式。

## Spring Data JPA
 - JPA的查詢語言是物件導向而非面向資料庫的，它以物件導向的自然語法構造查詢語句，可以看成是Hibernate HQL的等價物。
 - JPA定義了獨特的JPQL（Java Persistence Query Language），JPQL是EJB QL的一種擴展，它操作對象是實體，而不是關聯資料庫的表。
 - 它而且能夠支持批量更新和修改、JOIN、GROUP BY、HAVING 等通常只有 SQL 才能夠提供的高級查詢特性，甚至還能夠支持子查詢。

## Spring Bean 掃描功能
 - 為了能 Spring 能自動掃描 Bean 的存在，可以使用`@ComponentScan`，如此 Spring 預設會自動掃描同一套件以及其子套件下，是否有 Bean 元件的存在。

## RestTemplate的妙用
你可以使用RestTemplate發送request，並強行把回傳的string內容轉換成想要的Object
```java
private RespInfo SendRequest(Fundmentals funds, String body) {
        HttpHeaders headers = new HttpHeaders(); // 請求 Header
        headers.setContentType(MediaType.APPLICATION_JSON);
        headers.setAccept(Arrays.asList(MediaType.TEXT_PLAIN, MediaType.APPLICATION_JSON));
        headers.set("Ocp-Apim-Subscription-Key", funds.getApiKey());
        HttpEntity<String> httpEntity = new HttpEntity<>(body, headers);

        MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter = new MappingJackson2HttpMessageConverter();
        mappingJackson2HttpMessageConverter.setSupportedMediaTypes(Collections.singletonList(MediaType.TEXT_PLAIN)); // OP Response Header 為 text/plain RestTemplate 預設 為 application/json (允許接受text/plain)

        RestTemplate restTemplate = new RestTemplateBuilder()
                .setConnectTimeout(22000)// 連接Timeout單位毫秒
                .setReadTimeout(25000) // 讀取Timeout 單位毫秒
                .build();

        restTemplate.getMessageConverters().add(mappingJackson2HttpMessageConverter);// 調整可接收的 MediaType 不加會爆炸

        return restTemplate
                .exchange(funds.getAccessUrl(), HttpMethod.POST, httpEntity, RespInfo.class)
                .getBody();
    }
```

## String 特性
 - String 是不可變的，每次對String的操作都會產生新的String物件，建議使用StringBuffer或StringBuilder。
 - 當中StringBuffer是執行緒安全的，StringBuilder是非執行緒安全的，在單一執行緒下使用為好。

## @Controller 與 @RestController 
 - 差別就在於是否包含視圖(View)

## IOC 控制反轉。 
 - 指的是一種Spring的開發模式，不是由你去掌控流程，而是由框架去掌控流程。

## @Target
Target是針對Spring Bean的定義，用來告訴Java你的Annotation是想要標記在哪一些元素類別上，包括 TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, ANNOTATION_TYPE 等等…。在一般來說，我們最常用的應該也是有列出來的這些，像是 TYPE 其實就包括了我們常用的 Class, Interface, Enum 了。

## @Retention
Retention是針對Spring Bean的定義，主要有以下三種策略：SOURCE, CLASS (default), RUNTIME。
 - SOURCE 的意思就是你所使用的註解在 compile 後會被丟掉，如果要以最簡單的看法去理解這件事，可以看 IDE 在幫你 compile 後的 target 資料夾，會發現 `@Data` 的標記不見了，反而多了很多 Getter Setter。
 - 另外 CLASS 與 RUNTIME 的差別就在 VM 是否會記得你所寫的 Annotation，只有後者能夠透過reflection 去存取與使用元件。所以一般你若是需要進行一些業務邏輯的處理時，則都會宣告`ElementType.RUNTIME`

## @interface
如果說 interface 是一個給 implemented class 的介面，那 @interface 就是一個給 Annotation 互動的介面，因此所有的 Annotation 在宣告時都需要使用 @interface。並且在現在會有越來越多人不想寫 Comment，反而透過這種方式寫註解。

## @RepositoryRestResource
是 Spring Data REST 提供的註解，用於自動將 Spring Data Repositories 曝露為 RESTful Web API。也就是說會自動生成Entity的Rest API
```java
@RepositoryRestResource(path = "books", collectionResourceRel = "bookList", itemResourceRel = "bookItem")
public interface BookRepository extends JpaRepository<Book, Long> {
    // 可以自定義查詢方法
    List<Book> findByAuthor(String author);
}
```
將會自動生成
 - GET /books：獲取所有書籍的集合。 
 - GET /books/{id}：根據 ID 獲取單個書籍。 
 - POST /books：新增一本書。 
 - PUT /books/{id}：更新一本書（完整更新）。 
 - PATCH /books/{id}：部分更新一本書。 
 - DELETE /books/{id}：刪除一本書。

## ServerSocket
ServerSocket是Java最底層建立網路伺服器的基本概念，Server端透過Socket建立監聽，並持續等待Client的Request，使用Accept()取得Socket物件，就可以與Client端進行對話與取得資訊，接著就是IO的讀寫，最後釋放Socket資源。

![ServerSocket圖示](img/ServerSocket.png)

想自己動手做可以參考以下範例程式碼：

```java
public class Main {
    public static void main(String[] args) {
        // 定義埠號
        int serverPort = 5001;

        ServerSocket serverSocket = null;

        int receiveMsgSize = 0;

        byte[] receiveBuffer = new byte[32];

        try {
            // 建立 ServerSocket
            serverSocket = new ServerSocket(serverPort);

            while (true) {
                System.out.println("服務已經啟動，port: " + serverPort);

                Socket clientSocket = serverSocket.accept();
                SocketAddress clientAddress = clientSocket.getRemoteSocketAddress();
                System.out.println("收到客戶端連線, ip: " + clientAddress);
                InputStream in = clientSocket.getInputStream();
                OutputStream out = clientSocket.getOutputStream();

                while((receiveMsgSize = in.read(receiveBuffer)) != -1) {
                    out.write("伺服器回應: ".getBytes());
                    out.write(receiveBuffer, 0, receiveMsgSize);
                }
                // 釋放 Socket 資源
                clientSocket.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## SpringBoot Validation
```jsx
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```
在ObjectMapper進行序列化轉換時，有些方便工具
```jsx
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```
```java
    @NotEmpty(message = "Product name is undefined.")
    private String name;
    // 在 Java 物件的欄位加上 @JsonProperty 標記，可以調整序列化為 JSON 的欄位名稱。
    @JsonProperty("telephone")
    private String tel;
    
    // 在 Java 物件的欄位加上 @JsonIgnore 標記，可以在序列化為 JSON 時忽略該欄位，連 null 值都不會出現
    @JsonIgnore
    private String isbn;
    
    // 在 Java 物件的物件欄位加上 @JsonUnwrapped 標記，可以在序列化為 JSON 時，將該欄位展開。
    @JsonUnwrapped
    private Publisher publisher;
    
    @PostConstruct
    public void init() {
	    System.out.println("開始載入");
    }
    
    @PreDestroy
    public void init() {
	    System.out.println("完成，將銷毀");
    }
```

## Spring Filter
有時我們的SpringBoot應用程式會想要對API進入口之前做一些前處理，這時候不僅可以考慮Spring AOP，也可以使用Filter，以下是實現的程式碼
```java
public class LogProcessTimeFilter extends OncePerRequestFilter {
    // 紀錄API耗費時間
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws ServletException, IOException {
    long startTime = System.currentTimeMillis();
    chain.doFilter(request, response);
    long processTime = System.currentTimeMillis() - startTime;

    System.out.println(processTime + " ms");
}
}
```
實作完 Filter 的程式後，需要向 Spring 註冊，才會建立它的元件。
```java
@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean logProcessTimeFilter() {
        FilterRegistrationBean<LogProcessTimeFilter> bean = new FilterRegistrationBean<>();
        bean.setFilter(new LogProcessTimeFilter());
        bean.addUrlPatterns("/*");
        bean.setName("logProcessTimeFilter");

		    //bean.setOrder(1); //多個存在可以設置順序 
        return bean;
    }
}
```
FilterRegistrationBean 是 Spring 提供的工具，用於註冊 Filter 並配置其屬性：
 - setFilter()：設置具體的 Filter。 
 - addUrlPatterns()：指定哪些路徑會被該 Filter 攔截。 
 - setOrder()：設置該 Filter 的執行順序（可選）多filter時用來排序處理。

## Spring Exception處理
當你的API報錯回復你的自定義Exception時，可以為它加入指定的錯誤代碼
```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ItemNotFoundException extends RuntimeException {
    public ItemNotFoundException(String message) {
        super(message);
    }
}
```

為你的自定義Exception捕捉異常並處理回傳
```java
@ExceptionHandler(OperateAbsentItemsException.class)
public ResponseEntity<ExceptionResponse> handleOperateAbsentItem(OperateAbsentItemsException e) {
    Map<String, Object> info = Map.of("itemIds", e.getItemIds());
    var res = new ExceptionResponse();
    res.setType(BusinessExceptionType.OPERATE_ABSENT_ITEM);
    res.setInfo(info);

    return ResponseEntity.unprocessableEntity().body(res); // HTTP 422
}
```

另外還可以使用Global的Exception處理Annotation
```java
@RestControllerAdvice(assignableTypes = {ProductController.class, UserController.class})
public class GeneralAdvice {
    private final CustomDateEditor customDateEditor = new CustomDateEditor(CommonUtil.sdf, true);
    private final ToSearchTextEditor toSearchTextEditor = new ToSearchTextEditor();

    // 異常處理方法
    @ExceptionHandler(OperateAbsentItemsException.class)
    public ResponseEntity<ExceptionResponse> handleOperateAbsentItem(OperateAbsentItemsException e) {
        Map<String, Object> info = Map.of("itemIds", e.getItemIds());
        var res = new ExceptionResponse();
        res.setType(BusinessExceptionType.OPERATE_ABSENT_ITEM);
        res.setInfo(info);

        return ResponseEntity.unprocessableEntity().body(res);
    }

    // 綁定自定義屬性編輯器
    @InitBinder({"createdFrom", "createdTo"})
    public void bindDate(WebDataBinder binder) {
        binder.registerCustomEditor(Date.class, customDateEditor);
    }

    @InitBinder({"name", "email"})
    public void bindSearchText(WebDataBinder binder) {
        binder.registerCustomEditor(String.class, toSearchTextEditor);
    }
}
public class ToSearchTextEditor extends PropertyEditorSupport {

    @Override
    public void setAsText(String text) throws IllegalArgumentException {
        var t = CommonUtil.toSearchText(text);
        super.setValue(t);
    }
}
public static String toSearchText(String s) {
    return Optional.ofNullable(s)
            .map(String::trim)
            .map(String::toLowerCase)
            .orElse("");
}
```

## SpringMVC @ModelAttribute
1. 用來Controller上，執行home()前先執行top()把資料放到model中
```java
@ModelAttribute("top")
public Map top() {
    return pageTop.getDataMap();
}
@ReqeustMapping("/")
public String home(@Reqeust(require=true) Map<String,Object> params,Model model) {
    model.addAttriute("model",dataAssembly.homePageData(params));
    return "home";
}
```
2. 用來方法的參數中，這種用法會將 Model 中 key 為 "top" 的值注入到這個參數中
```java
@ModelAttribute("top")
public Map<String, Object> top() {
    // 假設這裡是從某個服務中獲取資料並放到 model 中
    return pageTop.getDataMap();
}
@ReqeustMapping("/")
public String home(@ModelAttribute("top") Map top,Model model) {
    JsonArray js = JSONArray.fromObject(map);
    return "test";
}
```

## Java Stream vs ParallelStream
Java Stream執行是串行的，但ParallelStream是並行的，也就是多執行緒。
Stream是對集合資料類型是順序執行的，但ParallelStream則是隨機執行的
比方說以下程式碼：
```java
        List<Integer> integerList = Lists.newArrayList();
        List<String> strList = Lists.newArrayList();

        int practicalSize = 1000000;

        for (int i = 0; i < practicalSize; i++) {
            strList.add(String.valueOf(i));
        }

        strList.parallelStream().forEach(each -> {
            integerList.add(Integer.parseInt(each));
        });

        System.out.println(strList.size()); // 1000000
        System.out.println(integerList.size()); // 因為平行化的緣故，幾乎不可能是1000000
```
應該要把資料型態改用SychronizeList或是Vector來處理多執行緒問題。
使用ParallelStream應該要考慮
1. 是否真正需要並行？
2. Task是否彼此獨立不受影響？
3. 是否有順序執行需求？

## 副作用(Side Effect) 是什麼？
在操作物件當中，無意間改變了物件的值或是型別。

## 記憶體洩漏(Memory Leak) 是什麼？
- 當一個程式被編譯成bytecode，在框架下運行的話，框架可以協助自動處理momery allocation, security, exception, GC等問題。
- 若沒有框架處理，就必須要手動處理這些問題，沒有把取用的記憶體全部釋放就會導致Memory leak。
- 一般程式來說，只要process結束後記憶體就會被虛擬機(JVM)全部釋放，不會有這個問題。
- C/C++ 因為沒有GC，所以經常出現。另外像是EventListener() 只移除了裡面的元素卻沒有移除Listener()也會造成。另外還有循環引用與存取全域變數。

## 循環引用 (Circular Import)
物件A參照到B，B又參照到A，就會造成循環引用，會造成記憶體洩漏，同時GC也會無法正常運行(Java除外)。


不時更新...

