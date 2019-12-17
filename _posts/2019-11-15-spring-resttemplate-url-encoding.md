---
layout: post
comments: true
title: Spring restTemplate UrlEncoding
tags: [spring, restTemplate]
---

### 개요  

RestTemplate을 사용하면서, request 파라미터를 설정시에 URL Encoding을 통해서 파라미터를 넘기는 케이스가 자주 있으실겁니다.  
하지만 이때, 주의해야할 사항이 있습니다.  
  
### 문제의 코드

```java
String accessUrl = String.format("%s?keyword=%s&attribute=%s&testTarget=quality", wmpSearchApiUrl,
        URLEncoder.encode(keyword,"UTF-8"),
        URLEncoder.encode(attributeElement,"UTF-8"));
 
RestTemplate restTemplate = new RestTemplate();
JsonNode jsonNode = objectMapper.readTree(restTemplate.getForEntity(accessUrl, String.class).getBody());
```

위 문제의 소스에서는 accessUrl을 String으로 만들고, 파라미터들에 특수문자가 오는경우가 많기때문에 URLEncoder로써 인코딩을 해줍니다.  
  
겉으로는 문제 없어 보이지만  restTemplate.getForEntity(accessUrl, String.class) 을 실행하면서 parameter로 넘겨진 accessUrl String은 내부적으로 URLEncoding이 한번더 일어나게되어 이중 인코딩 이되는 문제가 발생하게됩니다.  

### 변경된 코드  

```java
URI accessUrl = new URI(String.format("%s?keyword=%s&attribute=%s", wmpSearchApiUrl,
            URLEncoder.encode(keyword,"UTF-8"),
            URLEncoder.encode(attributeElement,"UTF-8")));
 
RestTemplate restTemplate = new RestTemplate();
JsonNode jsonNode = objectMapper.readTree(restTemplate.getForEntity(accessUrl, String.class).getBody());
```  

String Type으로 선언했던 Url을 URI Object로 선언하면, RestTemplate 내부에서 별도의 인코딩 처리가 일어나지 않습니다.  
