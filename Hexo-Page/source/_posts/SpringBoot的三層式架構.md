---
title: SpringBoot的三層式架構
category: Development
date: 2024-10-08 20:26:06
index_img: /img/Java_logo.png
 
tags: 
  - Java
  - SpringBoot
---

SpringBoot針對一個Rest Web Service，提出了三層式架構的概念，分別是Controller、Service、Repository。

以下我們簡單建立一個基本的三層式架構應用程式，並說明其中概念。

Controllers

Controller是API的進入口，主要的功能是傳入參數的檢測與一些資料的前處理。舉例來說，一個最基本的Cotnroller會像這樣。

```jsx
@RestController
public class StudentController {

    @Autowired
    private StudentService studentService;

    @GetMapping("/getOneStudent")
    public Student getOneStudent(@RequestParam("name") String name) {
        if (!name.matches("/s+")){
            System.out.println(name + "非姓名欄位");
            return null;
        }
        return studentService.getStudent(name);
    }
}
```

@RestController 是Spring的註解，代表這個class檔作為一個REST API 的 Controller被定義，在這裡可以撰寫各種endPoint(API入口)。

@Autowired 代表了交給Spring 管理的 Bean，並已全域模式注入到這個檔案中，這部分之後我會之後再說明，現在只需要了解它是一個Service，可以幫我們處理麻煩的事情就好。

@GetMapping(”/getOneStudent”) 代表了一個endPoint，路徑為 /getOneStudent。如果你啟動的話，他預設會連線到http://localhost:8080/getOneMerchant這個路徑下。

EndPoint內部接收了一個參數，如果當初URL帶入的查詢條件有包含name的話，這裡就會被載入進去，他接著會利用正規表達式判斷String是否符合純英文不包含數字，如果判斷失敗的話將會回傳Null，否則呼叫StudentService.getStudent()方法取得Student資料。

看完了Controller部分，我們接著來了解實際上API是怎麼取得我們所需要的資料的。

Service是程式主要的功能區域，處理一個API最為複雜的商業邏輯部分。

```jsx
@Service
public class StudentService {

    @Autowired
    private StudentRepository studentRepository;

    public Student getStudent(String name) {
        return studentRepository.findByName(name);
    }
}
```

@Service 宣告了這個.class是一個Service(廢話)，當中@Autowired引入了StudentRepository。

getStudent()的方法很簡單，就是又呼叫了一層，取得Student的資料。

你也許現在會很困惑為什麼要搞得這麼麻煩，但我必須說，當系統功能逐漸複雜，規模逐漸增大後，良好的把功能切分出來是很重要的，Debug時你就會很感謝的。

最後是Repository的部分，Repository掌管與資料庫的互動，主要是把資料庫的部分從Service層獨立出來。實際的程式碼會像是以下：

```jsx
@Repository
public interface StudentRepository extends CrudRepository<Student, Long> {
    Student findByName(String name);
}
```

@Repository的功用…額我就不說了，你們猜得到的。這個StudentRepository的介面繼承了CrudRepository這個類別，這意味著它可以使用以下方法

```jsx
public interface CrudRepository<T, ID> extends Repository<T, ID> {
    <S extends T> S save(S entity);
    <S extends T> Iterable<S> saveAll(Iterable<S> entities);
    Optional<T> findById(ID id);
    boolean existsById(ID id);
    Iterable<T> findAll();
    Iterable<T> findAllById(Iterable<ID> ids);
    long count();
    void deleteById(ID id);
    void delete(T entity);
    void deleteAllById(Iterable<? extends ID> ids);
    void deleteAll(Iterable<? extends T> entities);
    void deleteAll();
}
```

上面我就不一一細說，總之就是符合一般日常的對資料庫的異動功能。像上面的功能就是利用Name從資料庫中找出Student的資料，並回傳到Service，再回傳到Controller，最後回覆給API的使用者。以上就是一個最基本的三層式架構的應用程式。

今天的部分就到此為止了，明天見吧。