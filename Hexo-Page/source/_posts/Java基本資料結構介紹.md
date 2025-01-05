---
title: Java基本資料結構介紹
date: 2024-11-02 09:24:22
category: Software
tags: 
  - Data Structure
  - Java
---
這篇文章主要在介紹Java的各種集合資料結構，久久來複習一下基本功。

## Java 集合框架的主要類型與差異

Java 的集合框架提供了三大主要類型：`List`、`Set` 和 `Map`，每種類型都有不同的特性與適用場景。
### List
特性：有序、允許重複元素、以索引存取

| 類型 | 特性 | 適用場景 |
| --- | --- | --- |
| **ArrayList** | - 動態大小，基於陣列實現<br/>- 隨機讀取快 (`O(1)`)<br/>- 插入、刪除效率低 (`O(n)`)<br/>- 非同步<br/>- 容量自動擴充（1.5 倍）<br/>- 預設初始容量為 10 | - 資料讀取頻繁、變動少的場合<br/>- 需要快速隨機存取<br/>- 主要在尾部新增/刪除元素 |
| **LinkedList** | - 雙向鏈結串列<br/>- 插入、刪除快 (`O(1)`)<br/>- 讀取效率低 (`O(n)`)<br/>- 非同步<br/>- 實現了 Deque 介面，可當作佇列或堆疊使用 | - 資料頻繁插入、刪除的情況<br/>- 需要從兩端操作集合的場景<br/>- 不需要隨機存取 |
| **Vector** | - 同步版的 ArrayList<br/>- 執行緒安全但效率較低<br/>- 容量自動擴充（2 倍）<br/>- 已被 Collections.synchronizedList() 取代 | - 需要執行緒安全但不頻繁操作的場合<br/>- 建議使用 ArrayList 配合 Collections.synchronizedList() |

### Set
特性：無序、不允許重複元素、基於 equals() 和 hashCode() 判斷重複

| 類型 | 特性 | 適用場景 |
| --- | --- | --- |
| **HashSet** | - 基於 `HashMap` 實現<br/>- 無序儲存，存取快 (`O(1)`)<br/>- 允許 null 元素<br/>- 非同步 | - 需要快速去重的集合<br/>- 不在意元素順序<br/>- 需要 O(1) 時間複雜度的添加/刪除/查詢 |
| **TreeSet** | - 基於 `TreeMap` 實現<br/>- 元素排序儲存<br/>- 不允許 null 元素<br/>- 使用自然排序或自定義 Comparator<br/>- 操作複雜度為 O(log n) | - 需要有序的去重集合<br/>- 需要範圍查詢<br/>- 需要獲得排序後的元素 |
| **LinkedHashSet** | - 基於 `LinkedHashMap` 實現<br/>- 維護插入順序<br/>- 結合 HashSet 的快速和有序性 | - 需要記住插入順序的去重集合<br/>- 需要可預測的迭代順序 |

### Map
特性：key-value 資料結構、key 不可重複、value 可重複

| 類型 | 特性 | 適用場景 |
| --- | --- | --- |
| **HashMap** | - 無序鍵值映射<br/>- 存取快 (`O(1)`)<br/>- 非同步<br/>- 允許 null 鍵和值<br/>- Java 8 後使用紅黑樹優化碰撞 | - 單執行緒下的快速查詢與更新<br/>- 不需要排序的鍵值對應用 |
| **TreeMap** | - 鍵值排序存儲<br/>- 基於紅黑樹<br/>- 存取速度為 `O(log n)`<br/>- 不允許 null 鍵，允許 null 值<br/>- 可使用自定義 Comparator | - 需要按鍵排序的映射<br/>- 需要範圍操作<br/>- 需要最接近值查詢 |
| **Hashtable** | - 同步版的 HashMap<br/>- 執行緒安全但效率較低<br/>- 不允許 null 鍵和值<br/>- 已被 ConcurrentHashMap 取代 | - 建議使用 ConcurrentHashMap 替代<br/>- 舊系統維護 |
| **LinkedHashMap** | - 維護插入順序或訪問順序<br/>- 結合 HashMap 的快速和有序性<br/>- 可實現 LRU 快取 | - 需要記住鍵值對插入順序<br/>- 需要實現 LRU 快取機制 |

### 補充說明

1. **執行緒安全的選擇**：
    - 使用 Collections 的同步包裝器：`Collections.synchronizedList/Set/Map()`
    - 使用 `java.util.concurrent` 包中的實現：
        - `CopyOnWriteArrayList`
        - `ConcurrentHashMap`
        - `ConcurrentSkipListSet/Map`

2. **效能考量**：
    - 初始容量設置：若預知集合大小，建議設定適當的初始容量
    - 載入因子：HashMap/HashSet 預設 0.75，可根據需求調整
    - 迭代效能：LinkedHashMap/Set 比 HashMap/Set 稍慢，但提供可預測的迭代順序

3. **選擇建議**：
    - 一般場景：ArrayList、HashMap
    - 需要排序：TreeSet、TreeMap
    - 需要執行緒安全：ConcurrentHashMap、CopyOnWriteArrayList
    - 需要記住順序：LinkedHashMap、LinkedHashSet

## Array 與 ArrayList 的差別
### **1. Array（陣列）**

**簡介**：

`Array` 是 Java 中最基本的陣列結構，提供了固定大小的、連續存儲的數據結構。特性：
- **固定大小**：在創建時需要指定大小，且不能動態調整。
- **快速存取**：`Array` 提供 **O(1)** 的隨機存取速度，非常適合頻繁讀取資料的場合。
- **元素類型固定**：只能存儲相同類型的元素，且元素可以是任何物件或基本類型（如 `int`、`char` 等）。

兩者之間的比較

| \        | Array | ArrayList |
|----------| --- | --- |
| **大小**   | 固定 | 可動態調整 |
| **型別**   | 支援基本類型與物件 | 只支援物件類型 |
| **效能**   | 記憶體占用較少，效能較高 | 增刪元素時效能較低 |
| **線程安全** | 非同步 | 非同步 |

### 堆疊 (Stack)：後進先出 (LIFO) 原理

**堆疊（Stack）** 是一種遵循**後進先出（Last In, First Out, LIFO）**的數據結構。在 Java 中，當方法被調用時，會被壓入堆疊頂部，方法結束後會從堆疊中彈出。

#### 運作原理：

假設方法調用順序如下：

`method1()` → `method2()` → `method3()`
1. `method3()` 會先執行並完成，然後從堆疊中彈出。
2. 接著 `method2()` 完成並彈出。
3. 最後 `method1()` 完成。

### 案例：呼叫堆疊 (Call Stack) 與 堆疊追蹤 (Stack Trace)

- **Call Stack**：管理方法調用順序，每次方法調用會生成一個 `StackTraceElement`，記錄方法名稱、文件名、行號等，方法完成後該元素從堆疊中彈出。
- **Stack Trace**：當發生錯誤時，堆疊追蹤會列出從方法調用開始到錯誤發生時的完整堆疊路徑，幫助開發者定位錯誤來源。

大致介紹如上。