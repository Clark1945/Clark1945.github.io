---
title: '[2024鐵人賽]02-Call By Value？、Call By Reference？ =與==與equals()關聯'
category: Development
date: 2024-10-02 18:59:35
index_img: /img/Java_logo.png
tags:
  - Java
---

這章主要討論的點有兩個。

1. Java Call By Value與Call By Reference都是什麼？
2. Java =、==、equals() 之間的差別以及如何使用？

在開始之前，請讓我講個大約五十字的幹話。
這次感謝XX糾我一同參加鐵人賽，若不是他，我現在肯定躺在床上耍廢玩手機…嗯？我該感謝他嗎？唔…嗯…

好，算了，正文開始。

在初學Java時，有一些Java的觀念是你必須要認識的，像是 基本型別 (Primitive Type)與參考型別(Reference Type)，這兩個的差別在，當變數被改變時，基本型別被改變的是值本身，而參考型別改變的是參考的對象。

針對Call By Value可以參考以下例子

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
}

private void replaceByHacker(int schoolId, Integer age, String name, char rank) {
    schoolId = 99999;
    age = 9999;
    name = "Stolen";
    rank = 'F';
}
Student student = new Student(1,14,"John",'B');
System.out.println(student);
replaceByHacker(student.schoolId,student.age,student.name,student.rank);
System.out.println(student);

//output
schoolId=1, age=14, name=John, rank=B
schoolId=1, age=14, name=John, rank=B
```
比方說，我們有一個學生類別，我們定義它之後，先印出學生的資料，接著執行replaceByHacker()方法，想要竄改學生資料，但最後學生的資料並沒有被改變。
原因是傳入的參數是基本型別，也就是傳入的是”值”而不是”參考”，因此實際上在方法內部被更改後，也不會影響該變數。
接著我們來看另外一個例子

```java
private void replaceByHacker2(Student student) {
  student.age = 777;
}
  System.out.println(student);
  replaceByHacker2(student);
  System.out.println(student);
  
  //output
schoolId=1, age=14, name=John, rank=B
schoolId=1, age=777, name=John, rank=B
```

在這個例子中，你可以在方法中更改student這個物件的屬性，但對物件本身也是無法被更改的，比方說你不能在方法中把student指定為null。

我自己的理解是，方法中你操作的這些參數都只是一個代理的對象，並不是值本身，所以以=取代不會生效，但如果使用物件本身的方法改變物件的狀態，因為不是使用方法內的，所以可以被改變。
另外，Java本身也無法傳入記憶體位址，因此並沒有沒有Call By Reference的概念。
至於哪些屬於基本型別，哪些屬於參考型別的，可以看看這篇文章：https://yubin551.gitbook.io/java-note/basic_java_programming/datatype/referencedatatype

綜合以上範例我們可以得出一道結論。針對 基本型別中 = 所傳遞的是值，而針對參考型別 = 所傳遞的是值的參考，為了更好地說明這件事，我們再舉個例子：

```java
int a = 10;
int k = 10;
int b = 5;
b=a;
```

這三行程式碼做了以下事：

1. JVM在記憶體中宣告一個空間放了10，並把a指定給10
2. 把k指定給10
3. JVM在記憶體中宣告一個空間放了5，並把b指定給5。
4. 把a存放的10指定給b
5. a,b等於10

這是傳遞值的例子，接下來是傳遞參考的例子。

```java
String a = new String("Some");
String b = a;

System.out.println("is a equal b ?" + (a == b)); // output: true
String c = new String("Car");
String d = new String("Car");
System.out.println("is c equal d ?" + (c == d)); // output: false
System.out.println("is c equal d ?" + (c.equals(d))); // output: true
```

前面都一樣，差別只是在第二行a將它的參考給了b，所以a b此時指向了同一個位置（記憶體位址）。

值得一提的是 c d 兩個變數，兩個都宣告為同一個”Car”，但是因為c d 兩個使用new String()方法宣告，並擁有各自的記憶體位址，且兩者並沒有指向同一個參考。所以 == 比較的結果是 false。

同時，你也可以得知 == 並不是值的比較，而是判斷兩個變數是否指向同一個位址。

所以，若要比較兩個字串實際上是否相同，則要使用.equals()。

基本上，非Primitive Type的型別我都不建議使用 == 作為判斷是否相等的依據，因為不準。

再來我們來看下一題

```java
Student student = new Student(1,14,"John",'B');
Student copyStudent = new Student(1,14,"John",'B');
System.out.println(student == copyStudent); //output: false
System.out.println(student.equals(copyStudent)); //output: false
```

從剛剛我們可以得知==比較的是記憶體位址，因此第三行為false是可理解的，但我想你一定會好奇為甚麼第四行不是true呢？這個原因是因為物件本身若沒有實作.equals()方法時預設套用Object的.equals()，也就是以下程式。

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

看到沒，他結果還是使用== 去比較，所以，這也是為很多物件在建立後必須要複寫.equals()的原因，因為不準，之所以我說不準是因為，並不是所有時候都會呈現錯誤的結果，在一些複雜的情況下，可能你並不知道兩個物件的記憶體位址是否同源。

所以在實務上必須要額外覆寫成我們需要的判斷。

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
```

以上是一些Java的觀念，下一回也會繼續談有關Java的一些特性，我們明天見吧。