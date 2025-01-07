---
title: Spring @Lombok介紹
date: 2024-10-11 20:39:02
index_img: img/Java_logo.png
category: Software
tags:
  - Java
  - SpringBoot
---
今天沒什麼時間，決定要來介紹輕鬆又簡單的Lombok，Lombok是一套Java的函式庫，內部包含了許多功能，包括：

- @Setter
- @Getter
- @Slf4j - 撰寫log的工具，經常以log開頭( ex.log.info())
- @ToString - 撰寫物件的toString() 方法，若不複寫只會印出位址資訊
- @NoArgsConstructor 建立無參數建構式
- @AllArgsConstructor 建立所有參數的建構式
- @RequireConstructor 建立包含所有final參數的建構式
- @EqualsAndHash 自動複寫equals與hash方法
- @Builder 遵循builerPattern 來建立物件
- @Data >> getter + setter + toString + equals + hashCode + requireArgsConstructor

以下是一個範例：

```jsx

@Slf4j
@NoArgsConstructor
@AllArgsConstructor
@Builder
@Data
class Student {
   private final int id;
   private final String name;
   private String status;
   private String nickname;
}
```

這是使用了Lombok，讓我們看看沒使用Lombok的版本 ( 同功能 )

```jsx
public class Student {
    private static final Logger log = LoggerFactory.getLogger(Student.class);

    private final int id;
    private final String name;
    private String status;
    private String nickname;

    // No-args constructor
    public Student() {
        this.id = 0;
        this.name = null;
        this.status = null;
        this.nickname = null;
    }

    // All-args constructor
    public Student(int id, String name, String status, String nickname) {
        this.id = id;
        this.name = name;
        this.status = status;
        this.nickname = nickname;
    }

    // Getters
    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getStatus() {
        return status;
    }

    public String getNickname() {
        return nickname;
    }

    // Setters
    public void setStatus(String status) {
        this.status = status;
    }

    public void setNickname(String nickname) {
        this.nickname = nickname;
    }

    // toString
    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", status='" + status + '\'' +
                ", nickname='" + nickname + '\'' +
                '}';
    }

    // equals and hashCode
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Student student = (Student) o;
        return id == student.id && 
               Objects.equals(name, student.name) &&
               Objects.equals(status, student.status) &&
               Objects.equals(nickname, student.nickname);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name, status, nickname);
    }
```

總之，Lombok利用註解來幫你省去一大筆要慢慢手寫物件建立與讀取方法的時間，我過去也很喜歡使用Lombok，但是他也未必都是好處。以下統整出它的缺點：

1. 高度侵入性。 Lombok 即使如此實用，他也沒有被加入springBoot裡面，而是一個第三方的套件，而且只要在專案使用了，就會建議一律使用，避免同時存在兩種方法不好管理。
2. 管理不易，通常來說，我們可以透過點選function來找出他的usage，但lombok目前不行，也就是說如果使用Lombok，會導致你無法確認到底有多少地方使用它。
3. 容易導致壞習慣，當想到建立物件時，想都沒想就放一個@Data上去，並不是一個好習慣，但使用Lombok常常會導致我們這麼做。
4. 外部套件依賴有時會導致意想不到的問題發生，之前我有一次開發新專案時，引入了Lombok，結果因為套件版本異常，getter與setter出來的值全部變成null，但沒有任何exception回報，我找了一下子才發現是lombok的問題。

以上就是我今天偷懶的分享，我們明天見，掰掰