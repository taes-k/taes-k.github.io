---
layout: post
comments: true
title: Spring Feign으로 http 통신하기
tags: [spring, fiegn]
---

### Feign  

- Netflix 에서 개발한 선언적 http 통신을 위한 오픈소스 입니다.  
- 선언적 통신을 통해 http 통신 코드를 확연하게 줄여줄수 있다는 장점이 있습니다.  
- 현재는 Spring cloud 에서 공식 라이브러리로 지원하고 있습니다.  
- netflix-feign의 last-update는 2016년 7월이며(v8.18), Spring cloud 에서는 1.2.x 버전(2016년 9월)부터 netflix-feign이 아닌 open-feign을 통해 공식 라이브러리로서 제공하고 있습니다.  
  
maven repository : https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-feign  
  
### Dependency  

- Spring cloud openfeign  
![1]({{ site.images | relative_url }}/posts/2019-11-05-spring-feign/1.png)    
  
### Feign 적용을 위한 Code   

- dependency 추가  

```java
// build.gradle

    compile 'org.springframework.cloud:spring-cloud-starter-openfeign'
    compile 'io.github.openfeign:feign-httpclient'
```


- `@EnableFeignClients` 설정  

```java
// SampleApplication.java
  
@EnableFeignClients
@SpringBootApplication
public class SampleApplication
{
    public static void main(String[] args)
    {
        SpringApplication.run(SampleApplication.class, args);
    }
}
```  

- FeignClient 구현  

```java
// SampleClient.java

@FeignClient(name = "sample-api", url = "${sample.connection.api.url}")
public interface SampleClient
{
    @GetMapping(value = "/api/v1.0/sample")
    String getSample(@RequestParam("param") String param);
}
```

- FeignClient call 구현

```java
// SampleService.java

private final SampleClient sampleClient;
 
@Autowired
public SampleService(SampleClient sampleClient)
{
    this.sampleClient = sampleClient;
}
 
...
...
 
private String getSample(String param)
{
    return wmpTotalSearchClient.getSearchCount(param);
}
```

### Feign을 위한 환경 설정

- annotation 설정  

```java
// feignClientInteface

@FeignClient(
        name = (서비스ID를 지정함으로서, Eureka, Ribbon등에서 나타납니다.),
        url = (실제 호출할 서비스의 URL 입니다.),
        decode404 = (404에러시 FeignExeption을 발생시킬지, 아니면 응답을 decode할 지 여부를 설정합니다.)ㅡ
        configuration =(custom한 설정을 지정할때, feign configuration class 지정해 줍니다.),
        fallback = (hystrix fallback class 지정하여 예외처리를 해줍니다.),
        fallbackFactory = (hystrix fallbak factory 지정해줍니다.))
```

- property 설정  

```java
// application.yml

feign:
    client:
        config:
            feignName: # @FeignClient에서 name 값, 전역으로 설정하려면 default
                connectTimeout: 5000
                readTimeout: 5000
                loggerLevel: full
                errorDecoder: com.example.SimpleErrorDecoder
                retryer: com.example.SimpleRetryer
                requestInterceptors:
                    - com.example.FooRequestInterceptor
                    - com.example.BarRequestInterceptor
                decode404: false
                encoder: com.example.SimpleEncoder
                decoder: com.example.SimpleDecoder
                contract: com.example.SimpleContract
```

### Feign Custom configuration   

위의 Feign 설정내용을 확인하시면, Feign을 구현하실때 별도의 Cofiguration을 설정해 줄 수 있습니다.  
일반적으로 통신에 많이들 적용하는 몇가지 설정들에대해서 정리해보도로고 하겠습니다.   

- BasicAuthorization 사용하기  

```java
// SampleClient.java

@FeignClient(configuration = {FeignConfiguration.class})
...
```

```java
// FeignConfiguration.java

@Bean
public BasicAuthRequestInterceptor basicAuthRequestInterceptor()
{
    return new BasicAuthRequestInterceptor(id,password);
}
```

- x-www-from 사용하기  
From 형식의 파라미터를 사용할때에는, 일반 자바 object serialization으로는 파라미터가 제대로 전달되지 않기때문에, 별도의 설정을 진행해 주어야 합니다.  

```java
// SampleClient.java
 
@FeignClient(name = "sample-client", url = "${sample.connection.api.url}", configuration = {FormConfig.class})
public interface SampleClient
{
    @PostMapping(value = "/api/v1.0/sample", consumes = "application/x-www-form-urlencoded", produces = "application/json")
    String getSample(@RequestBody SampleParam param);
    ...

```

```java
// FeignConfiguration.java
 
@Autowired
private final ObjectFactory<HttpMessageConverters> messageConverters;
 
@Bean
public Encoder formEncoder()
{
    return new FormEncoder(new SpringEncoder(this.messageConverters));
}
```

- Custom header 사용하기  
통신을할때 Cusotm header 적용을위한 interceptor를 등록해줄수도 있습니다.

```java
// CustomHeaderConfig.java

@Override
public void apply(RequestTemplate template)
{
 
    if (RequestContextHolder.getRequestAttributes() != null)
    {
        HttpServletRequest req = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
 
        template.header("Authorization", req.getHeader(HttpHeaders.AUTHORIZATION));
    }
    else
    {
        throw new ApiException(ErrorCodes.INTERNAL_SERVER_ERROR, "Authorization Header를 불러오는데 실패했습니다.");
    }
}
```

### Connection pool  

feign은 기본적으로 JDK의 native URLConnection을 이용하여 HTTP request를 보냅니다.   
따라서 기본 설정으로는 별도의 Connection pool을 설정할수 없는데, Connection pool을 설정하기위해서는 apache HTTPClient, OKHttp로 feign의 client 바꾸어 사용하여야 합니다.  

### 장점  
- Spring annotation을 통해 http 통신을 간단하게 메서드화 시킬수 있습니다.  
- response를 받아 별도의 mapping 과정없이 바로 response 객체로써 return 받아 사용할수 있습니다.  
- 별도의 connection 환경 설정등의 작업없이 사용이 가능합니다.   
- 자체 retry 설정이 가능합니다.  
- 에러로직을 별도로 관리하여 분기가 가능합니다.  
- 기존방식 대비 Header 설정이 매우 편리합니다.  
- 실제로 RestTemplate을 사용하던 사내 개발 코드가 Feign으로 변경후 30% 이상 줄어들었습니다.  

### 참고 URL
- https://brunch.co.kr/@springboot/202  
