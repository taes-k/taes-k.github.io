---
layout: default
comments: true
title: (요령과 기본) 2.1 WAS를 사용하는 이유
parent: 요령과 기본
date: 2019.05.24
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### 2.1 WAS를 사용하는 이유
{: .no_toc }
자바기반으로 웹 서비스를 개발하셨다면 배포할때 대부분 WAS를 사용해서 배포를 하는데요, 이번 챕터에서는 WAS를 사용하는 이유와 동작 프로세스에 대해 알아보도록 하겠습니다.  

## Contents List
{: .no_toc .text-delta }

1. TOC
{:toc}

---

### WAS (Web Application Server)

웹 애플리케이션 서버는 서버단에서 웹 애플리케이션을 동작 할 수 있도록 지원해주는 소프트웨어 미들웨어로써,  동적 서버 콘텐츠를 처리에 최적화 되어있어 정적 콘텐츠를 주로 제공하는 '웹서버'와는 구별되어집니다. '웹서버'는 다음 챕터에서 다루도록 하겠습니다.  
  
WAS가 기본적으로 제공하는 기능들은 다음과 같습니다.  
- 어플리케이션 실행환경 제공  
- 데이터베이스 접속 기능 제공
- 여러 트랜잭션 관리
- 비즈니스 로직 수행
  
  

---  

### WAS의 종류

Tomcat
아파치


WebLogic
BEA

Jetty

JBoss


### WAS의 동작 프로세스


### <마무리>
Reactive는 현재 기술 생태계에서 좋은 분위기를 가지고 있습니다. 하지만 단순히 그러한 인식만을 바탕으로 서비스에 Spring WebFlux를 도입하려는 시도보다는 WebFlux의 동작 구조와 이점을 정확히 이해하고 본인의 서비스에 적합한지를 한번더 확인 한 후에 적용을 한다면 분명히 큰 결과를 얻어내실수 있을것 같습니다.   

--- 

### 참조 문서
{: .no_toc }
Spring 5.1.6 release docs  
<https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux>


---
