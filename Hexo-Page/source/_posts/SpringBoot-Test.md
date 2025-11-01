---
title: SpringBoot Test
category: Development
date: 2025-01-08 22:24:44
index_img: img/Java_logo.png
 
tags:
  - SpringBoot
  - Test
---
在開發完你的應用程式後，儘管非必要，但許多人都建議要撰寫測試去確保功能是可以正常運作的。 ( TDD 甚至主張 測試撰寫要在 開發之前 )，不論你是在開發前撰寫測試或是在開發後才寫測試，我都認為測試是重要且不可少的。

對我來說，撰寫測試的主要好處是建立信心，一個覆蓋率高的測試案例，可以讓我更大膽地針對程式碼進行重構，改完後也不需要重新撰寫測試(或者撰寫少量)。他可以帶來的好處是非當下的，也就是說當下撰寫本身沒有辦法產生效益，也有些公司覺得無法產生效益而不做。但無論如何，了解概念都是重要的。

本回會使用SpringBootTest撰寫測試。主要要介紹的測試有三種：

1. 單元測試 UNIT TEST
2. 整合測試 INTEGER TEST
3. 端對端測試 END TO END TEST

單元測試 中的 單元 ，意味著程式中最小的一個單位，也就是方法，單元測試主要用來確保一個被撰寫好的方法，是否正常運作。舉例來說以下有一個Member的驗證方法，我撰寫了一個假資料來確保這樣的資料結構是否足夠通過驗證。

```java

@RunWith(SpringRunner.class)
@SpringBootTest
class AioApiApplicationTests {

@Test
    void testUnitsTest() throws Exception {
        Member member = Member.builder()
                .name("Clark")
                .account("clark1232")
                .password("cclark23123")
                .email("test@gmail.con")
                .address("台北").build();
        assertTrue(Member.registerValidate(member));

    }
}
```

整合測試的層級比單元測試高，整合測試主要著重的是在確保當下元件正常之外，與外部服務之間的互動也必須要符合預期結果，撰寫難度也因此比單元測試高。舉例來說，我想要去取得會員的資料並進行驗證，但MemberService同時依賴MemberRepository，所以如果我想測試他的話，我不只要初始化MemberService也要初始化MemberRepository。

```java
    public class MemberService {

    private final MemberRepository memberRepository;
    public MemberService(MemberRepository repo) {
    this.repository = repo;
    }
    
    public Optional<MemberPO> findAccountById(Long id) {
        return memberRepository.findById(id);
    }
}
```

測試時，為了力求測試的準確度，我們希望會造成影響的變因越小越好，然後實際上，一個整合測試中經常性的依賴多個服務，針對多個服務都直接使用的話，開銷將會非常高。因此現在常見的做法是針對外部依賴採用假造(Mock)的形式完成。

那怎麼假造呢？這時就可以使用SpringBoot提供的Mockito去完成。

```java
    @MockBean
    MemberRepository memberRepository;
    @Autowired
    MemberService memberService;

@Test
    void optionalTest2() {
        MemberPO memberPOTest = new MemberPO();
        memberPOTest.setStatus("INACTIVE");
        Mockito.when(memberRepository.findByAccount(Mockito.isNull())).thenReturn(memberPOTest);
        Optional<MemberPO> memberPO1 = memberService.findActiveAccount(null);
        assert !memberPO1.isPresent();
    }
```

因為MemberPository是一個Bean，所以我引入了MockBean去假造它。此外，我使用了Mockito這個測試框架去模擬memberPository的行為，只要findByAccount方法中傳出了Null，就回傳一個我是先建立好的一個模擬物件(memberPOTest)

基本上Mockito的寫法是 Mockito.when(要被測試的方法).thenReturn(你預計的回傳內容)，透過這樣的方法可以在測試的@Test中覆蓋原先memberPosiory.findByAccount()的方法 ( 應該也是回傳Null，SpringBootTest 不會真的引入Bean )

最後，我們使用 assert 斷言去判斷memberPO 是否存在

最後，我們來看一下端對端測試吧。端對端測試我理解的就是API層級的測試，它包含了更多貼近商業情境的測試，也是測試上不好測試，經常會透過人工執行的測試。SpringBoot我經常使用Mockmvc來進行整合測試，舉例如下：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc // 注意要額外定義@AutoConfigureMockMvc，否則無法使用
class AioApiApplicationTests {

@Autowired
private MockMvc mockMvc;

@Test
void testGetMember() throws Exception {
    MemberPO memberPO = new MemberPO();
    memberPO.setStatus("ACTIVE");
    memberPO.setName("Clark");
    Mockito.when(memberRepository.findById(1L)).thenReturn(Optional.ofNullable(memberPO));

    mockMvc.perform(get("/api/1.0/member")
                    .param("id", "1")
                    .accept(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(content().contentType(MediaType.APPLICATION_JSON))
            .andExpect(jsonPath("$.info.name").value("Clark"));
}

}
```

這三種測試是我在初學階段時經常搞不清楚的，這次也趁著鐵人賽的機會一口氣把它們釐清，希望以上內容能幫助到什麼人。那今天的分享就到此為止了，我們明天見。
