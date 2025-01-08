---
title: Spring Reflection
date: 2024-10-20 21:46:52
category: Software
tags:
  - SpringBoot
  - Reflection
---

Spring Reflection──反射，是不少新手開發者會感到不熟悉的部分，而他的概念確實也比較複雜，本次就是好好講個Reflection，以及Reflection怎麼使用。

在幾篇前我們有提到過Java的生命週期是 透過javac把Java程式碼轉換成位元碼，再用JVM去讀取ByteCode並把ByteCode轉換成個平台都可以支援的機器碼。而程式碼的類別都是在編譯階段就已經被載入，並固定下來了。

而Reflection最大的特點就是允許程式碼在執行階段才載入類別。這麼做有甚麼好處呢？就是可以動態化處理問題，假設你今天有一個用來處理Excel資料的物件XLSWorker，他擁有一個readFile方法，來讀取xls的資料。今天你開發了一個新的功能XLSXWorker，而你想要測試這兩個物件的同一個readFile()方法時。這時，你對程式的關注點就不會是 物件本身，而是你希望的方法是否可以得到符合期待的輸出。程式碼的部分以例子來說，會像這樣。

```java
// 沒有使用Reflection的寫法
XLSService xlsService = new XLSService();
xlsService.readFile(xlsFile);

//反射方法
Class<?> clazz = Class.forName("XLSService"); // 替换为你的 XLSService 类的全限定名
Object xlsServiceInstance = clazz.getDeclaredConstructor().newInstance(); // 使用默认构造函数创建实例

// 2. 通过反射获取 readFile 方法
Method readFileMethod = clazz.getMethod("readFile", String.class); // 假设 readFile 方法接受一个 String 类型的参数

// 3. 调用 readFile 方法
readFileMethod.invoke(xlsServiceInstance, xlsFile); // 传入 xlsFile 作为参数
```

除此之外，它還可以掃描類別以下擁有的方法，甚至可以從外部訪問類別的私有方法，這部分使她有了安全性上的疑慮，比方說以下的程式就可以掃描出類別下有哪些方法。

```java
Class<Person> personClass = Person.class;

Field[] fields = personClass.getDeclaredFields();
for (Field i : fields) {
    System.out.println("Field = "+i.getName()); // 印出類別中所有的屬性
}
Method[] methods = personClass.getDeclaredMethods(); //宣告的方法
for (Method m : methods) {
    System.out.println("Method = " + m.getName()); // 印出類別中所有的方法
}
```

假設你是一名不懷好意的攻擊者，你就可以使用Reflection去讀取類別的所有屬性與方法藉而得知如何竊取或竄改資料，像是以下程式碼，就可以任意set 物件的值，你只需要知道類別的方法名稱就可以了。

```java
private <T> T setObjectValue(Object object,String colName,Class colNameType,Object colVal) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
    Class<?> aClass = object.getClass(); // 取得類別
    Method method = aClass.getDeclaredMethod("set"+StringUtils.capitalize(colName),colNameType); // 取得指定值的set方法
    method.invoke(object,colVal);
    return (T) object;
}
// TEST 
Billing billing = new Billing();
Billing billingAfterSetId = setObjectValue(billing,"id",int.class,id);
```

小弟過去的使用場景主要還是在撰寫測試時使用比較常見，有時當程式碼規模很大，你需要去測試某個你撰寫出來的private方法，但你又不想要修改方法的作用域時，Reflection就是一個比較好的選擇，另外Reflection也適合大量測試。

而Reflection的缺點也很明顯。一，Reflection的寫法會讓程式碼的維護難度直線上升，少了Compiler自動幫你處理型別判斷後，就連大小寫都會導致你開發上功能異常。二，安全性，因為可以透過Reflection方法直接訪問類別的內部結構與自動觸發方法，導致她很危險。三，在執行期間訪問類別會無可避免地導致效能低落。

以上就是我的一些整理，那麼明天見了。