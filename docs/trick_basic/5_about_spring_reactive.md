---
layout: default
comments: true
title: (요령과 기본) 1.5 Spring5 Reactive
parent: 요령과 기본
date: 2019.05.15
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### 1.5 Spring5 Reactive
{: .no_toc }
Spring에서는 "Reactive"라는 단어를 다음과 같이 정의하고 있습니다.   
  
> The term, “reactive,” refers to programming models that are built around reacting to change — network components reacting to I/O events, UI controllers reacting to mouse events, and others. In that sense, non-blocking is reactive, because, instead of being blocked, we are now in the mode of reacting to notifications as operations complete or data becomes available.  
  
"Reactive"는 non-blocking 입니다. 차단이 되지 않기때문에 작업이 완료되었거나 데이터가 사용가능해질때 알림을 받아 반응 할 수 있습니다. 이 Reactive의 개념은 Spring에서 만든것은 아니지만, Spring 5.0 부터 리액트 프로그래밍을 위한 Reative stream 라이브러리를 지원하고 있습니다.   

Spring5가 나오면서, 스프링에서는 Spring MVC와 병행하여 reactive-stack에서 Spring MVC를 대체하는 'Spring WebFlux'를 소개하고 있습니다. 이번 섹션에서는 Spring WebFlux에대해 알아보도록 하겠습니다.  


## Contents List
{: .no_toc .text-delta }

1. TOC
{:toc}

---

### <1.5.1 Reactive 프로그래밍>
### Reactive 프로그래밍이란?

Spring WebFlux에대해 알아보기전에, 먼저 리액티브 프로그래밍에대해 알아보려고 합니다. 위키피디아에서는 리액티브 프로그래밍에대해 다음과같이 설명하고 있습니다.  
> In computing, reactive programming is a declarative programming paradigm concerned with data streams and the propagation of change. With this paradigm it is possible to express static (e.g., arrays) or dynamic (e.g., event emitters) data streams with ease, and also communicate that an inferred dependency within the associated execution model exists, which facilitates the automatic propagation of the changed data flow.  
>  
> [원문보기](https://en.wikipedia.org/wiki/Reactive_programming)  

리액티브 프로그래밍은 데이터의 흐름과 변화의 전파에 중점을 둔 프로그래밍 패러다임 입니다.  
작성한 코드의 순서대로 진행되는 기존의 명령형 프로그래밍과는 다르게 데이터의 흐름을 먼저 정의하면 데이터의 변화 혹은 작업의 종료에 따라 반응하여 진행되는 프로그래밍 입니다.

### Reactive 프로그래밍의 목적
그렇다면 리액티브 프로그래밍은 어떤 목적에서 생겨났을까요?  서버에서의 리액티브 프로그래밍의 탄생은 리소스의 효율적 사용을 위함에 있었습니다. 리액트 이전에는 멀티쓰레드로써 병렬처리를 했지만, 쓰레드의 확장만으로는 CPU와 메모리의 제한이 있기에 비동기와 논블로킹의 새로운 프로그래밍 모델이 제안되었습니다.  

논블로킹은 일반적인 쓰레드를 통한 비동기 작업과는 다르게, 쓰레드를 점유하지 않고 작업을 수행하여 하나의 쓰레드 내에서 동시에 많은 작업을 수행 할 수 있습니다. 이와 같이 작업을 하는데 있어서 너무 많은 트래픽이 몰릴경우 문제가 발생 하거나 성능이 제대로 나오지 않을수 있기에 Back-pressure (배압, 역압)을 통해 요청의 갯수를 제한하여 고가용성을 보장해 줍니다.  

---  
  
### <1.5.2 Spring WebFlux>
### Spring MVC와 Spring WebFlux
  
--- 

### <마무리>

Spring MVC 구조에서 실제로 request를 어떻게 처리하는지 알아보았습니다. MVC구조가 단순히 Model, View, Controller 로써 구성된다는것 뿐만아니라 DispatcherServlet을 통해 어떠한 프로세스로 동작하고 컨텍스트의 계층구조까지 이해하고 개발을 할 수 있기를 바랍니다.  

--- 

### 참조 문서
{: .no_toc }
Spring 5.1.6 release docs  
<https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc>


---
