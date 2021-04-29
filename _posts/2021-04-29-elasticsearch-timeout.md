---
layout: post
comments: true
title: 에러노트 - elasticsearch timeout
tags: [spring, elasticsearch]
---

### 에러 메세지

Spring 에서 `RestHighClient`로 Elasticsearch 를 연결해서 사용중에 아래와 같은 오류가 발생했습니다.

```
org.springframework.dao.DataAccessResourceFailureException: 30,000 milliseconds timeout on connection
http-outgoing-0 [ACTIVE]; nested exception is java.lang.RuntimeException: 30,000 milliseconds timeout on connection http-outgoing-0 [ACTIVE] at
...

```

---

### 실행환경

- Java8
- Spring boot 2.3.1.RELEASE
- spring-boot-starter-data-elasticsearch

---

### 에러코드

```java
// ElasticsearchConfig.java

    @Bean
    public RestHighLevelClient taskLogElasticsearchClient(ElasticsearchProperty elasticsearchProperty)
    {
        ElasticsearchProperty.TaskLog property = elasticsearchProperty.getTaskLog();

        HttpHost[] httpHosts = property.getHost().stream()
            .map(host -> new HttpHost(host, property.getPort(), "http"))
            .toArray(HttpHost[]::new);

        CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
        credentialsProvider.setCredentials(
            AuthScope.ANY,
            new UsernamePasswordCredentials(property.getUsername(), property.getPassword())
        );

        return new RestHighLevelClient(
            RestClient.builder(httpHosts)
                .setHttpClientConfigCallback(
                    httpClientBuilder -> httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider)
                )
        );
    }
```

---

### 에러 원인 분석

에러 상황

- Elasticsearch 조회 수행시 빈번하게 connection timeout 오류가 발생함
- default timeout 설정값으로 30초 (30,000ms)가 설정되어 있는 상태
- 실제 ES는 문제없이 정상 수행중

원인 분석

- 클라이언트에서 connection 종료를 인지하지 못하고 계속들고있다가 lost connection으로 시도하는것으로 추측됨


---

### 해결방법

- keep-alive를 사용해 connection을 재상요 할 수 있도록 함
- 관련 이슈 (https://github.com/elastic/elasticsearch/issues/65213)
- 위 이슈에서 제안하는 방법은 OS TCP 레이어 설정을 직접 바꿔야 하기 때문에, 아래 방법으로 keep-alive 설정 하는것을 추천

```java
// ElasticsearchConfig.java

    @Bean
    public RestHighLevelClient taskLogElasticsearchClient(ElasticsearchProperty elasticsearchProperty)
    {
        ElasticsearchProperty.TaskLog property = elasticsearchProperty.getTaskLog();

        HttpHost[] httpHosts = property.getHost().stream()
            .map(host -> new HttpHost(host, property.getPort(), "http"))
            .toArray(HttpHost[]::new);

        CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
        credentialsProvider.setCredentials(
            AuthScope.ANY,
            new UsernamePasswordCredentials(property.getUsername(), property.getPassword())
        );

        return new RestHighLevelClient(
            RestClient.builder(httpHosts)
                .setHttpClientConfigCallback(
                    httpClientBuilder ->
                        httpClientBuilder
                            .setDefaultCredentialsProvider(credentialsProvider)
                            .setConnectionReuseStrategy((response, context) -> true) // keepAlive use true
                            .setKeepAliveStrategy((response, context) -> 300) // keepAlive timeout sec

                )

        );
    }

```