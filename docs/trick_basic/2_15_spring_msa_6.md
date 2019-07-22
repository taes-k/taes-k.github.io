---
layout: default
comments: true
title: 2.15 Spring MSA (6) -  리액티브(WebFlux)
parent: 요령과 기본(Spring)
date: 2019.06.20
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

## 2.15 Spring MSA (6) -  리액티브(WebFlux)
{: .no_toc }
Spring 팀에서는 Spring framework5 부터 기존의 Spring MVC와는 별도로 리액티브를 위한 Spring WebFlux를 지원하고 있습니다. 이전챕터에서는 RabbitMQ를 통한 MSA에서의 리액티브 서비스를 구현해보았는데, 이번에는 Webflux를 통한 리액티브 서비스를 구현해보도록 하겠습니다.

## Contents List
{: .no_toc .text-delta }

1. TOC
{:toc}

---
## Webflux
Webflux에 대한 포스팅은 이전에 진행한적이 있기 때문에 다음에서 확인하실수 있습니다.   
[1.5 Spring5 리액티브](https://taes-k.github.io/docs/trick_basic/1_5_about_spring_reactive/)  
  
## Spring Webflux 동작구조 
Webflux에서는 기존의 탐캣이 아닌 비동기 논블록 구조에서 고효율을 보이는 Netty 서버를 기본적으로 사용합니다. 이러한 차이로인해 실제 서비스의 동작방식에 있어서 완전히 다르게 동작합니다.  
  
기존 MVC의 동작구조는 다음 다이어그램과 같습니다.  
<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_6/spring_mvc_work.png" style="height:300px; ">
</div>   

위와같이 request가 올때마다 쓰레드를 생성시켜 요청을 처리하는 서블릿 구조입니다.  
  
그렇다면 Webflux는 어떻게 다를까요?  
<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_6/spring_webflux_work.png" style="height:300px; ">
</div>   

WebFlux에서는 마치 메시지 브로커의 역할처럼 버퍼가 request를 담고 있다가 순차적으로 single-thread 기반으로 요청을 처리 할 수 있도록 해줍니다.   
  
일반적으로 블록킹 프로세스에서는 요청의 처리가 완료될때까지 응답을 돌려주지 않습니다. 따라서 많은 요청이 들어올때 순차적처리가 일어나게된다면 대부분의 응답을 대기후에 받게 될것입니다. 이를위해 멀티스레드를 통해 요청을 처리하는 방식으로 처리를 하게되는데 이 또한 쓰레드가 많아질수록 컨텍스트 스위칭, 쓰레드 경합이라는 오버헤드가 생기게 되고 시스템 자원 역시 더 좋은 성능의 자원이 필요하게 됩니다.   
  
WebFlux는 논블럭킹 기반의 프레임워크로, 모든 요청들에 대한 응답을 바로 줄 필요는 없습니다. 해당의 이유와 MSA의 확산으로 인해 WebFlux에서는 Netty를 기본 서버로 사용하여 적은 성능으로 논블럭킹 기반 최고의 효율을 줄 수 있게 지원하고 있습니다.  
  
## Mono & Flux
WebFlux를 시작하기전 알아야할것이 있습니다. 그것은 바로 webFlux를 구성하고 있는 리액터의 스트림객체인 'Mono'와 'Flux'입니다. 이 스트림 객체는 Publisher 인터페이스를 구현해 이전 챕터 메시지 브로커에서 다루었던 Publisher의 역할을 해줍니다.   
  
Mono와 Flux의 차이는 전송하는 데이터의 갯수의 차이에 있습니다. 이름에서 알수있듯 Mono는 0-1개의 데이터를 전달하며 Flux는 0-N개의 데이터를 전달할 수 있습니다.

## WebClient
일반적으로 Spring에서는 RestTemplate을 통해 서버간 통신을 진행했습니다. 하지만 RestTemplate은 블록킹 기반의 통신이라 넌블록킹 기반의 WebFlux서버와 통신할때는 맞지 않습니다. 따라서 Spring5부터는 WebFlux와 함께, WebClient인터페이스를 제공해 넌블록킹 클라이언트를 제공해주고 있습니다.  
  
## 리액티브 MSA 프로젝트

자 이제 Spring WebFlux를 이용해 직접 리액티브 서비스를 구현해보도록 하겠습니다.  이전 방식과 마찬가지로 리액티브로 동작할 트랜잭션을 다시 확인해보도록 하겠습니다.
```
뉴스 리뷰 작성 -> 뉴스 점수 증가
```
***news-api(:8005)***
가장먼저 우리가 해주어야 할 것은, webFlux를 사용하기위한 dependency 설정입니다.  
```c
// build.gradle

implementation 'org.springframework.boot:spring-boot-starter-webflux'
```  
  
Mono 객체를 이용한 Rest api `/review/add` 를 작성해 줍니다. 
```java
// NewsController.java

@RestController
public class NewsController {

    @RequestMapping("/")
    public String getNews() {
        return "news Information";
    }

    @RequestMapping("/review/add")
    public Mono<ResponseEntity> addReviewCount(@RequestParam(value="newsId") int newsId) {
        System.out.println("add news Review Count / news Id : "+newsId);
        return Mono.just(ResponseEntity.ok().build());
    }

}
```
  
해당 구현을 통해 넌블럭으로 review count를 증가시키는 news api 서버 구성이 완료되었습니다.

***review-api(:8010)***
review 서버에도 마찬가지로 webflux를 사용하기 위한 설정을 해 줍니다. 
```c
// build.gradle

implementation 'org.springframework.boot:spring-boot-starter-webflux'
```
  
넌블럭 클라이언트 통신을 위해 Webclient를 사용하여 news api를 호출해주도록 하겠습니다.  



---

## <마무리>



---

## 샘플 프로젝트 
{: .no_toc }

위 프로젝트는 다음 링크에서 확인하실수 있습니다.  
<https://github.com/taes-k/spring-example/tree/master/spring-msa>


---
