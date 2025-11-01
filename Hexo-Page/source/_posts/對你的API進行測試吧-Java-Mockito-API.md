---
title: 對你的API進行測試吧 - Java Mockito API
category: Development
date: 2024-03-02 21:01:01
 
index_img: /img/code.jpg
tags: 
  - SpringBoot
  - Mockito
  - UnitTest
  - IntegrationTest
---

在開發SpringBoot應用程式的途中，一般來說會建議盡量要寫測試(但也有例外，比如說我現在待的這間)，其中測試又有分為單元測試、整合測試、端對端測試等等。
但本文並不是要說明這三者測試之間的差別，而是介紹測試用的Mock框架。所以想要了解那三者之間的差別的人，可以參考：

一般來說，想要在SpringBoot中撰寫測試時，就會在預設的@SpringBootTest中撰寫測試程式碼。以下是一個測試的範例：

RoutingHealth是一個存放於資料庫的物件，該物件可以決定通路(聯邦銀行)是否被鎖定，系統假設設定了鎖定的條件，而我要測試鎖定的狀況，
```java
    @MockBean
    private RoutingHealthRepository rhRepo;
    // 這是Mockito使用的假造一個bean，使用假造的代表裡面很乾淨，
    //不會有什麼Spring的配置，poperties等都沒有
    
    @Test
    public void HealthTunnelDuringLockTime() {
        String outChannel = "UBOT";
        String reqTime = "2023-11-24T16:30:48.183"; // 交易時間
        String lastTimeWasLock = "2023-11-24T16:25:48.183"; // 上次UBOT被鎖定時間
        String healthFlag = "Y"; // 健康度分流功能
        MockDBObject(outChannel,lastTimeWasLock,healthFlag); // 偽造DB物件
        boolean isPass = routingHealthService.getRoutingHealthByOutChannel(outChannel,reqTime);
        Assert.assertFalse(isPass);
        System.out.println("鎖定狀態下，健康度分流已開啟，交易時間30分鐘內 block success!");
    }
    @Test
    public void HealthTunnelOverLockTime() {
        String outChannel = "UBOT";
        String reqTime = "2023-11-24T16:56:48.183"; // 時間設定超過前次鎖定時間加30分鐘
        String lastTimeWasLock = "2023-11-24T16:25:48.183";
        String healthFlag = "Y";
        MockDBObject(outChannel,lastTimeWasLock,healthFlag);
        boolean isPass = routingHealthService.getRoutingHealthByOutChannel(outChannel,reqTime);
        Assert.assertTrue(isPass);
        System.out.println("鎖定狀態下，健康度分流已開啟，交易時間超過30分鐘 pass success!");
    }
    @Test
    public void HealthTunnelClosed() {
        String outChannel = "UBOT";
        String reqTime = "2023-11-24T16:30:48.183";
        String lastTimeWasLock = "2023-11-24T16:25:48.183";
        String healthFlag = "N"; // 關閉健康度分流
        MockDBObject(outChannel,lastTimeWasLock,healthFlag);
        boolean isPass = routingHealthService.getRoutingHealthByOutChannel(outChannel,reqTime);
        Assert.assertTrue(isPass);
        System.out.println("鎖定狀態下，健康度分流已關閉，交易時間30分鐘內 pass success!");
    }

    @Test
    public void HealthTunnelUBTestBefore() { // UB未鎖定，
        String outChannel = "UBOT";
        String reqTime = "2024-03-15T14:00:48.183";
        String lastTimeWasLock = "2024-03-18T16:00:48.183";
        String healthFlag = "Y"; // 啟用健康度分流
        MockDBObjectUBTest(outChannel,lastTimeWasLock,healthFlag);
        boolean isPass = routingHealthService.getRoutingHealthByOutChannel(outChannel,reqTime);
        Assert.assertTrue(isPass);
        System.out.println("鎖定狀態下，健康度分流已啟用，交易時間早於鎖定時間 pass success!");
    }

    @Test
    public void HealthTunnelUBTestAfter() { // UB未鎖定，
        String outChannel = "UBOT";
        String reqTime = "2024-03-18T17:00:48.183";
        String lastTimeWasLock = "2024-03-18T16:00:48.183";
        String healthFlag = "Y"; // 啟用健康度分流
        MockDBObjectUBTest(outChannel,lastTimeWasLock,healthFlag);
        boolean isPass = routingHealthService.getRoutingHealthByOutChannel(outChannel,reqTime);
        Assert.assertTrue(isPass);
        System.out.println("鎖定狀態下，健康度分流已啟用，交易時間晚於鎖定時間30分鐘後 pass success!");
    }

    @Test
    public void HealthTunnelUBTestDuring() { // UB未鎖定，
        String outChannel = "UBOT";
        String reqTime = "2024-03-18T16:00:49.183";
        String lastTimeWasLock = "2024-03-18T16:00:48.183";
        String healthFlag = "Y"; // 啟用健康度分流
        MockDBObjectUBTest(outChannel,lastTimeWasLock,healthFlag);
        boolean isPass = routingHealthService.getRoutingHealthByOutChannel(outChannel,reqTime);
        Assert.assertFalse(isPass);
        System.out.println("鎖定狀態下，健康度分流已啟用，交易時間於鎖定時間30分鐘內 ban success!");
    }

    public void MockDBObjectUBTest(String outChannel,String lastTimeWasLock,String healthFlag) {
        RoutingHealth rh = new RoutingHealth();
        rh.setRhType(outChannel);
        rh.setRhIsActive("N"); // 假設聯邦Gateway是被鎖住的
        rh.setRhLockTime(LocalDateTime.parse(lastTimeWasLock)); // 聯邦Gateway上次被鎖定的時間點
        Mockito.when(coreFacade.getSystemSetting(SSCode.LOCK_MINUTES)).thenReturn("30");
        Mockito.when(coreFacade.getSystemSetting(SSCode.ROUTING_HEALTH_FLAG)).thenReturn(healthFlag); // 健康度分流是否開啟
        Mockito.when(rhRepo.getRoutingHealthByRhTypeAndRhLockTimeBetween(outChannel)).thenReturn(rh);
    }

    public void MockDBObject(String outChannel,String lastTimeWasLock,String healthFlag) {
        RoutingHealth rh = new RoutingHealth();
        rh.setRhType(outChannel);
        rh.setRhIsActive("N"); // 假設聯邦Gateway是被鎖住的
        rh.setRhLockTime(LocalDateTime.parse(lastTimeWasLock)); // 聯邦Gateway上次被鎖定的時間點
        Mockito.when(coreFacade.getSystemSetting(SSCode.LOCK_MINUTES)).thenReturn("30");
        Mockito.when(coreFacade.getSystemSetting(SSCode.ROUTING_HEALTH_FLAG)).thenReturn(healthFlag); // 健康度分流是否開啟
        Mockito.when(rhRepo.getRoutingHealthByRhTypeAndRhLockTimeBetween(outChannel)).thenReturn(rh);
    }
    
    // poperties 的假造
  @Test
	public void testRefundPoint() throws Exception {
		MainService mainService = MockConfig();

		HashMap reqJson = new HashMap();
		reqJson.put("pmMemberNo","GID76683851296887");
		reqJson.put("pmPointAmt","50000");
		reqJson.put("pointId","ResIsFatF2-4");
		reqJson.put("pmSellerName","testMerchantLaLaLa");
		reqJson.put("allowInsufficient","Y");
		reqJson.put("merchantNo","0904000001");
		reqJson.put("refundId","ResIsChubby01");
		reqJson.put("mchId","ACpay");

		Fundmentals fundmentals = Fundmentals.builder()
				.tradeType(TradeType.CANCEL)
				.apiKey("xyz123")
				.accessUrl("https://icash-apim.icashsys.com.tw/Point-UAT/v1/RefundPoint").build();
		String respJSON = mainService.mainprocedure(reqJson,fundmentals);
		HashMap<String,String> result = objectMapper.readValue(respJSON, HashMap.class);
		Assert.assertEquals("3003",result.get("resultCode"));
	}
	
	public MainService MockConfig() throws Exception {
		MainService mainService = new MainService();
		MainService mainService_mock = Mockito.spy(mainService);
		Mockito.doReturn(null).when(mainService_mock).SendRequest(Mockito.anyObject(),Mockito.anyString()); // 先不發到OP
		Mockito.doNothing().when(mainService_mock).validate(Mockito.anyObject()); // 先不做欄位檢查
		ReflectionTestUtils.setField(mainService_mock, "queryDomain", "https://icash-apim.icashsys.com.tw/Point-UAT/v1/QueryPoint"); // 仿造 @Value
		ReflectionTestUtils.setField(mainService_mock, "apiKey", "xyz123"); // 仿造 @Value
		return mainService_mock;
	}
```

### MVCResult 測試 端對端測試
```java
HttpHeaders httpHeaders = new HttpHeaders();
httpHeaders.add("Content-Type", "application/json");
httpHeaders.add("Version", "1.0.1");
httpHeaders.add("ClientIp", "1.1.1.1");

RequestBuilder requestBuilder =
        MockMvcRequestBuilders
                .get("/api/getPointExpectAmt")
                .headers(httpHeaders)
                .param("pmslType", "AM")
                .param("purchaseAmt", "10000");

MvcResult result = mockMvc.perform(requestBuilder)
        .andDo(print())
        .andExpect(jsonPath("$.code").value("0"))
        .andExpect(jsonPath("$.message").value("SUCCESS"))
        .andReturn();
```




