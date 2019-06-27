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
  
이전 예제프로젝트를 예를 들어서, 뉴스 리뷰를 적으면 해당뉴스의 점수가 높아져 상단에 노출되는 트랜잭션을 만든다고 가정해보도록 하겠습니다.  
  
***동기(Synchronous) + 블로킹(Blocking)***  
  
<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_5/syncronous_blocking.png" style="height:400px;">
</div>   


***비동기(Asynchronous) + 논블로킹(Non-blocking)***  
  
  <div style="text-align:center; margin:50px 0;">
  <img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_5/asyncronous_nonblocking.png" style="height400px;">
  </div>   
   
   
위에서 알아본 내용으로 블록킹과 동기에대해서 다음과 같이 정리 해 보겠습니다.  

> 동기 : 해당 작업에서 완료 Callback을 기다림  
> 비동기 : 해당 작업에서 완료 Callback을 기다리지 않음. (별도의 처리)    
>    
> 블록킹 : Callback 이후 제어권 반환  
> 논 블록킹 : 호출후, 제어권 바로 반환  
  
따라서 리액티브는, 호출후 제어권을 바로 돌려받고 해당 작업에서 Callback을 기다리지 않고 별도의 작업으로 처리하는 구조라고 이제 쉽게 이해가 가능하실것 같습니다.  

## 비동기 논블로킹 구현

위에서 리액티브에 대해 알아보았는데, 그렇다면 실제로 어떻게 구현이 되는가에대해서 알아보도록 하겠습니다. 이 비동기, 논블로킹을 구현하기위해 일반적으로 '메시지큐'가 이용됩니다. 

<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_5/message_queue.png" style="height400px;">
</div>   
  
메시지큐를 이용한 구조는 위의 다이어그램과 같이 메시지큐를 구독하고있는 서버에서는 데이터가 들어오면 해당 데이터를 받아볼수 있는 시스템으로 구현할수 있습니다.  
  
특히 이 '메시지큐'를 지원하는 오픈소스 미들웨어들(RabbitMQ, ActiveMQ, Kafka 등)이 제공되고있는데 이번예제에서는 RabbitMQ를 활용하여 진행해보도록 하겠습니다.  


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
