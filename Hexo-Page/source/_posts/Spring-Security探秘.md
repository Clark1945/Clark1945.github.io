---
title: Spring Security探秘
date: 2025-01-05 18:56:10
category: Software
index_img: /img/SpringBoot_logo.png
tags:
  - Java
  - Spring Security
  - JWT
---

### SpringBoot引入Spring Security做了什麼？
Spring Boot 引入 Spring Security 後的預設行為：
 - 自動保護所有 endpoint
 - 提供預設帳號 (user) 與隨機密碼
 - 提供登入/登出表單
 - 未認證的請求會導向登入頁或回傳 401
 - 提供多種安全防護機制:
   - CSRF 防護
   - Session Fixation 防護
   - 強制 HTTPS (HSTS)
   - 防止內容類型嗅探
   - 緩存控制
   - 點擊劫持防護 (X-Frame-Options)
 - 使用 [Strict-Transport-Security](https://docs.spring.io/spring-security/reference/servlet/exploits/headers.html#servlet-headers-hsts)確保是HTTPS，HSTS用於強制瀏覽器僅通過 HTTPS（而非 HTTP）訪問該網站
 - 使用 [X-Content-Type-Options](https://docs.spring.io/spring-security/reference/servlet/exploits/headers.html#servlet-headers-content-type-options) 以緩解 [sniffing attacks](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html#x-content-type-options)
 - 使用 [Cache Control headers](https://docs.spring.io/spring-security/reference/servlet/exploits/headers.html#servlet-headers-cache-control) that 保護授權後的資源，Cache-Control 和 Pragma 標頭用於指示瀏覽器或代理不要快取授權後的敏感資源，防止未經授權的用戶訪問緩存。
 - 使用 [X-Frame-Options](https://docs.spring.io/spring-security/reference/servlet/exploits/headers.html#servlet-headers-frame-options) 緩解 [Clickjacking](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html#x-frame-options)，頭防止網頁內容被嵌入到 `<iframe>` 中，避免 Clickjacking 攻擊
 - 與`HttpServletReqeust`的認證方法整合
 - 發布成功認證與失敗認證訊息 (`AuthenticationSuccessHandler` 與 `AuthenticationFailureHandler` )
#### Session Fixation攻擊模式介紹
 1. **目標**：
    - 攻擊者試圖固定用戶的會話ID，然後在用戶登錄後劫持會話。
 2. **方法**：
    - 攻擊者需要事先將特定的會話ID注入到用戶的會話中。
 3. **SpringSecurity防禦措施**：
    - 在用戶登錄後重新生成新的會話ID，使用安全屬性的cookie設置（例如HttpOnly、Secure）。
#### CSRF攻擊模式介紹
 1. **目標**：
    - 攻擊者利用用戶已經登錄的狀態，在未經用戶同意的情況下執行操作。
 2. **方法**：
    - 攻擊者誘導用戶訪問惡意網站，然後利用用戶的瀏覽器發送請求。
 3. **SpringSecurity防禦措施**：
    - 使用CSRF令牌（token），驗證請求來源的Referer頭，確保關鍵操作需要用戶確認（例如多因素認證）。

### Spring Security基本設定步驟：

#### 認證流程的重要元件：
- AuthenticationManager: 負責認證流程
- UserDetailsService: 載入使用者資訊
- PasswordEncoder: 密碼編碼與驗證
- SecurityContextHolder: 儲存認證信息
- SecurityContext: 保存當前使用者的認證信息

基本範例：
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    // 1. 配置使用者資訊
    @Bean
    public InMemoryUserDetailsManager userDetailsManager() {
        // 建立使用者帳號、密碼、權限
       UserDetails user1 = User
               .withUsername("user1")
               .password("{noop}111")
               .authorities("STUDENT")
               .build();
       UserDetails user2 = User
               .withUsername("user2")
               .password("{noop}222")
               .authorities("TEACHER")
               .build();
       UserDetails user3 = User
               .withUsername("user3")
               .password("{bcrypt}333")
               .authorities("ADMIN", "TEACHER")
               .build();
       return new InMemoryUserDetailsManager(List.of(user1, user2, user3));
    }

    // 2. 配置安全過濾器鏈
    @Bean 
    public SecurityFilterChain filterChain(HttpSecurity http) {
        return http
            .authorizeRequests() // 設定訪問權限
            .formLogin()        // 啟用表單登入
            .httpBasic()       // 啟用 HTTP Basic 認證
            .build();
    }
    // OR
   @Bean
   public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception {
      return httpSecurity
              .formLogin(Customizer.withDefaults())
              .authorizeHttpRequests(requests -> requests
                      .requestMatchers(HttpMethod.GET, "/register").permitAll()
                      .requestMatchers(HttpMethod.GET, "/selected-courses").hasAuthority("STUDENT")
                      .requestMatchers(HttpMethod.GET, "/course-feedback").hasAnyAuthority("TEACHER", "ADMIN")
                      .requestMatchers(HttpMethod.GET, "/members").hasAuthority("ADMIN")
                      .anyRequest().authenticated()
              )
              .csrf(csrf -> csrf.disable())
              .build();
   }
}
```
這裡註冊了一個SecurityFilterChain，定義它們要開放給具有哪些權限的人存取。
Spring Security 會將 HttpSecurity 物件注入到建立元件的方法中。透過該物件的一系列方法呼叫，我們能站在安全管理的角度，自定義 request 到達後端時的應對方式。
一開始的 formLogin 方法，是啟用先前的登入畫面，便於我們繼續進行測試。
接下來的 authorizeHttpRequests 方法，其用途是設定要如何進行授權。定義時，需提供「API」與「授權規則」這兩個部份。
呼叫 requestMatchers 方法，可傳入 API 路徑與 HTTP 方法；呼叫 anyRequests 方法，代表要對「其餘」的 API 做設定。這是有先後順序之分的，就像 Java 語言的「if → else if → else」，是由上而下逐一判斷。
提供完 API 後，接著要定義授權規則。以下舉例幾個可用的方法：

| 方法名稱 | 意義 |
| --- | --- |
| permitAll | 不必登入就能存取。 |
| hasAuthority | 需具備某一個權限才能存取。 |
| hasAnyAuthority | 只要具備任一個權限就能存取。 |
| authenticated | 需登入才能存取。 |

看 `InMemoryUserDetailsManager` 類別的原始碼，會發現它頂層實作了 `UserDetailsService` 介面。該介面是 Spring Security 用來進行認證的重要元件。它提供一個叫做 `loadUserByUsername` 的方法，用途是接收帳號的值，並回傳內含使用者資料的 `UserDetails` 介面物件。繼續追蹤 `InMemoryUserDetailsManager` 的原始碼，會發現它是用 `Map` 資料結構來儲存使用者。
啟動程式時，Spring Security 會檢查專案中是否有 `UserDetailsService` 元件。若無，則自動建立一個 `InMemoryUserDetailsManager` 元件，並包含一個帳號為「user」、密碼為隨機（可在 console 找到）的使用者。
權限資料是透過 `GrantedAuthority` 介面來傳遞。Spring Security 內建了一個叫做 `SimpleGrantedAuthority` 的實作類別。

認證方式還有許多變形，例如：
#### HTTP_BASIC (Not Http Form)
```java
// OR
   @Bean
   public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception {
      return httpSecurity
              .authorizeHttpRequests(requests -> requests
                      .requestMatchers(HttpMethod.GET, "/home").permitAll()
                      .requestMatchers(HttpMethod.POST, "/select-course").hasAuthority("STUDENT")
                      .requestMatchers(HttpMethod.PUT, "/courses").hasAuthority("TEACHER")
                      .anyRequest().authenticated()
              )
              .csrf(csrf -> csrf.disable())
              .httpBasic(Customizer.withDefaults())
              .build();
   }
```
Spring Security 支援一種叫做「HTTP Basic」的認證方式。其做法是在「Authorization」這個 request header 攜帶帳號與密碼，就像在登入畫面輸入帳密一樣。而後端會在每次接收到 request 時，就先進行認證。

透過上面內容，我們已經知道Spring Security提供了甚麼安裝防護，以及註冊User的過程。那假設我們想要在API取得有關User的資料的話，可以這麼做：
```java
    @GetMapping("/home")
    public String home() {
        SecurityContext context = SecurityContextHolder.getContext();
        Authentication auth = context.getAuthentication();
        Object principal = auth.getPrincipal(); // 取得Spring Security User，若無顯示anonymousUser

        if ("anonymousUser".equals(principal)) {
            return "你尚未經過身份認證";
        }
        
        UserDetails userDetails = (UserDetails) principal;
        return String.format(
                "你好，%s，你的權限是：%s",
                userDetails.getUsername(),
                userDetails.getAuthorities()
        );
  }
```
Spring Security 是透過 Java Servlet 的「Filter」，才能在 request 到達 Controller 前進行認證與授權。其中 `BasicAuthenticationFilter` 是專門處理 HTTP Basic 認證的 Filter。
Filter 接收到 request 後，首先 `AuthenticationConverter` 會從 `HttpServletRequest` 取出「Authorization」這個 header 的值，進行 Base64 解碼。解析出帳密後，封裝成 `Authentication` 物件。
而 `AuthenticationConverter` 介面的實作類別是 `BasicAuthenticationConverter`，它會回傳 `Authentication` 介面的實作類別 `UsernamePasswordAuthenticationToken`。解析出的帳號，會放在 `UsernamePasswordAuthenticationToken` 物件的 `principal` 欄位，而密碼放在 `credentials` 欄位。
![AuthenticationConverter](/img/spring_security_1.jpg)

### AuthenticationManager 進行認證
將帳號與密碼的值封裝成 `Authentication` 介面的物件後，接下來 `AuthenticationManager` 會進行身份認證，並回傳另一個新的 `Authentication` 物件。兩個物件的差別在於，新物件會包含認證後的結果，即使用者資料。
另外，這個 `AuthenticationManager` 介面，有個實作類別叫做 `ProviderManager`，它擁有多個 `AuthenticationProvider` 介面的物件，讀者可理解成「認證功能的提供者」。
![AuthenticationConverter](/img/spring_security_2.jpg)
在 HTTP Basic 認證的情況下，會由 `DaoAuthenticationProvider` 提供認證功能。它接收 `Authentication` 物件後，使用了 `UserDetailsService` 與 `PasswordEncoder` 進行帳號與密碼的認證。

認證成功後，`DaoAuthenticationProvider` 會將 `UserDetailsService` 回傳的 `UserDetails` 放入 `UsernamePasswordAuthenticationToken` 物件的 `principal` 欄位中，並以 `Authentication` 介面回傳。

### SecurityContextHolderStrategy 管理認證資訊
Spring Security 會將認證後的使用者資料儲存於記憶體，讓我們在程式邏輯中能使用。就像在 Controller 的範例程式取出 `UserDetails` 物件那樣。
負責管理認證資訊的是 `SecurityContextHolder` 中的 `SecurityContextHolderStrategy`。後者是一個介面，代表管理的「策略」，Spring Security 內建數種實作好的策略。
而前者是一個類別，會在啟動程式時選擇其中一種管理策略。除此之外，就只是對外提供存取認證資訊的方法罷了。
![AuthenticationConverter](/img/spring_security_3.jpg)
預設的管理策略是 `ThreadLocalSecurityContextHolderStrategy`，它採用了「ThreadLocal」的技術，讓每個執行緒只能取得屬於自己的資料。因此，即便 Spring Boot 同時處理許多 request，但並不會意外地取得他人的認證資訊。
回到 Filter 的邏輯。接下來會產生一個 `SecurityContext` 介面的物件，將認證後的 `Authentication` 物件封裝起來。
最後將 `SecurityContext` 存回 `SecurityContextHolderStrategy`，就完成 HTTP Basic 認證的流程。

認證方法有很多種，除了HTTP_BASIC，也可以考慮安全性更高的JWT Token 或是 Oauth。以下僅介紹JWT
JWT是一種特殊的Token格式，主要由
- Header (類型、演算法)
- Payload (聲明資訊)
- Signature (簽名)
三個部分形成，實務上則有兩種形式
- Access Token 用戶存取 API 時會攜帶於 request header，證明自己的身份。為了安全性，其有效期限較短，例如 1 小時。因此若外洩，則盜用者也無法使用太久的時間
- Refresh Token 在換發新的 Access Token 時提供。有效期限較長，例如 7 天。

透過 Refresh Token，就能在不重新登入的條件下，取得新的 Access Token，有助於使用者體驗。等到 Refresh Token 到期，就真的要重新登入，取得以上 2 種 Token 了。




