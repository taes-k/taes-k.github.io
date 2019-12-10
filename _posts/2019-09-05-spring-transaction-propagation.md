---
layout: post
comments: true
title: spring transaction propagation
tags: [spring]
---

### Transaction propagation(전파)
Transaction 사용시 트랜잭션이 겹치게될때 (트랜잭션 내에서 다른 트랜잭션 서비스를 호출시) 트랜잭션처리를 어떻게 해야하는지에 대해 각각의 트랜잭션마다 propagation(전파범위) 옵션을 설정하여 지정해줄수 있습니다.
  
### 종류
- REQUIRED  
- REQUIRES_NEW  
- SUPPORTS  
- NOTSUPPORTED  
- MANDATORY  
- NEVER  
- NESTED   

### propagation 설정  
일반적으로 spring에서 사용하는 선언적 방식의 annotation에서 propagation을 설정해주는 방법은 다음과 같습니다.  

```
@Transactional( propagation = Propagation.REQUIRED)
```

`@Transactional`에 위에서 알아본 propagation 설정 옵션으로 간단하게 Transaction 끼리 중첩 되었을때 트랜잭션의 범위를 설정해줄수 있습니다.  
  
### propagations options
**1\.REQUIRED (Default)**  
부모 트랜잭션 내에서 실행하며 부모 트랜잭션이 없을경우 새로운 트랜잭션 생성 합니다.  
> A {  B { }  <ERROR\> }  
> -> A,B 모두 롤백  
> 
> A {  B { <ERROR\> }  }  
> -> A,B 모두 롤백  
  
**2\.REQUIRES_NEW**  
부모 트랜잭션을 무시하고 무조건 새로운 트랜잭션 생성 합니다.    
> A{ B{ } <ERROR\> }  
> -> A만 롤백  

**3\.SUPPORTS**  
부모 트랜잭션이 있다면, 부모 트래잭션 내에서 실행하며 없다면 트래잭션 없는 상태로 수행 합니다.  
> A{ B{ } <ERROR\> }  
> -> A,B 모두 롤백

**4\.NOT_SUPPORTS**  
트랜잭션 없는 상태로 수행하며, 부모 트랜잭션이 있다면 일시정지 합니다.  
> A{ B{ } <ERROR\> }  
> -> A 롤백

**5\.MADATORY**  
부모 트랜잭션 내에서 실행되며, 부모 트랜잭션이 없다면 에러를 발생시킵니다.  
> A{ B{ } <ERROR\> }  
> -> A,B 모두 롤백  
> 
> B{}  
> -> 에러

**6\.NEVER**   
트랜잭션 없는 상태로 수행하며, 부모 트랜잭션이 있다면 에러를 발생시킵니다.  
> A{ B{} }  
> -> 에러

**7\.NESTED**  
부모 트랜잭션 내에서 실행시 별개로 커밋되거나 롤백가능, 없을경우 REQUIRED와 동일하게 작동합니다.  
> A{ B{ } <ERROR\> }  
> -> A 롤백  
> 
> A{ B{ <ERROR\> }}  
> -> A,B 모두 롤백
