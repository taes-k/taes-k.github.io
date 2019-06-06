---
layout: default
comments: true
title: (요령과 기본) 1.5 Spring5 리액티브
parent: 요령과 기본
date: 2019.05.21
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

## 1.5 Spring5 Reactive
{: .no_toc }
이번챕터에서 다룰 Spring Reactive는 제가 실무에서는 다루어보지 못한 기술입니다. 하지만 Spring5가 나오면서 가장 열정적으로 소개하고있는 기술이기에 정리를 해보려고 합니다.  
  
Spring에서는 "Reactive"라는 단어를 다음과 같이 정의하고 있습니다.   
  
> The term, “reactive,” refers to programming models that are built around reacting to change — network components reacting to I/O events, UI controllers reacting to mouse events, and others. In that sense, non-blocking is reactive, because, instead of being blocked, we are now in the mode of reacting to notifications as operations complete or data becomes available.  
  
"Reactive"는 non-blocking 입니다. 차단이 되지 않기때문에 작업이 완료되었거나 데이터가 사용가능해질때 알림을 받아 반응 할 수 있습니다. 이 Reactive의 개념은 Spring에서 만든것은 아니지만, Spring 5.0 부터 리액트 프로그래밍을 위한 Reative stream 라이브러리를 지원하고 있습니다.   

Spring5가 나오면서, 스프링에서는 Spring MVC와 병행하여 reactive-stack에서 Spring MVC를 대체하는 'Spring WebFlux'를 소개하고 있습니다. 이번 섹션에서는 Spring WebFlux에대해 알아보도록 하겠습니다.  


## Contents List
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## <1.5.1 Reactive 프로그래밍>
## Reactive 프로그래밍이란?

Spring WebFlux에대해 알아보기전에, 먼저 리액티브 프로그래밍에대해 알아보려고 합니다. 위키피디아에서는 리액티브 프로그래밍에대해 다음과같이 설명하고 있습니다.  
> In computing, reactive programming is a declarative programming paradigm concerned with data streams and the propagation of change. With this paradigm it is possible to express static (e.g., arrays) or dynamic (e.g., event emitters) data streams with ease, and also communicate that an inferred dependency within the associated execution model exists, which facilitates the automatic propagation of the changed data flow.  
>  
> [원문보기](https://en.wikipedia.org/wiki/Reactive_programming)  

리액티브 프로그래밍은 데이터의 흐름과 변화의 전파에 중점을 둔 프로그래밍 패러다임 입니다.  
작성한 코드의 순서대로 진행되는 기존의 명령형 프로그래밍과는 다르게 데이터의 흐름을 먼저 정의하면 데이터의 변화 혹은 작업의 종료에 따라 반응하여 진행되는 프로그래밍 입니다.

## Reactive 프로그래밍의 목적
그렇다면 리액티브 프로그래밍은 어떤 목적에서 생겨났을까요?  서버에서의 리액티브 프로그래밍의 탄생은 리소스의 효율적 사용을 위함에 있었습니다. 리액트 이전에는 멀티쓰레드로써 병렬처리를 했지만, 쓰레드의 확장만으로는 CPU와 메모리의 제한이 있기에 비동기와 논블로킹의 새로운 프로그래밍 모델이 제안되었습니다.  

논블로킹은 일반적인 쓰레드를 통한 비동기 작업과는 다르게, 쓰레드를 점유하지 않고 작업을 수행하여 하나의 쓰레드 내에서 동시에 많은 작업을 수행 할 수 있습니다. 이와 같이 작업을 하는데 있어서 너무 많은 트래픽이 몰릴경우 문제가 발생 하거나 성능이 제대로 나오지 않을수 있기에 Back-pressure (배압, 역압)을 통해 요청의 갯수를 제한하여 '고가용성'을 보장해 줍니다.  
  
좋은 성능을 보인다기 보다는 '고가용성'의 단어를 쓴 이유는 non-blocking을 통해 애플리케이션의 실행속도의 퍼포먼스가 좋아진다는 아니기 때문입니다. 오히려 속도적인 성능은 더 나빠질 수도 있습니다. 다만, 작은 고정된 수의 스레드와 적은 메모리로 최대한의 효율을 내면서 확장 할수 있다는 의미입니다.

---  
  
## <1.5.2 Spring WebFlux>
## Spring MVC와 Spring WebFlux

이전챕터였던 Spring MVC 공식문서를 보셨다면 아시겠지만, Spring WebFlux 대부분의 구조 및 설정이 Spring MVC와 동일하다고 명시되어 있습니다. 가장 큰 차이점은 MVC에서는 애플리케이션이 스레드를 차단할수 있다는 가정하에 큰 스레드풀을 가지고 있고, WebFlux에서는 스레드가 차단되지 않아 적은 스레드풀을 사용하여 request들을 처리한다는 점 일것입니다.    
  
<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/about_spring_reactive/mvc_same.png" style="height:250px; border:1px solid #d0d0d0;">
</div>  
   
그러면 어떤상황에서 Spring WebFlux 모델을 사용할수 있을까요? 이에 대한 답변또한 Spring 에서 답변해주고 있습니다.  
  
<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/about_spring_reactive/mvc_webflux_diagram.png" style="height:350px; border:1px solid #d0d0d0;">
</div>  
  
Spring MVC와 Spring WebFlux 는 서로 연속성과 일관성을 유지되도록 설계되었으며 각 영역에 특화된 옵션들을 지원해주고 있습니다. Spring Webflux를 도입하는데 고려할 사항으로는 다음과 같습니다.  
  
- 이미 잘동작하는 Spring MVC 어플리케이션을 가지고 있다면 변경 할 필요가 없다.  
- non-blocking 웹을 이미 운영하고 있다면 Spring WebFlux는 도움이 될것이다.  
- Java8 람다 또는 코틀린과 함께 사용하기위한 가볍고 기능적인 웹프레임워크에 관심이 있다면 Spring WebFlux가 적합하다.
- 마이크로서비스 아키텍쳐에서 Spring MVC 혹은 다른 Spring WebFlux와 함께 사용가능하다.
- JPA,JDBC 같은 blocking 기반 지속성 api를 사용하거나 네트워킹 API를 사용하는경우 Spring MVC를 사용하는것이 좋다.
- Spring MVC에서 원격서비스를 사용하는 경우 reactive WebClient 사용을 시도를 추천한다.
- 규모가 큰 팀인 경우에는 non-blocking, 기능및 선언적 프로그래밍으로 전환하는데 있어서의 러닝커브를 주의한다.  
  
비동기-논블록킹 리액티브 서비스를 만들고자 할때와 특히 서비스간의 호출이 잦은 마이크로서비스 아키텍쳐이신 상황에서 NodeJS나 기타 프레임워크가 아니라 Spring을 계속해서 사용하시고 싶으시다면 Spring Webflux의 도입을 고려해보시면 좋을것 같습니다.  


---

## Spring Webflux  

먼저 Spring WebFlux의 구조부터 알아보도록 하겠습니다. Spring MVC의 경우 서블릿 컨테이너와 서블릿을 기반으로 웹 추상화 계층을 제공 했는데 WebFlux는 서블릿컨테이너 + 네트워크 어플리케이션 프레임워크(Netty, Undertow)를 함께 지원해 Reactive Stream 기반으로 웹 추상화계층을 제공합니다.  
  
WebFlux에는 reactive를 지원하기 위해 다음과 같은 기본 지원들이 들이  `spring-web` 모듈에 포함되어 있습니다.  
- HttpHandler : non-blocking I/O 와 Reactive Stream을 통한 HTTP Request 처리를 위한 기본 규약으로 Reactor Netty, Undertow, Tomcat, Jetty, and any Servlet 3.1+ container가 포함되어 있음
- WebHandler API : request process를 위한 좀더 높은수준의 범용 웹 API  
  

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/about_spring_reactive/webfluxProcess.png" style="height:350px; border:1px solid #d0d0d0;">
</div>  
  
위의 process 구조도를 보시면 MVC와 크게 다르지 않습니다. 다만, MVC에서 가지고 있던 FrontController구조에서의 DispatcherServlet가 WebHandler가 대신 수행합니다. 또한 Result를 위한 Handler가 따로있어서 ResponseEntityResultHandler, ServerResponseResultHandler, ResponseBodyResultHandler, ViewResolutionResultHandler Handler의 타입에따라 return을 보내줍니다.  
  
|type|return value|
|:--:|:--|
|ResponseEntityResultHandler|ResponseEntity, typically from @Controller instances|
|ServerResponseResultHandler|ServerResponse, typically from functional endpoints|
|ResponseBodyResultHandler|Handle return values from @ResponseBody methods or @RestController classes|
|ViewResolutionResultHandler|CharSequence, View, Model, Map, Rendering, or any other Object is treated as a model attribute|
  

--- 

## Spring WebFlux 프로젝트 구현해보기

지금가지 WebFlux에대해서 알아보았는데, 그렇다면 실제 예제 WebFlux 프로젝트를 구현해보도록 하겠습니다.  
  
- 구동환경  
Spring boot 2.1.5  
  
- 디렉토리 구조 
  
<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/about_spring_reactive/webflux_directory.png" style="height:400px; border:1px solid #d0d0d0;">
</div>  
  
- 소스   
  
0\. pom.xml

```c
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>

```  
  
1\. Controller 

```java
@RestController
public class ExampleController {

    @Autowired
    ExampleService exampleService;

    @GetMapping("/example")
    public Flux getExample() {
        Flux examples = Flux.just(exampleService.getSampleExmaple1(),
        exampleService.getSampleExmaple2(),
        exampleService.getSampleExmaple3(),
        exampleService.getSampleExmaple4())
        .doOnNext(el -> System.out.println("getExample : "+el));

        return examples;
    }
}


```  
  
2\. Service

```java
@Service
public class ExampleService {

    public String getSampleExmaple1() {
        return "Example1";
    }

    public String getSampleExmaple2() {
        return "Example2";
    }

    public String getSampleExmaple3() {
        return "Example3";
    }

    public String getSampleExmaple4() {
        return "Example4";
    }
}

```  

3\. 결과 (api)

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/about_spring_reactive/webflux_result.png" style="height:250px; border:1px solid #d0d0d0;">
</div>  

4\. 결과 (콘솔)

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/about_spring_reactive/webflux_console.png" style="height:250px; border:1px solid #d0d0d0;">
</div>  
  
위의 프로젝트 예제가 Spring WebFlux를 잘 나타낸 예제라고는 말씀을 못드리지만, 일반적인  Spring MVC 구조와 크게 다르지않게 프로젝트를 구성하여 이용하실수 있습니다.   
  
- 샘플 코드
<https://github.com/taes-k/spring-example/tree/master/spring-webflux-example>


---

## <마무리>
Reactive는 현재 기술 생태계에서 좋은 분위기를 가지고 있습니다. 하지만 단순히 그러한 인식만을 바탕으로 서비스에 Spring WebFlux를 도입하려는 시도보다는 WebFlux의 동작 구조와 이점을 정확히 이해하고 본인의 서비스에 적합한지를 한번더 확인 한 후에 적용을 한다면 분명히 큰 결과를 얻어내실수 있을것 같습니다.   

--- 

## 참조 문서
{: .no_toc }
Spring 5.1.6 release docs  
<https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux>


---
