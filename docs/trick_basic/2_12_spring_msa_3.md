---
layout: default
comments: true
title: 2.12 Spring MSA (3) - 기본구성
parent: 요령과 기본(Spring)
date: 2019.06.12
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

## 2.12 Spring MSA (3) - 기본구성
{: .no_toc }
이전까지 알아본 MSA와 Spring cloud를 사용해 이제 예제프로젝트를 본격적으로 진행해보도록 하겠습니다.  

## Contents List
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 예제프로젝트 기획

먼저 예제 프로젝트에대해 소개하도록 하겠습니다.  이전까지 진행하던 프로젝트와 마찬가지로 뉴스 페이지를 만드는데, 이때 로그인회원에 한하여 뉴스별 리뷰를 쓸수있는 시스템을 만들어 보도록 하겠습니다.   
  
간단한 서비스 구성은 다음과 같습니다.  

<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_3/servicie-diagram.png" style="height:300px; ">
</div>   
  


## 아키텍쳐 구성

위의 간략한 서비스 구성을 바탕으로 실제 서비스의 아키텍쳐를 구성해보도록 하겠습니다. 

<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_3/architecture.png" style="height:300px; ">
</div>   

***구동환경*** 

- Spring boot 2.1.5
- Spring cloud eureka
- Spring cloud zuul
- Spring cloud security
- H2

***프로젝트 구성***

|서비스|프로젝트|포트|
|:--:|:--:|:--:|
|웹|web-site|8080|
|유레카 서버|eureka-server|8761|
|API 게이트웨이|api-gateway|8090|
|인증 서버|auth-server|8095|
|유저 api|user-api|8000|
|뉴스 api|news-api|8005|
|리뷰 api|reviewe-api|8010|
|광고 api|advertising-api|8015|


## 아키텍쳐 기본 골조 개발

자 이제 앞으로 험난한 길이 될것을 예고 드립니다... 지금부터 위의 아키텍쳐를 기반으로 프로젝트 개발을 시작해보도록 하겠습니다. 먼저 프로젝트들을 만들어주는것 부터 시작해보도록 하겠습니다.  
  
***web-site(8080)***

```c
// build.gradle

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    compile 'org.apache.tomcat.embed:tomcat-embed-jasper'
    compile 'javax.servlet:jstl:1.2'
}
```  
  
  
```c
// application.properties

server.port = 8080
spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
```
  
  
  
```java
// ViewControllelr.java

@Controller
public class ViewController {

    @RequestMapping("/")
    public String index(Model model) {

        return "index";
    }
}
```

다음과같이 website를 위한 기본 구성만 해두도록 하겠습니다. 
  
***유레카 서버(8761)***

```c
// build.gradle

ext {
    set('springCloudVersion', "Greenwich.SR1")
}
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-server'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
    }
}
```  
 
  
```c
// application.properties

spring.application.name=msa-eureka-server
server.port = 8761
eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/
eureka.client.registerWithEureka=false // 자기자신을 서비스로 등록시키지 않음
eureka.client.fetchRegistry=false // 서비스 목록 로컬에 캐시 하지 않음
```  
  
  
```java
// MsaEurekaServerApplication.java

@EnableEurekaServer
@SpringBootApplication
public class MsaEurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }

}
```  
  
자 이로써 간단하게 유레카 서버 설정이 완료되었습니다!   
현재태에서 유레카서버를 동작시켜서 사이트에 접속해보면 다음과 같은 창이 뜨신다면 성공입니다. 이후에 유레카 클라이언트들을 등록하게되면 Instance 목록에 해당 서비스들이 등록될겁니다.

<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_3/eureka_server.png" style="height:400px; border:1px solid #d0d0d0; ">
</div>   


***유저 서비스(8000)***

```c
// build.gradle

ext {
    set('springCloudVersion', "Greenwich.SR1")
}
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    runtimeOnly 'com.h2database:h2'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
    }
}
```  
  
  
```c
// application.properties

spring.application.name=msa-user-service
server.port = 8000
eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/
```   
  
  
```java
// MsaUserApiApplication.java

@EnableDiscoveryClient
@SpringBootApplication
public class MsaUserApiApplication {

    public static void main(String[] args) {
        SpringApplication.run(MsaUserApiApplication.class, args);
    }

}
```  

```java
// UserController.java

@RestController
public class UserController {

    @RequestMapping("/")
    public String getUser() {

        return "user Information";
    }
}
```  

이렇게 유레카 클라이언트 중 하나인 user-api가 완성되었습니다. 이제 유레카 클라이언트 서비스가 완성됬으니 서비스를 구동시킨후 유레카 서버를 확인해보도록 하겠습니다.

<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_3/eureka_server_user_api.png" style="height:400px; border:1px solid #d0d0d0; ">
</div>   

위와같이 서비스가 정상작동되는 것이 확인된 MSA-USER-API 가 유레카 서버 인스턴스로 등록된것을 알수있습니다. 그럼 나머지 api 들도 위와같은 형태로 작성해 주도록 하겠습니다.  

[뉴스 서비스(8005)]   
[리뷰 서비스(8010)]   
[광고 서비스(8015)]   

나머지 서비스들까지 모두 등록시킨후 유레카 서버를 확인하면 모든 서비스들이 등록되어진것을 확인 할 수 있습니다.   

<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_3/eureka_server_all_api.png" style="height:400px; border:1px solid #d0d0d0; ">
</div>   


***API 게이트웨이(8090)***

이제 Spring zuul을 이용해 위에서 만든 4개의 api 서비스들을 하나의 게이트웨이를 통해 묶어주도록 차례입니다.  

```c
// build.gradle


ext {
    set('springCloudVersion', "Greenwich.SR1")
}
dependencies {
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-zuul'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
    }
}
```  
  
  
```c
// application.properties

spring.application.name=msa-api-gateway
server.port = 8090
eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/
zuul.routes.msa-user-api.path=/api/user/**
zuul.routes.msa-news-api.path=/api/news/**
zuul.routes.msa-review-api.path=/api/review/**
zuul.routes.msa-advertising-api.path=/api/advertising/**
```  
   
zuul.routes.<serviceId>.path 를 통해 해당 url로 오는 request들을 유레카서버에 등록되어 있는 service ID를 탐색하여 해당 서비스로 매칭시켜줄수 있습니다.  
  
   
```java
// MsaApiGatewayApplication.java

@EnableZuulProxy
@EnableDiscoveryClient
@SpringBootApplication
public class MsaApiGatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(MsaApiGatewayApplication.class, args);
    }

}
```

자 여기까지 완료되었다면, API Gateway가 벌써 준비가 완료되었으니 직접 확인해도록 하겠습니다.   

<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_3/news_api.png" style="height:400px; border:1px solid #d0d0d0; ">
</div>   

`http://localhost:8005/` 뉴스api 컨트롤러에서 설정해두었던 String값을 정상적으로 얻을수 있는것을 확인했습니다.  
  
<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_3/zuul_news_api.png" style="height:400px; border:1px solid #d0d0d0; ">
</div>   

zuul properties 설정에서 `zuul.routes.msa-news-api.path=/api/news/**` 설정에 따라서 `http://localhost:8090/api/news/` 로 접속한 url은 뉴스 api 서비스(msa-news-api)로 매칭되어 
게이트웨이 역할을 잘 하고있음이 확인됩니다.  

<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_3/zuul_advertising_api.png" style="height:400px; border:1px solid #d0d0d0; ">
</div>   

news 서비스 뿐만아니라 다른 서비스들 모두 정상적으로 작동이 잘 되고 있습니다.  


---

## <마무리>

우선적으로 아키텍쳐를 구상하고 기본 보일러 플레이트정도만 구성을 완료했습니다. 이제 다음 챕터부터 내용을 붙여나가면서 계속해서 msa 프로젝트를 진행해보도록 하겠습니다.  

---

## 샘플 프로젝트 
{: .no_toc }

위 프로젝트는 다음 링크에서 확인하실수 있습니다.  
<https://github.com/taes-k/spring-example/tree/master/spring-msa>


---
