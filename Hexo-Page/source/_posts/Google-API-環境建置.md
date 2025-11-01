---
title: Google API 環境建置
category: Development
date: 2024-09-15 12:04:43
index_img: /img/code.jpg
tags: Tutorial
---
> **Google Docs API 提供了讀取、寫入、更新三種API**
>

![image.png](/img/google_survey_1.png)

每個Google文件有各自專屬的documentId，如下：

<aside>
💡 `https://docs.google.com/document/d/{**DOCUMENT_ID}**/edit`

</aside>

## 環境設定&Sample程式碼：

- 開發環境 Java 17 with SpringBoot / Java 1.8 以上版本必要。
- 套件管理 maven / [Gradle 7.0 以上版本](https://gradle.org/install/)

1. 在Google Cloud 專案啟用 API，如果沒有專案請新建一個。
2. 啟用Google Docs API。

![image.png](/img/google_survey_2.png)

1. 開啟Google Cloud Console，點選「選單」 > 「API 與 服務」 > 「同意畫面」 > User Type 選擇內部後建立。


![image.png](/img/google_survey_3.png)

2. 輸入支援Email後建立。

![image.png](/img/google_survey_4.png)

1. 在Google Cloud console中，點選「選單」> 「API與服務」 > 「憑證」 > 「建立憑證」 > 「OAuth Client ID」 > 應用程式類型選擇「電腦版應用程式」、命名一個憑證名稱。

![image.png](/img/google_survey_5.png)

   1. 建立完成後，需要下載憑證(**credentials.json檔案)並放置到檔案的工作目錄**
   2. 引入需要的套件

```java
<dependency>
    <groupId>com.google.apis</groupId>
    <artifactId>google-api-services-docs</artifactId>
    <version>v1-rev20220609-2.0.0</version>
</dependency>
<dependency>
    <groupId>com.google.api-client</groupId>
    <artifactId>google-api-client</artifactId>
    <version>2.0.0</version>
</dependency>
<dependency>
    <groupId>com.google.oauth-client</groupId>
    <artifactId>google-oauth-client-jetty</artifactId>
    <version>1.34.1</version>
</dependency>
<dependency> //Optional 
    <groupId>com.google.apis</groupId>
    <artifactId>google-api-services-drive</artifactId>
    <version>v3-rev20240730-2.0.0</version>
</dependency>

// 若使用Gradle則是以下
dependencies {
    implementation 'com.google.api-client:google-api-client:2.0.0'
    implementation 'com.google.oauth-client:google-oauth-client-jetty:1.34.1'
    implementation 'com.google.apis:google-api-services-docs:v1-rev20220609-2.0.0'
}
```

## 如果要使用其他功能，會需要Enable 其他的API

# 參考文件：

https://developers.google.com/docs/api/how-tos/overview?hl=zh-tw

https://developers.google.com/docs/api/quickstart/java?hl=zh-tw

https://developers.google.com/docs/api/reference/rest?hl=zh-tw

https://developers.google.com/docs/api/reference/rest/v1/documents/get?hl=zh-tw

https://developers.google.com/docs/api/reference/rest/v1/documents/batchUpdate?hl=zh-tw

https://developers.google.com/docs/api/samples/output-json?hl=zh-tw

https://developers.google.com/docs/api/how-tos/move-text?hl=zh-tw

https://developers.google.com/drive/api/guides/create-file?hl=zh-tw