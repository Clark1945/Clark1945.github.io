---
title: Clean Code 筆記(1)
category: Development
date: 2024-12-26 20:59:45
index_img: /img/clean_code.jpg
tags:
  - Clean Code
  - Java
  - Programming
---
Clean Code的原則包含了多種面向，透過Clean Code的學習，可以使你全方位的檢視自己撰寫與他人撰寫的程式碼是否足夠整潔。
以下為自己整理的Clean Code學習筆記：

### 變數
好的變數命名應該要能夠名副其實，比方說：
```java
int day;
```
永遠比
```java
int expireDate;
```
要來得糟。

---
 - 避免使用 *O* *0* *I* *L* 這些容易誤導、寫錯的字元
 - 使用能唸出來的名稱，*團隊溝通會使用到*
 - 使用可被搜尋到的名字 *除錯時會使用到*
 - 介面與繼承，可使用 Student、StudentImpl作為命名
 - 命名應該直觀，符合開發人員的想像
 - 類別應以名詞命名、方法應以動詞命名

---
### 函式

設計原則
 - 讓函式只做一件事，SRP。
 - 參數的數量越少越好，方便開發者了解參數對整理輸出的影響
 - 不要帶boolean進function，違背了SRP。
 - 如果要傳入多個參數，用物件包裝會更好，至少符合相同概念。
 - 輸入與指令分離，ex:判斷條件與操作物件是兩個不同層級的概念，不要放在一起
 - 不要有副作用，避免不可預料的BUG發生
 - 以try catch 取代if 判斷，減少巢狀迴圈。

---
### 物件與資料結構

1. 抽象化設計界面
```java
FuelTankCapacityInGallons();
double getGallonsOfGasoline();
```
```java
public interface Vehicle {
    double getPercentFuelRemaining();
}
```
採用後者的設計方式可以隱藏資料的細節。

1. 資料結構的程式碼
```java
class NPC {
    void talk(Object object) {
        if (object instanceof Merchant) {
            // do something
        } else if (object instanceof Smith) {
            // do something
        } else {
            throw new Exception("error");
        }
    }
}
```
 - 在NPC類別中，要擴充一個新的類別很麻煩，必須要更動所有原先相關的程式碼，以符合函式的架構
 - 在NPC類別中，新增新的方法很容易，不需更動既有程式碼
 - 
2. 物件導向的程式碼
```java
class Merchant implements NPC {
    public talk() {
        // do something
    }
}
class Smith implements NPC {
    public talk() {
        // do something
    }
}
interface NPC {
    talk();
}
```
- 在NPC類別中，要寫一個新的類別很容易，不需更動既有程式碼
- 在NPC類別中，新增新的方法很麻煩，必須要更動每一個繼承NPC的類別

### 例外處理

- 視情況，選擇封裝Exception
- 不要傳遞Null

節錄<Clean Code> Ch1~Ch8
