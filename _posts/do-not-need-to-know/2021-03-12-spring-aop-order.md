---
layout: post
comments: true
title:  몰라도 되는 Spring - AOP Order
tags: [spring, aop, proxy]
---

### 중첩 AOP

동일한 조인포인트에 여러개의 Advice가 지정된 경우, 여러개의 AOP가 같은 시점에 동작하게 됩니다.  
특히나, Spring framework 에서 정의한 AOP가 사용 될 때 자연스럽게 중첩 AOP되는 경우가 많이 있는데, 어떤식으로 동작하는지 그림으로 알아보도록 하겠습니다.

![1]({{ site.images | relative_url }}/posts/2021-03-12-spring-aop-order/1.png)

Spring에서 자주 사용하는 `@Transactional` 어노테이션을 사용하는경우 `proxy`를 통해 위와같이 수행되게 되는데,   
위 그림처럼 `Trasnaction Advisor` 다음에 `Custom Advisor`가 위치하면, `Custom Advisor`또한 `Transaction`에 포함되는것을 그림으로 확인 하실 수 있습니다.  

위와같은 이유로 인해 중첩 AOP 사용시, `Advisor`가 수행되는 시점은 굉장히 중요 할 것입니다.  
수행 순서에 따라 결과 데이터가 다를 수도 있고, 트랜잭션 포함여부가 달라 질 수 있습니다.

---

### AOP 순서 설정

AOP의 실행 순서는 결국 먼저 수행 될 수록 다음에 실행되는 `Advisor`를 감싸는 형태가 되어지는 것인데, 실행순서는 `Advice` Ordering rule 에 의해 정의되어집니다.  
실행순서를 정의해주는 것은 `@Order` 어노테이션을 통해 간단하게 설정이 가능합니다.

```java
@Aspect
@Order(1)
@Component
public class FirstAOP {
	@Around("execution(* com.example.demo.SampleService.createSample(..))")
	public Object execute(ProceedingJoinPoint joinpoint) throws Throwable {
		
		System.out.println("FirstAOP - Before");
        //----------------------------------------------------------------------
        Object result = joinpoint.proceed();
        //----------------------------------------------------------------------

		System.out.println("FirstAOP - After");

		return result;
	}
}

@Aspect
@Order(2)
@Component
public class SecondAOP {
	@Around("execution(* com.example.demo.SampleService.createSample(..))")
	public Object execute(ProceedingJoinPoint joinpoint) throws Throwable {
		
		System.out.println("SecondAOP - Before");
        //----------------------------------------------------------------------
        Object result = joinpoint.proceed();
        //----------------------------------------------------------------------

		System.out.println("SecondAOP - After");

		return result;
	}
}
```

위와같이 `Order`를 선언함으로써 원하는대로 advice의 실행 순서를 정의 할 수 있습니다. 

```
// 실행결과

FirstAOP - Before
SecondAOP - Before
create Sample
SecondAOP - After
FirstAOP - After

```

여기서 주의할 점은 `@Order(1)` 순서가 낮을수록 우선순위가 높다는 것입니다.  
`Order`를 설정하지 않을경우 내부적으로 순서를 판단해서 동작시키기 때문에 원하는대로 `AOP`를 수행하기위해서는 `순서` 또한 중요한 설정 요소임을 잊으셔서는 안됩니다.

---

### 참고문서

- Spring doc : [https://docs.spring.io/spring-framework/docs/current/reference/html/core.htm](https://docs.spring.io/spring-framework/docs/current/reference/html/core.htm)
