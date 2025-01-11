---
title: 隔離層級測試紀錄-使用Grails
date: 2025-01-11 10:13:27
category: Software
tags:
  - Grails
  - Isolation Level
  - Hibernate
---

我本來的想法是 READ_COMMITTED 只要commit()後的資料就應該要被讀取到，但實際上卻出現還是讀取到舊資料的問題。

```java
def testRepeatableRead() {
    Thread t1 = new Thread({
        testService.readData(2106)
    })
    Thread t2 = new Thread({
        Thread.sleep(5000) // Delay to ensure the first read in t1 has completed
        modifyDataService.modifyData(2106)
    })
    t1.start()
    t2.start()
    t1.join()
    t2.join()
    return "Success"
}

@Transactional(isolation=Isolation.READ_COMMITTED)
def repeatReadData(Long id) {
    MeetingRoomBooking booking = MeetingRoomBooking.get(id)
    println "First read: ${booking?.content}"
    // Simulate some processing time
    Thread.sleep(10000)
    // Read the same data again
    booking = MeetingRoomBooking.get(id)
    println "Second read: ${booking?.content}"
}

@Transactional
class ModifyDataService {
    def modifyData(Long id) {
        MeetingRoomBooking booking = MeetingRoomBooking.get(id)
        booking.content = "2"
        booking.save(flush: true)
        println "Data updated to: ${booking.content}"
    }
}

/// First read: 1
/// Data updated to: 2
/// Second updated: 1
```

## 發生原因

出现这种情况的原因是 Grails 的一级缓存（Session 级别缓存）导致的。默认情况下，Grails 使用 Hibernate 作为 ORM 框架，并且 Hibernate 会维护一个一级缓存，这个缓存会存储当前 Session 中加载的所有对象。在你的 `readData` 方法中，第一次调用 `MeetingRoomBooking.get(id)` 时，会从数据库加载对象并缓存到当前 Session 中。在第二次调用 `MeetingRoomBooking.get(id)` 时，Hibernate 会直接从一级缓存中获取对象，而不会再去查询数据库

为了确保你在第二次读取时能读取到最新的数据，你可以清理一级缓存。你可以使用 `withNewSession` 方法或者 `clear()` 方法来达到这个目的。

## 解法

1. withNewSession

```java
// Read the same data again in a new session
MeetingRoomBooking.withNewSession {
    booking = MeetingRoomBooking.get(id)
    println "Second read: ${booking?.content}"

```

1. clear()

```java
def readData(Long id) {
    sessionFactory.currentSession.clear()
}

```

Q: 那假設REPEATABLE_READ隔離層級下，可讀取到在其他地方被commit之後的資料嗎？

A:  clear Session無效，只能建立新的Session。

```java
@Transactional(isolation = Isolation.READ_COMMITTED)
def readData(Long id) {
    MeetingRoomBooking booking = MeetingRoomBooking.get(id)
    println "First read: ${booking?.content}"
    modifyData(id)
    Thread.sleep(5000)

    MeetingRoomBooking.withNewSession {
        booking = MeetingRoomBooking.get(id)
        println "Second Read:${booking?.content}"
    }
    booking = MeetingRoomBooking.get(id)
    println "Third read: ${booking?.content}"
}

//First read: 1
//Update to 2
//Second Read:1 (因為交易結束後才會commit(),withNewTransaction會讀到資料庫的舊資料
//Third Read:2 (直接讀取會讀取Hibernate的一級緩存，所以是修改後的值

```

Q: 有辦法在交易裡面強迫commit()嗎？

A:可以，使用Hibernate Session

```java
def sessionFactory

@Transactional(isolation = Isolation.READ_COMMITTED)
def readData(Long id) {
    MeetingRoomBooking booking
    booking = MeetingRoomBooking.get(id)
    println "First read: ${booking?.content}"

    Session session = sessionFactory.openSession()
    Transaction tx = null
    try {
        tx = session.beginTransaction()
        modifyData(id)
        Thread.sleep(5000)
        // Manually commit the transaction
        tx.commit()
    } catch (Exception e) {
        if (tx != null) tx.rollback()
        throw e
    } finally {
        session.close()
    }

    MeetingRoomBooking.withNewSession {
        booking = MeetingRoomBooking.get(id)
        println "new Transaction Read:${booking?.content}"
    }
    booking = MeetingRoomBooking.get(id)
    println "Direct read: ${booking?.content}"
}

// 1
// Update to 2
// 2 (此時已經commit，資料已經寫到資料庫了)
// 2

```

Q: 那如果我提高隔離層級呢

A:  依然讀取的是Hibernate快取，不符合可重複讀特性

```java
@Transactional
def readData(Long id) {

    MeetingRoomBooking booking = MeetingRoomBooking.get(id)
    println "First read: ${booking?.content}"

    Session session = sessionFactory.openSession()
    Transaction tx = null
    try {
        tx = session.beginTransaction()
        modifyData(id)
        Thread.sleep(5000)
        // Manually commit the transaction
        tx.commit()
    } catch (Exception e) {
        if (tx != null) tx.rollback()
        throw e
    } finally {
        session.close()
    }

    MeetingRoomBooking.withNewSession {
        booking = MeetingRoomBooking.get(id)
        println "new Transaction Read:${booking?.content}"
    }
    booking = MeetingRoomBooking.get(id)
    println "Direct read: ${booking?.content}"
}

//First read: A
//Data updated to: B
//new Transaction Read:B
//Direct read: B ( 照理來說應該是A 因為多次讀取都應該是同一個值。可能是Grails的行為？)

```

Q: 確認是否因為同一個交易的Transaction導致Hibernate緩存改變

A: 拆開

```java
class MeetingTestService {
    @Transactional
    def readData(Long id) {
        MeetingRoomBooking booking = MeetingRoomBooking.get(id)
        println "First read: ${booking?.content}"
        Thread.sleep(10000)
        MeetingRoomBooking.withNewSession {
            booking = MeetingRoomBooking.get(id)
            println "new Transaction Read:${booking?.content}"
        }
        booking = MeetingRoomBooking.get(id)
        println "Direct read: ${booking?.content}"
    }
}

// 1
//Update to 2
// 2 (因為已經commit，值已經改變)
// 1 (確認來自Hibernate快取，符合可重複讀特性)

```

嘗試降隔離層級

```java
class MeetingTestService {
    @Transactional(isolation = Isolation.READ_COMMITTED)
    def readData(Long id) {
        MeetingRoomBooking booking
        booking = MeetingRoomBooking.get(id)
        println "First read: ${booking?.content}"
        Thread.sleep(10000)
        MeetingRoomBooking.withNewSession {
            booking = MeetingRoomBooking.get(id)
            println "new Transaction Read:${booking?.content}"
        }
        booking = MeetingRoomBooking.get(id)
        println "Direct read: ${booking?.content}"
    }
}
//1
//2
//2
//1 依然受Hibernate緩存影響

```

如果清除Hibernate快取呢

```java
SessionFactory sessionFactory
    @Transactional(isolation = Isolation.READ_COMMITTED)
    def readData(Long id) {
        MeetingRoomBooking booking
        booking = MeetingRoomBooking.get(id)
        println "First read: ${booking?.content}"
        Thread.sleep(10000)
        MeetingRoomBooking.withNewSession {
            booking = MeetingRoomBooking.get(id)
            println "new Transaction Read:${booking?.content}"
        }
        sessionFactory.currentSession.clear()
        booking = MeetingRoomBooking.get(id)
        println "Direct read: ${booking?.content}"
  }
// A
// Update to B
// B
// B (正確，如果有改變，就使用改變後的值)
```

大概測試是這樣

## 結論

1. Hibernate 的一級緩存機制會影響資料讀取的行為
2. 要讀取最新資料，可以：
    - 使用 withNewSession
    - 清除當前 Session 緩存
3. 在使用 READ_COMMITTED 隔離層級時，需要特別注意緩存的影響
4. 使用獨立 Session 進行修改可以確保資料立即提交