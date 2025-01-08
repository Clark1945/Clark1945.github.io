---
title: Spring MapStruct
date: 2024-10-21 21:50:57
index_img: img/Java_logo.png
category: Software
tags:
  - MapStruct
  - Java
---

在Java的歷史中，我們為了更責任分明的處理資料，誕生出了許多物件，如對應資料庫的Persistent Object(PO)、用來傳輸溝通使用的Data Transfer Object(DTO)、以及用來顯示的Value Object(VO)，等等…

這樣針對同一個(廣義來說)物件，根據不同使用情境進行封裝，很好展現出Java的特色(隱藏不必要的細節)，但缺點是管理這些物件變得比較麻煩，我們會經常將一個物件轉換為另外一個物件，比方說我們把DTO轉為PO並存入資料庫中。那有沒有一個方法可以幫助我們處理物件轉換。

有，就今天要介紹的Mapstruct。MapStruct比起傳統常看到的ObjectMapper，多了更多的功能可以使用，以符合現在更複雜的場景。

要使用Mapstruct，你首先要導入依賴。

```jsx
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.6.0.Beta1</version>
</dependency>
```

假設我們現在有一隻註冊API，他負責將傳來的會員資料，判斷是否可以被註冊，並將資料存入資料庫中，我們主要要將Member轉為MemberPO，規格如下。

```java
public class Member {

    private String name;
    @Schema(description = "Unique identifier of the User", example = "Clark")
    private String account;
    private String password;
    private String email;
    private String phone;
    private String address;
    private LocalDate birthday;
}

public class MemberPO {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(nullable = false)
    private String name;
    @Column(nullable = false, unique = true)
    private String account;
    @Column(nullable = false)
    private String password;
    @Column(columnDefinition = "INET")
    private InetAddress ip;
    @Column(name = "wallet_id")
    private Long walletId;
    @Column(nullable = false, unique = true)
    private String email;
    @Column(name = "phone_number")
    private String phoneNumber;
    @Column(columnDefinition = "TEXT")
    private String address;
    // Update by SQL Trigger
    @Column(name = "created_at", nullable = false, updatable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private LocalDateTime createdAt;
    // Update by SQL Trigger
    @Column(name = "updated_at", nullable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private LocalDateTime updatedAt;
    @Column(nullable = false)
    private String status = MemberStatus.ACTIVE.name();
    @Column(nullable = false)
    private String role = MemberRole.USER.name();
    @Column
    private LocalDate birthdate;
    @Column(name = "profile_picture_url")
    private String profilePictureUrl;
    @Column(name = "last_login")
    private LocalDateTime lastLogin;
    @Column(name = "login_attempts", nullable = false)
    private int loginAttempts = 0;
    @Column(name = "security_question")
    private String securityQuestion;
    @Column(name = "security_answer")
    private String securityAnswer;
    @Column(name = "two_factor_enabled", nullable = false)
    private boolean twoFactorEnabled = false;
}
```

這兩個物件差異很大，欄位名稱，資料數量等等都不同，要怎麼做呢？

首先，我們要先定義出一個MapStruct來處理

```java
@Mapper
public interface MemberMapper {
		// 實例化Mapstruct
    MemberMapper INSTANCE = Mappers.getMapper(MemberMapper.class);
}
```

接著我們可以加入我們的轉換方法

```java
@Mapper
public interface MemberMapper {
		// 實例化Mapstruct
    MemberMapper INSTANCE = Mappers.getMapper(MemberMapper.class);
    
    MemberPO memberToMemberPo(Member member);
    Member memberPOToMember(MemberPO memberPO);
}
// TEST
MemberPO memberPO = MemberMapper.INSTANCE.memberToMemberPo(member);
Member member = MemberMapper.INSTANCE.memberPOToMember(memberPO);
```

我建立了兩個方法，以應對Member轉MemberPO與MemberPO轉回Member物件的情境。

基本上只要這樣兩個物件就可以進行互相轉換了。但我們剛剛有提到，兩者物件的欄位名稱並不相同，針對這部分，可以加上@Mapping進行Mapping，如果邏輯比較特殊，也可以自己掛一個轉換器，我們在加上這部分吧。

```java
@Mapping(source = "phone",target = "phoneNumber")
@Mapping(source = "birthday",target = "birthdate")
@Mapping(source = "ip", target = "ip", qualifiedByName = "stringToInetAddress")
MemberPO memberToMemberPo(Member member);

@Named("stringToInetAddress")
    default InetAddress stringToInetAddress(String ip) {
        try {
            return InetAddress.getByName(ip);
        } catch (UnknownHostException e) {
            throw new RuntimeException("Invalid IP address: " + ip, e);
        }
    }

```

source 代表來源，也就是傳入的物件屬性，target則代表回傳的物件屬性，透過這樣的方式去映射兩個物件，qualifiedByName則可以掛一個轉換器，以下把String的 IP轉換為 InetAddress物件。

如果你想要針對一些欄位做出預設值？Mapstruct也支援這個功能，比方說在Member轉換為MemberPO後自動配置角色權限為一般使用者(USER)、並預設帳號狀態為啟用狀態。

```java
/**
 * 轉型後設定預設值
 * @param memberPO 轉換對象
 * @param member 轉換來源
 */
@AfterMapping
default void setDefaultValues(@MappingTarget MemberPO memberPO, Member member) {
  if (memberPO.getRole() == null) {
      memberPO.setRole(MemberRole.USER.name());
  }
  if (memberPO.getStatus() == null) {
      memberPO.setStatus(MemberStatus.ACTIVE.name());
  }
}
```

基本上，我常用的主要操作就是以上這些了。Mapstruct非常靈活，提供了額外寫一個方法進行手動轉換以外的方法，有興趣的朋友可以試試看這個工具，那今天的部分就到這裡了，就明天見吧。