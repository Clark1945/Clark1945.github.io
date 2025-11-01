---
title: 在SpringBoot 寫測試 @Mock,@MockBean,MvcResult
category: Development
date: 2024-05-01 11:06:01
tags: SpringBoot
---

在Spring開發完後，我們需要測試自己撰寫的API有沒有問題。除了在SpringBoot專案預設的SpringBootTest中針對單個方法進行單位測試外，有時也會有需要去Mock，就是假造一個物件的需求。
假設今天程式比較複雜，需根據DB回傳的資料做判斷，而DB剛好沒有這樣的資料時，與其更動DB以符合測試，不如假造DB的內容，單測試API。換句話說，因為成本太高，所以利用假造的方式來撰寫符合需要規模的程式碼。

### Mockito框架
Mockito是一個Mock框架，同時適用單元測試與整合測試。以下介紹常見的用法：
<!-- more -->
1. @MockBean 可以用來假造Repository，也就是假造資料庫回傳的資料，因為是假造一整個物件，所以比較適合在整合測試中使用。舉例來說，像是以下用法：

```
    @MockBean
    private StudentRepository stuRepo;

    @Test
    public void studentMock() {
        Student stu = new Student();
        stu.setStudentId("001");
        stu.setStudentName("Joe"); 
        stu.setStudentCreateTime(LocalDateTime.now()); 
        // 假造一個Student物件

        Mockito.when(studentRepo.getAbsentStudent(Mockito.anyString())).thenReturn(stu);
        // 當 想要取得請假學生的資料時，就回傳假造的Student物件
    }
```

2. 想要假造方法或是變數，假設我有一個物件MainService。

```
    @Test
	public MainService MockConfig() throws Exception {
		MainService mainService = new MainService();
		MainService mainService_mock = Mockito.spy(mainService); 
        // 假造一個Mock物件
		
        Mockito.doReturn(null).when(mainService_mock).SendRequest(Mockito.anyObject(),Mockito.anyString());
        // 使MainService中的SendRequest()回傳Null
		
        Mockito.doNothing().when(mainService_mock).validate(Mockito.anyObject()); // 先不做欄位檢查
		// 使MainService中的validate()方法跳過不執行(僅限方法沒有回傳值的狀況 void)

        ReflectionTestUtils.setField(mainService_mock, "domain", "https://www.google.com.tw/"); 
        // 仿造MainService中的@Value
		
        ReflectionTestUtils.setField(mainService_mock, "apiKey", "a4d1206bf951479fbcbe2e4bb6475d41"); // 仿造 @Value
		// 仿造MainService中的@Value

	}

```

### SpringMvc 單元測試
如果想要做部分整合測試，可以使用MvcResult去測試一個Rest API
```
HttpHeaders httpHeaders = new HttpHeaders();
httpHeaders.add("Content-Type", "application/json");
httpHeaders.add("Version", "1.0.1");
httpHeaders.add("ClientIp", "1.1.1.1");

RequestBuilder requestBuilder =
        MockMvcRequestBuilders
                .get("/api/transaction")
                .headers(httpHeaders)
                .param("type", "Credit")
                .param("sum", "10000");
// 要被測試的API需要建立一個ReqeustBuilder

MvcResult result = mockMvc.perform(requestBuilder)
        .andDo(print())
        .andExpect(jsonPath("$.code").value("0"))
        .andExpect(jsonPath("$.message").value("SUCCESS"))
        .andReturn();
// 測試，回傳會印出一個標準化的規格。若不符合andExpect()的條件就會回報AssertionError


```

以上。