---
layout: post
comments: true
title: Spring restTemplate connection reset error
tags: [spring]
---

### 에러 환경

- Java8
- Spring boot 2.1.7.RELEASE  
- Spring batch

---

### 에러 상황

[원부-배치-속성] 인스턴스에서 배치 종료에 대한 메시지를 MS-Teams 채널로 보내는 상황에서 에러 발생, 시작시에는 문제 없음.  
배치 시작시 1회 connection 이후, 1시간뒤 connection이 일어날때 에러 발생  
@Component bean으로 등록된 class 내부에 멤버 변수로 restTemplate이 선언되어있는 상황  

```java

@Component
public class ErrorUtil
{
    private RestTemplate restTemplate;
 
    @PostConstruct
    public void init()
    {
        HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory();
        factory.setReadTimeout(5000); // 읽기시간초과, ms
        factory.setConnectTimeout(3000); // 연결시간초과, ms
        HttpClient httpClient = HttpClientBuilder.create()
            .setMaxConnTotal(100) // connection pool 적용
            .setMaxConnPerRoute(5) // connection pool 적용
            .build();
        factory.setHttpClient(httpClient); // 동기실행에 사용될 HttpClient 세팅
 
        restTemplate = new RestTemplate(factory);
        ...    
 
 
    }
}
```

---

### 에러로그  

```c
org.springframework.web.client.ResourceAccessException: I/O error on POST request for "https://*.*.*": Connection reset; nested exception is java.net.SocketException: Connection reset
        at org.springframework.web.client.RestTemplate.doExecute(RestTemplate.java:744)
        at org.springframework.web.client.RestTemplate.execute(RestTemplate.java:670)
        at org.springframework.web.client.RestTemplate.exchange(RestTemplate.java:579)
        at com.wmp.catalog.batch.util.TeamsNotificationBUtil.postRequestHttp(TeamsNotificationBUtil.java:75)
        at com.wmp.catalog.batch.util.TeamsNotificationBUtil.sendMessage(TeamsNotificationBUtil.java:52)
        at com.wmp.catalog.batch.job.step.JobSettingStep.sendMessage(JobSettingStep.java:162)
        at com.wmp.catalog.batch.job.step.JobSettingStep.lambda$setJobFinish$1(JobSettingStep.java:70)
        at org.springframework.batch.core.step.tasklet.TaskletStep$ChunkTransactionCallback.doInTransaction(TaskletStep.java:407)
        at org.springframework.batch.core.step.tasklet.TaskletStep$ChunkTransactionCallback.doInTransaction(TaskletStep.java:331)
        at org.springframework.transaction.support.TransactionTemplate.execute(TransactionTemplate.java:140)
        at org.springframework.batch.core.step.tasklet.TaskletStep$2.doInChunkContext(TaskletStep.java:273)
```

---
### 에러 이유

- 배치 작업에 사용하면서, 배치 시작할때 커넥션 이후 끝날때 커넥션을 진행하면서 장기간 미사용된 커넥션에 대한 문제로 보임
- 정확한 원인 파악중

---

### 문제 해결

- restTemplate 객체를 message 보낼때 지역변수로 선언해줌. (현재 배치주기상 1일 2회의 커넥션 사용함)  
- retry를 통해 restTemplate 실패시 재시도 하도록 설정  

```java
@Retryable(value = Exception.class, maxAttempts = 3, backoff = @Backoff(delay = 2000))
public void sendMessage(String title, String content) throws JsonProcessingException
{
    String response = postRequestHttp();
}
 
@Retryable(value = Exception.class, maxAttempts = 3, backoff = @Backoff(delay = 2000))
public void sendErrorMessage(String title, String content) throws JsonProcessingException
{
    String response = postRequestHttp(;
}
 
private String postRequestHttp() throws JsonProcessingException
{
    HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory();
    factory.setReadTimeout(5000); // 읽기시간초과, ms
    factory.setConnectTimeout(3000); // 연결시간초과, ms
 
    RestTemplate restTemplate = new RestTemplate(factory);
 
    HttpEntity<String> request;
    ObjectMapper mapper = new ObjectMapper();
    request = new HttpEntity<>(mapper.writeValueAsString(message), headers);
    ResponseEntity<String> result = restTemplate.exchange(HttpMethod.POST, request, String.class);
 
    return result.getBody();
}
 
@Recover
private void doNothing()
{
    // retry recover 을 위해 아무것도 하지않는 메서드
}
```