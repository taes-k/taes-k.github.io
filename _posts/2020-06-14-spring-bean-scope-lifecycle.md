---
layout: post
comments: true
title: Scope 에 따른 Spring bean life-cycle
tags: [spring, mysql]
---

### Spring bean

`bean`의 life-cycle을 알아보기에 앞서서 `bean`의 개념부터 간단하게 알아보도록 하겠습니다.  
(해당 포스팅 내용은 정확한 내용 전달을 위해 Spring doc 내용을 발췌 번역한 내용임을 알립니다. - [docs.spring.io](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans))

`bean`은 SpringFramework에서 가장 중추적인 역할을 하는 `Spring IoC Contaienr`에서 관리되는 객체를 말합니다. `Spring IoC Container`는 `bean`의 initializing, configure, assembling, destroy 등 생성부터 소멸까지의 통칭 `life-cycle`을 관리해 줍니다.

---

### Spring bean scope

`bean`을 선언할때 각 객체의 Scope(범위 : 생명주기 범위)를 지정 할 수 있습니다.

다음은 spring docs에서 설명하고 있는 각 scope들에 대한 설명입니다.

|scope|description|
|:--|:--|
|singleton|(Default) Scopes a single bean definition to a single object instance for each Spring IoC container.
|prototype|Scopes a single bean definition to any number of object instances.|
|request|Scopes a single bean definition to the lifecycle of a single HTTP request. That is, each HTTP request has its own instance of a bean created off the back of a single bean definition. Only valid in the context of a web-aware Spring ApplicationContext.|
|session|Scopes a single bean definition to the lifecycle of an HTTP Session. Only valid in the context of a web-aware Spring ApplicationContext.|
|application|Scopes a single bean definition to the lifecycle of a ServletContext. Only valid in the context of a web-aware Spring ApplicationContext.|
|websocket|Scopes a single bean definition to the lifecycle of a WebSocket. Only valid in the context of a web-aware Spring ApplicationContext.|

위에서 정의된 기본 scope들 외에서 사용자가 직접 custom한 scope를 정의하여 사용 할 수도 있습니다.  

아래에서는 주로 사용되는 `singleton`과 `prototype` scope에서의 bean의 life-cycle을 알아보도록 하겠습니다.


---

### Singleton bean life-cycle

가장먼저, Spring bean의 기본 scope인 `singleton-bean`의 life-cycle에 대해서 알아보도록 하겠습니다. 위의 설명과 같이 `singleton-bean`은 `Spring IoC container`에서 오로지 하나의 객체만 존재하며 컨테이너의 생성 및 소멸 시점과 함께 빈이 생성되고 소멸됩니다.  


![1]({{ site.images | relative_url }}/posts/2020-06-14-spring-bean-scope-lifecycle/1.png)  


`prototype-bean`은 `singletone`과는 다르게 DI 요청이 있을때마다 새로운 객체가 생성되어 `Spring IoC container`에서 여러개의 객체가 존재 할 수 있습니다.  

---

### Prototype bean life-cycle

그렇다면 `prototype-bean`의 소멸은 어떻게 이루어지게 될까요?  
사실 이 의문이 이번 포스팅을 쓰게된 이유였습니다. Spring 에서는  `prototype-bean`의 life-cycle에 대해서 다음과같이 설명하고 있습니다.

>  In contrast to the other scopes, Spring does not manage the complete lifecycle of a prototype bean. The container instantiates, configures, and otherwise assembles a prototype object and hands it to the client, with no further record of that prototype instance. Thus, although initialization lifecycle callback methods are called on all objects regardless of scope, in the case of prototypes, configured destruction lifecycle callbacks are not called. The client code must clean up prototype-scoped objects and release expensive resources that the prototype beans hold. To get the Spring container to release resources held by prototype-scoped beans, try using a custom bean post-processor, which holds a reference to beans that need to be cleaned up.

해당문서 상에서 확인할수 있듯이, `Spring IoC container`는 `prototype-bean`의 경우 `bean`생성의 과정 이후 소멸의 과정에는 관여를 하지 않습니다. 이는 `bean` 생성 이후에 객체의 소멸의 과정은 `Garbage Collector`에 의해서 처리되게끔 설계된 구조입니다.  

> client code must clean up prototype-scoped objects and release expensive resources that the prototype beans hold

하지만 위 본문에서는 다음과같은 내용이 있어 혼란스러울 수도 있습니다.  
위에서 설명드렸던 'GC 에 의해 처리된다' 라는 말과 다르게 작업자의 코드에서 `prototype-bean`를 소멸시켜 비싼 자원을 해제 시켜야한다고 명시하고 있는데, 해당 객체가 `DB Connection pool` 과 같은 자원을 들고 있는 경우라면 해당 `prototype-bean`은 `GC`에의해 소멸되지 않을것입니다. 

이렇게 `GC`에 의해서 삭제되지 않는 객체일 경우 `custom bean post-processor`를 구현하여 객체의 소멸을 직접적으로 관리 할 수 있습니다.  

한가지 더 주의하셔야 할 점이 있습니다.  
위의 구조들을 모두 이해하셨다면 너무나도 당연하게 생각하시겠지만, 만약 `singleton-bean` (A) 에서 `prototype-bean` (B)을 DI 받아 사용한다면 해당 B는 A가 생성될때 함께 `한번` 생성되고, A가 소멸될때 B또한 소멸 될 수 있을 것입니다.  


![2]({{ site.images | relative_url }}/posts/2020-06-14-spring-bean-scope-lifecycle/2.png)  


---

