---
layout: default
comments: true
title: 2.14 Spring MSA (6) -  리액티브
parent: 요령과 기본(Spring)
date: 2019.06.27
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

## 2.14 Spring MSA (6) -  리액티브
{: .no_toc }
처음 MSA 포스팅에서 일체형 서비스와 비교해 서비스간의 통신으로 인한 오버헤드가 단점으로 지적된다고 말씀드렸습니다. 이 단점을 해결하기위한 스프링 MSA에서의 리액티브는 어떻게 사용되는지 알아보도록 하겠습니다.

또하나의 문제점으로는 일체형에서는 컴포넌트간 통신으로 빠른 응답을 보였던것과 비교해, 각 서비스마다의 통신으로 인해 오버헤드가 생기면서 성능에대한 이슈가 당연히 나타났습니다. 이로인해 '리액티브'가 더 활성화를 띄게되면서 마이크로서비스의 비동기, 메세지기반 통신의 중요성이 높아지고 특히나 Spring 5.0 업데이트에서 Reactive 기반의 WebFlux가 출시되기도 하였습니다.  

## Contents List
{: .no_toc .text-delta }

1. TOC
{:toc}

---
## 리액티브 프로그래밍  

리액티브 프로그래밍에 대한 내용은 [1.5 Spring5 리액티브](https://taes-k.github.io/docs/trick_basic/1_5_about_spring_reactive/) 에서 한번 다루었지만, 다시한번 어떤식으로 동작하는것이 리액티브인가?를 알아보도록 하겠습니다.   

리액티브에서 중요한 개념은 '논블록킹'과 '비동기'입니다. 같은 의미가 아닌가? 하시는분도 계실텐데 정확히 어떤 서비스상의 차이가 있는지 알아보면서 정확한 개념을 정의해 보도록 하겠습니다.  
  

***논 블록킹***
논 블록킹을 알아보기전에 기본적으로 '블록킹' 시스템 구조를 알아보도로 하겠습니다.  
블록킹 : 
비동기 :  


***비동기***

동기 : 완료 Callback을 기다림.
비동기 : 완료 Callback을 기다리지 않음. (별도의 처리)


## 스프링 Webflux

스프링 Webflux는 mvc에서 무엇이 달라졌는가?

mvc WAS의 request 처리 구조

## rabbitMQ

중간 버퍼역할, 메시지 브로커 
rabbitMQ, kafka

## 리액티브 MSA 프로젝트

기존 프로젝트를 리액티브하게 변경








---

## <마무리>



---

## 샘플 프로젝트 
{: .no_toc }

위 프로젝트는 다음 링크에서 확인하실수 있습니다.  
<https://github.com/taes-k/spring-example/tree/master/spring-msa>


---
