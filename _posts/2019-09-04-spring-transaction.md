---
layout: post
comments: true
title: spring annotation 트랜잭션 & programming 트랜잭션
tags: [spring]
---

### 선언적 트랜잭션  
선언적 트랜잭션 이라함은 annotation을 이용한 트랜잭션으로, @Transactional을 사용한 트랜잭션을 의미합니다.  
@Transaction은 Sprign AOP를 통해 구현되어지는 방식으로 아래 어노테이션 선언을 하게되면 그림과같은 내부 로직이 구현됩니다. 
  
![1]({{ site.images | relative_url }}/posts/2019-09-04-spring-transaction/1.png)   
  
#### 특징
- 트랜잭션 로직과 비즈니스 로직을 분리하여 관리할수 있습니다.
- 유지보수가 쉽습니다.
- 다수의 트랜잭션을 관리할때 선호됩니다.
- AOP로 구현됨으로, 내부 메서드 transaction 으로는 사용이 불가능합니다.

---

### 프로그래밍 트랜잭션

스프링 프레임워크에서는 두가지 방식의 프로그래밍 트랜잭션구현 방법을 제공합니다.  
- TransactionTemplate  
- PlatformTrasactionManager 구현체 사용  
위 두가지 방법중, 스프링 팀에서는 Transaction Template을 사용하는것을 권장하고 있습니다.  
(https://docs.spring.io/spring/docs/current/spring-framework-reference/data-access.html#tx-prog-template)
  
  
#### 특징

- 소스코드에서 프로그래밍을 통해 트랜잭션을 관리 할수 있습니다.
- 트랜잭션 로직을 직접 하드코딩으로 관리 할 수 있습니다.
- 트랜잭션의 유연한 관리가 가능합니다.
- 다수의 트랜잭션을 관리하기는 어렵습니다.
- 상대적으로 적은양의 트랜잭션이 사용될때 사용하는것이 권장됩니다.

---
### 선언적 트랜잭션 vs 프로그래밍 트랜잭션
- 스프링 공식문서 제공 내용  
https://docs.spring.io/spring/docs/3.1.x/spring-framework-reference/html/transaction.html#tx-decl-vs-prog  
  
> Programmatic transaction management is usually a good idea only if you have a small number of transactional operations. For example, if you have a web application that require transactions only for certain update operations, you may not want to set up transactional proxies using Spring or any other technology. In this case, using the TransactionTemplate may be a good approach. Being able to set the transaction name explicitly is also something that can only be done using the programmatic approach to transaction management.
>  
> On the other hand, if your application has numerous transactional operations, declarative transaction management is usually worthwhile. It keeps transaction management out of business logic, and is not difficult to configure. When using the Spring Framework, rather than EJB CMT, the configuration cost of declarative transaction management is greatly reduced.
  
프로그램상에 트랜잭션이 적을때는 TransactionTemplate을 이용하는것이 좋습니다. (트랜잭션 프록시를 설정하지 않을수 있기때문)  
다수의 트랜잭션 작업이 있는경우 트랜잭션관리를 비즈니스 로직에서 제외시키고 구성할수 있기때문에 구성관리에 편리할수 있습니다.