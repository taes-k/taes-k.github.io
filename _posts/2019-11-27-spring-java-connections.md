---
layout: post
comments: true
title: Spring, java connection에 대한 고찰
tags: [spring, java, network]
---

### 개요  

MSA 프로젝트를 진행하면서 Java, Spring에서 제공하는 다양한 Connection tool들을 이용해 통신을 진행하는데, 이때 실제 Connection들이 어떻게 일어나고 있는지 알아보도록 하겠습니다.

---

### Connection
Tomcat WAS 와 연결할때는 기본적으로 소켓 통신으로 Connection을 설정합니다.  

---

### Connection close & timeout

![1]({{ site.images | relative_url }}/posts/2019-11-27-spring-java-connections/1.png)  

TCP 연결의 종료과정중, close()를 먼저요청한 쪽에서는 소켓이 닫히지 않고 TIME_WAIT상태로 대기하는 시간을 갖게됩니다.  
TIME_WAIT 상태를 유지하는 이유는  
1. 지연 패킷을 받기위해서  
2. 상대의 CLOSE 를 확인하기위해서   
  
위 두가지 이유로 인해 TIME_WAIT라는 상태를 통해 socket을 바로  close하지 않고 대기하는 시간을 갖습니다.  

> RFC 793 에는 TIME_WAIT을 2*MSL(Maximum Segment Lifetime)로 규정했으며 CentOS 6에서는 60초 동안 유지됩니다. 아울러 이 값은 조정할 수 없습니다.   
> struct tcp_timewait_sock는 고작 168바이트  
> 만일 서버 투 서버로 1:1 대용량 접속이 발생할 경우 한 대의 클라이언트에서 가능한 최대 요청 수는 500 RPS(Requests Per Second) 정도입니다.  
> 그러나 이 수치를 넘어선다면 클라이언트의 로컬 포트가 고갈 될 것이며 TIME_WAIT 상태를 재사용 해야 합니다.   
> 소켓서버는 로컬 포트를 점유하지 않습니다. (최초 연결된 포트 하나로 바인딩됨, 로컬 포트 갯수인 65,535 제한사항 없음)  
> 클라이언트에서 서버에 접속할때는 이론적으로는 30,000개정도의 포트 사용가능  
  
참고 URL : https://tech.kakao.com/2016/04/21/closewait-timewait/  

---

### Connection : Close-wait  

![2]({{ site.images | relative_url }}/posts/2019-11-27-spring-java-connections/2.png)   

![3]({{ site.images | relative_url }}/posts/2019-11-27-spring-java-connections/3.png)   

위와 같이 Client에서 Connection이 맺어지면 (ESTABLISHED) 상태가되고 소켓이 맺어질때 설정한 timeout 시간이 지나면   

![4]({{ site.images | relative_url }}/posts/2019-11-27-spring-java-connections/4.png)   

ESTABLISHED → CLOSE_WAIT 상태가 됨을 확인 하실수 있습니다.  
CLOSE_WAIT인 상태에서 재연결을 시도하게된다면 다음과같은 상황을 확인하실수 있습니다.  

![5]({{ site.images | relative_url }}/posts/2019-11-27-spring-java-connections/5.png)   

10.**.**.46:443 (CLOSE_WAIT) 상태의 연결이 해제되고  
10.**.**.125:443 (ESTABLISHED) 새로운 포트에 새로운 커넥션이 생성되는것을 확인 하실수 있습니다.  
(10.**.**.46:443 와 10.**.**.125:443 는 스케일아웃된 같은 서버입니다.)  
  
추가로, 계속해서 사용되지 않는 CLOSE_WAIT는 일정시간이 지나면 연결이 해제됩니다.  

---

### Connection : Keep-alive

일반적으로 connection은 통신이 끝나면 바로 자원이 해제가 됩니다.   
Http/1.1 Spec을 사용하면서, 한번 맺어진 connection에 대하여 통신이 아직 끝나지 않은경우를 위하여 재사용 할 수 있도록 하는 내용을 정의 해 줄 수 있습니다.  
이때 사용하게되는것이 Connection : Keep-alive header 입니다.   
  
Connection을 유지하지 않는다면, 매 연결마다 handshaking  과정을 통해서 socket을 연결해야하는데, 통신이 확정적으로 자주 일어나는 서비스간에는 이 과정이 매우 비효율적인 일일 것 입니다.  
이에 따라 서버측에서 Http/1.1Spec에서 Keep-alive 헤더만 설정해주면 커넥션을 유지하여 사용해줄수 있는데  

```
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
```

위와 같은 옵션이라면, Connection은 마지막 연결로 부터 5초동안 유지되며 최대 100회까지 유지되는 옵셥을 설정 해 커넥션을 관리해 줄 수 있습니다.  
만약 Keep-Alive Timeout Option이 설정되어 있지 않다면 기본적으로 Tomcat에서 설정된 Connection timeout 시간으로 설정됩니다.  

```java
// o.a.tomcat.util.net.AbstractEndpoint.java
 
public int getKeepAliveTimeout() {
    if (keepAliveTimeout == null) {
        return getConnectionTimeout();
    } else {
        return keepAliveTimeout.intValue();
    }
}
```  
  
참고 url 

- https://www.imperva.com/learn/performance/http-keep-alive/  
- https://12bme.tistory.com/458  

---

### Java URLConnection  

```java

URL url = new URL("ACCESS_URL");
HttpURLConnection con = (HttpURLConnection) url.openConnection();
con.setConnectTimeout(5000);
con.setReadTimeout(5000);
con.setRequestMethod("GET");
 
 
StringBuilder sb = new StringBuilder();
    if (con.getResponseCode() == HttpURLConnection.HTTP_OK) {
        BufferedReader br = new BufferedReader(
            new InputStreamReader(con.getInputStream(), "utf-8"));
        String line;
        while ((line = br.readLine()) != null) {
            sb.append(line).append("\n");
        }
        br.close();
        System.out.println("" + sb.toString());
    }
    else {
        System.out.println(con.getResponseMessage());
    }
```

- 기본적으로 JAVA에서 기본적으로 제공하는 URLConnection은 Connection pool을 사용하지 않습니다.  
- URLConnection은 대상측에서 “Connection : keep-alive” 헤더가 있다면, connection을 살려두고 재사용합니다. 단, 이경우 명시적으로 connection을 닫아주지 않는다면 stream이 종료되더라도 connection이 닫히지 않습니다.  
- URLConnection 객체를 재사용 할 수 없습니다.  
- http 연결에 있어서 다량의 선작업이 필요해 생산성은 떨어지지만, 성능 및 속도면에서 가장 우수하다는 의견이 있습니다.  
  
참고 URL   
- https://stackoverflow.com/questions/16256681/how-to-reuse-httpurlconnection  
- https://goddaehee.tistory.com/161  

---

### Spring RestTemplate

```java
URI accessUrl = new URI("ACCESS_URI");
 
 
RestTemplate restTemplate = new RestTemplate();
ResponseEntity<String> response = restTemplate.getForEntity(accessUrl, String.class);
 
if (response.getStatusCode().value() == 200)
{
    ObjectMapper objectMapper = new ObjectMapper();
    JsonNode jsonNode = objectMapper.readTree(response.getBody());
    ...
}
```

- RestTemplate이 동작할때 default 설정으로 java의 SimpleClientHttpRequestFactory를 이용합니다.  


#### Connection 지속과 재사용

restTemplate은 기본적으로 연결할때마다 connection을 맺고 종료합니다.  
connection을 close하는 과정중에서 위에서 알아보았던 WAIT_TIME이 발생하여 로컬 포트를 점유하게되는데, 만약 반복적인 Connection이 계속해서 발생하게된다면 WAIT_TIME으로 로컬 포트들이 모두 점유당해 connection을 맺을수 없는 현상이 일어날수 있습니다.   
  
이것을 방지하기 위해서는 Connection을 재사용 해 주어야합니다.   
만약 default RestTemplate header에 {Connection : Keep-alive} 옵션을 준다면, RestTemplate 객체는 새로 생성되더라도 connection은 살아있는상태로 유지되어 계속해서 같은 포트에서 같은커넥션을 재사용하게됩니다.   

#### Connection pool 사용

RestTemplate에서 Custom HttpClient를 통해 Connection pool을 사용할수 있습니다.  

```java
HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory();
factory.setReadTimeout(5000);
factory.setConnectTimeout(3000);
HttpClient httpClient = HttpClientBuilder.create() .setMaxConnTotal(100)
            .setMaxConnPerRoute(5)
            .build();
factory.setHttpClient(httpClient);
RestTemplate restTemplate = new RestTemplate(factory);
```

---

### Apache HttpClient

```java
@Bean
public HttpClient httpClient()
{
    return HttpClientBuilder.create()
        .setConnectionTimeToLive(10, TimeUnit.SECONDS)
        .setMaxConnTotal(100)
        .setMaxConnPerRoute(5)
        .build();
}
```

default 설정으로, Connection pool을 사용하여 커넥션을 재사용합니다. (max-route : 5, max-connection : 10)   
멀티쓰레드 환경에서도 Thread-safe 하게 Connection pool을 공유하여 사용합니다.  

#### Configuration

- maxConnTotal : total 최대 connection 갯수
- maxPerRoute : ip 별 최대 connection 갯수
- ConnectionTimeToLive : 쓰레드 풀 내에서 Connection의 TTL

