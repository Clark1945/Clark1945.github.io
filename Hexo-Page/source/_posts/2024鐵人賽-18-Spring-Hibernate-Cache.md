---
title: '[2024鐵人賽]18-Spring Hibernate Cache'
category: Development
date: 2024-10-18 20:31:26
index_img: /img/Java_logo.png
tags:
  - Java
---

在進行與資料庫溝通時，使用ORM是常見的作法，因此在開發上就少不了要面對Hibernate，儘管可能使用的技術是Spring JPA、JPQL等等，但Hibernate的持久化、快取等等都是同樣的。

之前在Transactional篇我們提到，在Session中與資料庫物件維持對應關係的狀態被稱作Persistent(持久)狀態，而在這個狀態中，為了減少對資料庫的負擔，Hibernate會將取出的資料放入在快取中，避免下一次索取相同資料時重複對資料庫發動不必要的請求而加重資料庫的負擔。

而這接下來就會有一個議題，Hibernate如何保持快取資料是正確的？

Hibernate會針對在Persistent狀態的物件建立一個複製品，用來比對是否物件遭到更改，也就是drity check的功能，來確保資料的正確。另外，只要在Session之中，一個持久化的物件就會對應到一個複製品，也就是存在兩個物件。

當今天在一個Session中資料非常大量存在時，因為這些資源在Session中不會被自動釋放，因此有可能會出現OOM (Out Of Memory) 的狀況。所以實務上為了避免這件事情發生，建議要固定將資料.flush()進資料庫並清除快取。

已預設情況來說，flush()的時機是在@Transactional結束後自動.flush()與，如果想要根據特殊情況手動觸發flush，以下舉例，假設我要每500筆資料強制寫入資料庫並清除快取。

```jsx
    @PersistenceContext
    private EntityManager entityManager; // 使用EntityMangere管理實體

    
    @Transactional
    public void batchInsertStudents(List<Student> students) {
        final int batchSize = 500;

        for (int i = 0; i < students.size(); i++) {
            // 物件持久化
            entityManager.persist(students.get(i));
            // 每500筆就flush 和 clear
            if (i % batchSize == 0 && i > 0) {
                entityManager.flush();  // 將資料更改同步到資料庫
                entityManager.clear();  // 清除Session快取
            }
        }
        entityManager.flush();
        entityManager.clear();
    }
```

另外Hibernate的快取還有分為兩種，Session Level與SessionFactory Level，前者是只快取在Session中，另外一個則是可以在多個Session中被共同使用，更常見的說法我們稱呼它們為 一級快取(Session Cache) 與 二級快取(SessionFactory Session)。Session Level的快取會在Session結束後被清除，二級快取則只會在資料被更新時刪除舊的快取，因此有時二級快取的管理會有一些潛在問題發生。更多可以參考底下的連結。

今天的分享就先到這邊了，我們明天見了。