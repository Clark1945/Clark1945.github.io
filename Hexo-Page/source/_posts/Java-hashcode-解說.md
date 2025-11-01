---
title: Java hashcode()解說
category: Development
date: 2024-10-05 20:19:36
index_img: /img/Java_logo.png
tags: 
  - Java

---

上回，我們針對Student物件覆寫了.equals()方法，通常在複寫.equals()後，很多人會建議需要一併覆寫.hashcode()方法，這是為什麼呢？

首先，我們先學習一個觀念。什麼是「Hash」？

中文叫做雜湊，Hash就是一種把資料重新處理過的一種方法，比方說雜湊演算法 ( SHA-1、MD5、SHA-256等等)，這樣針對資料產生出的一串亂數，我們常稱呼它為HashCode。舉常見的例子來說，數位簽章就是比對雙方產生出的Hash是否相同，來判斷內容有無遭到竄改。

看到這裡，你可能有些想法了，沒錯，hashCode()也在很多地方是用來比對的，比方說HashSet這個型別就會預設使用hashCode來判斷是否是同一個值。我們繼續上一篇的範例

```java
class Student {
        int schoolId;
        Integer age;
        String name;
        char rank;
        Student(int schoolId, Integer age, String name, char rank) {
            this.schoolId = schoolId;
            this.age = age;
            this.name = name;
            this.rank = rank;
        }

        public void setAge(Integer age) {
            this.age = age;
        }
        @Override
        public boolean equals(Object that) {
            return this.schoolId == ((Student) that).schoolId &&
                    Objects.equals(this.age, ((Student) that).age) &&
                    Objects.equals(this.name, ((Student) that).name) &&
                    this.rank == ((Student) that).rank;
        }
  }

Student student = new Student(1,14,"John",'B');
Student copyStudent = new Student(1,14,"John",'B');
System.out.println(student.hashCode()); //output: 162083492
System.out.println(copyStudent.hashCode()); //output: 1135300227

HashSet<Student> list = new HashSet<>();
list.add(student);
list.add(copyStudent);
System.out.println(list.size()); // 2
```

看第三、四行你可以發現，兩個Student物件的hashCode依然是不同的，這點導致了以下的HashSet中，將這兩個物件判斷為不同的物件，所以陣列長度為2。但我們知道，Set 集合這個資料結構 應該要能夠將重複的資料排除才對。

所以，我們要做的是，在複寫equals()方法後，同步複寫.hashCode()方法，程式碼如下：

```java
class Student {
        int schoolId;
        Integer age;
        String name;
        char rank;
        Student(int schoolId, Integer age, String name, char rank) {
            this.schoolId = schoolId;
            this.age = age;
            this.name = name;
            this.rank = rank;
        }

        public void setAge(Integer age) {
            this.age = age;
        }
        @Override
        public boolean equals(Object that) {
            return this.schoolId == ((Student) that).schoolId &&
                    Objects.equals(this.age, ((Student) that).age) &&
                    Objects.equals(this.name, ((Student) that).name) &&
                    this.rank == ((Student) that).rank;
        }
        @Override
        public int hashCode() {
            return Objects.hash(schoolId, age, name, rank);
        }
    }
```

這樣，兩個物件印出的hashCode就會是同一個值了，HashSet也才能夠正常運作。

接著我們來看看另外一個有關Hash的資料結構：HashMap。首先我們來看看HashMap的內部

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    ......
        static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
		......
```

可以看出，HashMap內部是一個Node結構，也就是LinkedList。內部包含了key、value、hash與下一個節點的指標。所以當一個HashMap放入一筆資料時，會先判斷當前位址有沒有存放，若已經存放了則前往下一個節點去存放值。

當使用.put()放入資料時，HashMap內部預設會根據hashCode決定要放在哪一個bucket裡面(hashCode 的來源是key值)，也會使用equals判斷物件是否內容相同，覆寫.hashCode可以提前讓Hashmap知道要放入的物件是否存在當前的結構中。

大概對物件的.equals()與.hashCode()方法就整理到這邊，細節上其實還有不少值得研究的部分，無奈本人學藝不精，只能講到這裡了。有想法的大大歡迎在底下補充，感謝各位的觀看。

參考資料：

https://codegym.cc/tw/groups/posts/tw.210.java-ha-xi-ma-

https://openhome.cc/Gossip/JavaEssence/ObjectEquality.html

https://chikuwa-tech-study.blogspot.com/2023/10/java-hashmap-structure-design.html