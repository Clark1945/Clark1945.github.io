---
title: SpringBoot ControllerAdvice
category: Development
date: 2024-10-14 21:03:52
index_img: /img/Java_logo.png
 
tags:
  - Java
  - SpringBoot
  - Exception
---

上一輪我們提到了，如果在Controller解析物件階段就出錯，就沒有辦法撰寫try catch去 捕捉這部分。這部分其實有很多種處理方法，以下我只介紹兩種。

第一種是比較簡易的做法，就是Spring的ErrorController，Spring預設顯示錯誤會使用ErrorController做為顯示，因此我們可以做的就是定義一個Controller類別並實作ErrorController，就能夠攔截例外，並封裝我們需要的資訊在裡面。比方說以下程式碼

```java
@RestController
public class CustomErrorController implements ErrorController {

    @RequestMapping("/error")
    public ResponseEntity<Map<String, Object>> handleError(HttpServletRequest request) {
        Map<String, Object> response = new HashMap<>();
        Integer statusCode = (Integer) request.getAttribute("javax.servlet.error.status_code");
        String errorMessage = (String) request.getAttribute("javax.servlet.error.message");

        response.put("status", statusCode);
        response.put("error", errorMessage != null ? errorMessage : "Unexpected error");

        HttpStatus status = statusCode != null ? HttpStatus.valueOf(statusCode) : HttpStatus.INTERNAL_SERVER_ERROR;

        return new ResponseEntity<>(response, status);
    }

    @Override
    public String getErrorPath() {
        return "/error";
    }
}
```

/error的程式碼中可以得知，我們利用了ResponseEntity包裝過了我們的回傳並加上了status 與 error 欄位，使回傳的資料是可以控制的。另外，getErrorPath()的作用是把錯誤的路徑轉到/error路徑下，統一做錯誤處理。

這樣的做法是簡單直觀，但是能夠套用的方法有限，如果想要更大範圍的處理錯誤，我會建議使用其他方法，也就是這次要介紹的Spring ControllerAdivce。

Spring ControllerAdvice本身已經整合在SpringBoot中，不需要額外引入，它主要提供的方法有三種。

1. Global 級別的的例外處理
2. 資料預先處理
3. 資料綁定

先介紹第一種，也是我最常用的例外處理方法：

```java
@RestControllerAdvice(assignableTypes = TestController.class)
public class GlobalExceptionHandler extends Throwable {

    @ResponseBody
    @ExceptionHandler(value = NameNotCorrectException.class)
    public String handleExceptionCollecter(HttpServletRequest req, Exception e) {
        // 处理NameNotCorrectException的逻辑
        return "nameNotCorrect"; // 返回错误页面或其他适当的响应
    }

    @ResponseBody
    @ExceptionHandler(value = IDNotCorrectException.class)
    public String handleNullPointerException(HttpServletRequest req, Exception e) {
        // 处理NullPointerException的逻辑
        return "ID NotCorrect"; // 返回错误页面或其他适当的响应
    }
    ...
```

@RestControllerAdvice宣告了類別是ControllerAdvice，assignType定義了作用的類別。在底下我宣告了兩個自定義類別，只要Controller爆出這兩種錯誤，就會來到這裡，並回復字串給使用者。

再來是第二種，資料的預先處理。以下的範例是，宣告一個Binder去針對傳入參數做預處理，使用了CustomDateEditor把字串的時間轉換成Date()型別。

```java
@InitBinder
public void initBinder(WebDataBinder binder) {
    // 把日期轉換為Date()
    SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
    dateFormat.setLenient(false);
    binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
}

@GetMapping("/profile")
    public String getUserProfile(@RequestParam("birthDate") Date birthDate) {
        // birthDate 已经通过 @InitBinder 进行转换
        return "User's birth date is: " + birthDate;
}
```

最後是第三種，在ModelAttribute中事先添加元素，使Controller可以直接使用這個屬性。

```java
@ModelAttribute
    public void addAttributes(Model model) {
        model.addAttribute("globalAttribute", "This is a global attribute");
}

@GetMapping("/profileInfo")
    public String getProductInfo(ModelMap model) {
        String globalAttribute = (String) model.get("globalAttribute");
        return "Product Info with Global Attribute: " + globalAttribute;
}
```

個人感想是，例外處理是最好用的，其他的部分儘管方便，但倒是也沒有到 非常必要，但是用來了解學習還是很不錯的，推薦給各位。

那麼今天的分享就到這邊，我們明天見，88。

參考資料：

https://www.cnblogs.com/antaia11/p/15092280.html