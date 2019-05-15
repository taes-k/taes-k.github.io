---
layout: default
comments: true
title: (요령과 기본) 1.3 Spring 의 Aspect
parent: 요령과 기본
date: 2019.05.15
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### 1.3 Spring의 Aspect
{: .no_toc }
이전 챕터에서 알아본 Spring의 IoC/DI와 더불어 Spring의 핵심 3요소중 하나인 AOP에 대해서 알아보려고 합니다. Aspect Oriented Programing 직역하면 '관점 지향 프로그래밍'이라고 하면 사실 어떤 프로그래밍 방식을 뜻하는지 알수가 없습니다. Spring에서 말하는 Aspect란 무엇인지 알아보도록 하겠습니다.

## Contents List
{: .no_toc .text-delta }

1. TOC
{:toc}

---

### 1.3.1 Aspect Oriented Programming with Spring
### AOP란?
먼저 스프링에서 말하는 AOP에 대해 정확히 알아보기 위해 스프링 공식 문서를 확인해보도록 하겠습니다.  

> Aspect-oriented Programming (AOP) complements Object-oriented Programming (OOP) by providing another way of thinking about program structure. The key unit of modularity in OOP is the class, whereas in AOP the unit of modularity is the aspect. Aspects enable the modularization of concerns (such as transaction management) that cut across multiple types and objects. (Such concerns are often termed “crosscutting” concerns in AOP literature.)  
>  
> One of the key components of Spring is the AOP framework. While the Spring IoC container does not depend on AOP (meaning you do not need to use AOP if you don’t want to), AOP complements Spring IoC to provide a very capable middleware solution.  
>  
> [원문보기](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop)  

- AOP란 OOP를 보완하는 구조.
- OOP에서는 클래스단위로 모듈화를 했다면 AOP에서는 '관점'으로 모듈화를 하는 구조.
- '관점'은 type과 object와는 무관하게 '관심사항'으로 모듈화를 가능하게 한다.
- 스프링에서 AOP는 스프링 IoC를 보완하며 뛰어난 미들웨어 솔루션을 제공한다.
  
즉, AOP는 클래스와 관계없이 여러 객체를 관통하는 '공통 관심 사항'들에 따라서 묶어서 관리 해 줄수있는 구조라고 할수 있을것 같습니다.  

---

### AOP 개념과 용어
AOP가 어떻게 사용되는지 알아보기 전에 사용되는 용어들을 먼저 정리해보도록 하겠습니다.  

|용어|설명|
|:--:|:--:|
|Aspect|여러 객체를 관통하는 '공통 관심 사항'을 구현한것을 의미합니다. Spring 에서는 설정을 통해 일반 클래스에 넣는 방식(schema-based approach) 혹은 어노테이션을 활용한 방식으로 클래스에 aspect를 줄수 있습니다.|
|Join point|특정 작업이 시작되는 시점을 나타내는 포인트로, 메서드 호출이나 예외발생 등의 시점들을 의미합니다.|
|Advice|특정 join point에서 실행되는 action을 뜻하며, 실행 시점에따라 'around', 'before', 'after'의 타입들을 가지고 있습니다.|
|Pointcut|Join point의 부분집합으로써, 실제 Advice가 실행되는 Join point들의 집합을 의미합니다.|
|Target object|advise가 적용되어질 타깃 객체를 의미합니다.|
|AOP proxy|Aspect를 구현하기위해 AOP 프레임워크에서 만들어낸 객체를 의미합니다.|
|Introduction|proxy 객체에 메소드나 필드를 추가한것을 의미합니다. |
|weaving|Aspect를 Target object에 적용하는것. 컴파일시, 로드타입시, 런타임시 적용시킬수 있으나 Spring에서는 런타임때 적용시킵니다.|

### Spring AOP의 목적 




--- 

### 참조 문서
{: .no_toc }
Spring 5.1.6 release docs  
<https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop>


---
