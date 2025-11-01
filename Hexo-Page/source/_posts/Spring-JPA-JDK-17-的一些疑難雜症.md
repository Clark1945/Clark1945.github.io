---
title: Spring JPA(JDK 17)的一些疑難雜症
category: Development
date: 2024-04-20 12:53:47
tags: Spring 
---
### 問題一

今天在自己做Project時，啟動Project時老是出現Spring JPA 冒出的錯誤訊息:not a managed type class...
<!-- more -->
遇到時當下我有點困惑，因為之前做時從來沒看過，所以在這邊做個紀錄：
錯誤的地方如下：
```
package com.example.payment_system.orm;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;
import java.time.LocalDateTime;

@Entity
@Table(name = "payment_member_info")
public class PaymentMemberInfo {

    @Id
    @Column(name = "pmi_member_id")
    private Long pmiMeberId;
    @Column(name = "pmi_member_name",nullable = false,length = 255)
    private String pmsMemberName;
    @Column(name = "pmi_member_phone",nullable = false,length = 30)
    private String pmiMemberPhone;
    @Column(name = "pmi_member_email",nullable = false,length = 255)
    private String pmiMemberEmail;
    @Column(name = "pmi_member_pwd",nullable = false,length = 255)
    private String pmiMemberpwd;
    @Column(name = "pmi_lastlogin_time",nullable = false)
    private LocalDateTime pmiLastloginTime;

}


```
這個錯誤發生的原因來自Spring的JDK版本問題，在新版的SpringBoot與JDK中，把JavaX給替換成了Jakarta，所以只要是SpringBoot 6+ 與 JDK 17+，都是使用jakarta persist api，只要換成jakarta就解決這個問題了。

### 問題二
在新建立專案時遇到錯誤
```
Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.

Reason: Failed to determine a suitable driver class


Action:

Consider the following:
	If you want an embedded database (H2, HSQL or Derby), please put it on the classpath.
	If you have database settings to be loaded from a particular profile you may need to activate it (no profiles are currently active).


Process finished with exit code 1

```
這很明顯的就是少了設定資料庫的poperties，所以如果遇到的話要檢查application.poperties是否存在以下config：
```
spring.datasource.url=jdbc:sqlserver://localhost:1433;databaseName=payment;encrypt=false;
spring.datasource.username=sa
spring.datasource.password=17E513579
spring.datasource.driver-class-name=com.microsoft.sqlserver.jdbc.SQLServerDriver

```
從上至下分別是：連線資料庫的url、連線資料庫的帳號、密碼、以及資料庫驅動程式，根據你使用不同的資料庫要添加的稍有不同，以下以SQL Server作為舉例。

### 問題三

```
"encrypt" 屬性設定為 "true" 且 "trustServerCertificate" 屬性設為 "false"，但驅動程式無法使用安全通訊端層 (SSL) 加密建立 SQL Server 的安全連線: 錯誤: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target。 ClientConnectionId:7633a811-f5ed-4644-b413-770e31275a44
```

如果在連線時出現以下問題，代表資料庫沒有設定SSL加密的，方法有兩種，解法有調整資料庫SSL加密，以及在application.poperties設定取消加密兩種方法。如果後者可以參考以下程式。
```
spring.datasource.url=jdbc:sqlserver://localhost:1433;databaseName=payment;encrypt=false;
```

以上

### 參考資料
1. https://stackoverflow.com/questions/28664064/spring-boot-not-a-managed-type
