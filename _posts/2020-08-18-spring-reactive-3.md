---
layout: post
comments: true
title: Spring WebFlux (Spring reactive-3)
tags: [spring, reactive, webflux]
---

### WebFlux

기존의 `Blocking` 기반의 Spring Web framework인 `Spring WebMVC`와 더불어 Spring 5.0 부터는 `Non-Blocking` 기반의 framework인 `Spring Webflux`를 지원하고 있습니다.  

Pivotal에서는 `Spring WebFlux`를 만든 이유에대해서 다음과 같이 설명하고 있습니다. 

> Why was Spring WebFlux created? 
> 
> Part of the answer is the need for a non-blocking web stack to handle concurrency with a small number of threads and scale with fewer hardware resources. Servlet 3.1 did provide an API for non-blocking I/O. However, using it leads away from the rest of the Servlet API, where contracts are synchronous (Filter, Servlet) or blocking (getParameter, getPart). This was the motivation for a new common API to serve as a foundation across any non-blocking runtime. That is important because of servers (such as Netty) that are well-established in the async, non-blocking space.
> 
> The other part of the answer is functional programming. Much as the addition of annotations in Java 5 created opportunities (such as annotated REST controllers or unit tests), the addition of lambda expressions in Java 8 created opportunities for functional APIs in Java. This is a boon for non-blocking applications and continuation-style APIs (as popularized by CompletableFuture and ReactiveX) that allow declarative composition of asynchronous logic. At the programming-model level, Java 8 enabled Spring WebFlux to offer functional web endpoints alongside annotated controllers.

내용을 요약해보자면 다음과 같습니다. 

- 고가용성을 위한 non-blocking Web 스택이 필요하다.
- Java8에서 람다 함수식을 지원하게되면서, non-blocking의 연속되는 Stream API를 구성 할 수 있는 기회가 생겼다.

최근 Spring의 업데이트사항으로 보면 이제는 너무나도 당연하게 된 `'MSA'`와 `'Reactive'`에 계속해서 힘을 실어주고 있는 모습으로 확인됩니다.  

앞으로도 적은 자원으로 고효율의 동시성을 낼 수 있는 구조로 설계된 `WebFlux`를 발전시켜 나가기 위해 계속해서 중점적으로 지원할 것으로 보입니다. 

---

### WebFlux vs WebMVC

위에서 `WebFlux`의 탄생 배경은 알아보았으니, 그렇다면 기존의 `WebMVC`와는 어떻게 다른지에 대해서 알아보도록 하겠습니다.

이미 Spring 개발을 하고 계신분들이라면 다들 아시겠지만, 현재 Spring에서는 `WebMVC`와 `WebFlux`를 완전 별도의 프로젝트로 지원 하고 있습니다. (사실상, 상호 보완적 관계 라기 보다는 다른 목적을 가진 프로젝트로 보셔도 무방합니다.)   

![1]({{ site.images | relative_url }}/posts/2020-08-18-spring-reactive-3/1.png) 

`WebFlux`와 `WebMVC`의 가장 큰 차이는 `'요청을 처리하는 쓰레드 모델'`이라고 볼 수 있습니다.  

결론을 먼저 말씀드리자면, `WebMVC`는 요청이 들어올때마다 쓰레드를 생성하는 반면에 (1 Request = 1 Thread) `WebFlux`는 요청이 들어오면 큐(Queue)에 두고 이벤트 루프에의해 Single-Thread 혹은 제한된 Thread (보통 core 수 만큼)로 동작하게됩니다.  

 
![2]({{ site.images | relative_url }}/posts/2020-08-18-spring-reactive-3/2.png)  

![3]({{ site.images | relative_url }}/posts/2020-08-18-spring-reactive-3/3.png)  

위와 같이 다른 처리 모델로 설계된 이유를 알아보면, 결국 `Blocking`과 `Non-Blocking`의 차이에 있다고 보실 수 있습니다.

상황을 하나 가정해 보도록 하겠습니다.  

> `A 서비스`는 `B 서비스`에 `Blocking` API를 요청합니다.  
> `B 서비스`는 요청에 대한 데이터를 모두 모아 응답을 보냅니다.   
> `A 서비스`는 `B 서비스`에서 응답이 올때까지 대기합니다.

`Blocking`의 특성상, 응답이 늦을수록 대기시간이 길어지기 때문에 그만큼 자원의 낭비가 커질 수 밖에 없습니다.  

따라서 `WebMVC`의 경우 요청에 대한 최종 처리를 빠르게 해 주는것이 중요하기에, 요청이 들어오면 쓰레드풀에 쓰레드를 바로 하나 할당해서 최대한 빠르게 응답을 줄 수 있도록 설계되어 있습니다.

다른 상황을 하나 가정해 보겠습니다.

> `A 서비스`는 `B 서비스`에 `Non-Blocking` API를 요청합니다.  
> `A 서비스`는 `C 서비스`에 `B 서비스`에 요청했던 내용을 기록하는 `Non-Blocking` API를 요청합니다.  
> `A 서비스`는 ... (다양한 일을 계속 하고 있습니다) 
>  
> `B 서비스`는 요청에 대한 데이터를 건건이 응답을 보냅니다.   
> `A 서비스`는 `B 서비스`에서 데이터가 올때마다 처리해줍니다.

`Non-Blocking`의 특성상, 응답속도에는 관계없이 무언가 요청을 하고도 다음 할일들 (해당 응답과 관계없는 작업)을 진행 할 수 있습니다.

따라서 `WebFlux`의 경우 요청에 대한 최종 처리를 '빠르게' 해 주는것이 가장 중요한것은 아닙니다. 실제로 Spring doc에서도 `WebFlux`의 성능관련한 내용에 대해 다음과같이 이야기 하고 있습니다.

> Performance has many characteristics and meanings. Reactive and non-blocking generally do not make applications run faster. They can, in some cases, (for example, if using the WebClient to run remote calls in parallel). On the whole, it requires more work to do things the non-blocking way and that can slightly increase the required processing time.
> 
> The key expected benefit of reactive and non-blocking is the ability to scale with a small, fixed number of threads and less memory. That makes applications more resilient under load, because they scale in a more predictable way. In order to observe those benefits, however, you need to have some latency (including a mix of slow and unpredictable network I/O). That is where the reactive stack begins to show its strengths, and the differences can be dramatic.

`Reactive`, `Non-Blocking`의 주요 목적은 '빠른 수행'이 아닌 적은수의 고정된 쓰레드와 적은 메모리를 사용면서도 확장에 유리한 `고탄력성`을 얻는데에 목적을 가지고 있습니다.

> '`WebMVC`도 서버 스케일 아웃/업을 통해 `탄력적`으로 운영 할 수 있는데?'  

라고 생각 할 수도 있으나, 커다란 Thread-pool에 수많은 Thread를 생성하는 `WebMVC`와는 달리 `WebFlux`는 제한된 Thread만을 가지고 `일정한` 처리를 하기 때문에, Thread의 `Context-swithching overhead`등 다양한 상황들을 배제하고 예측가능한 일정한 확장성을 가지고 `탄력적`으로 서비스를 운영 할 수 있다는 차이가 있다고 할 수 있겠습니다.

Spring DOC - WebFlux 원문(https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#spring-webflux)

---

### WebFlux 서비스 구현

`WebFlux`서비스 구현, 어렵지 않습니다. 

```groovy
    // build.gradle

	implementation 'org.springframework.boot:spring-boot-starter-webflux'

```

```java
    // FluxController.java

    @RestController
    public class FluxController {
        private final FluxService fluxService;

        @Autowired
        public FluxController(FluxService fluxService) {
            this.fluxService = fluxService;
        }

        @GetMapping("/flux")
        public Flux<String> getAll()
        {
            return fluxService.getAll();
        }
    }
```

```java
    // FluxService.java

    @Service
    public class FluxService {
        
        public Flux<String> getAll()
        {
            return Flux.just("하나", "둘", "셋", "넷", "다섯");
        }
    }

```

만약 `WebMVC`는 해보았는데, `WebFlux`로 구성된 코드는 처음보신 분이라면 물음표가 그려지실 수도 있습니다.  

`WebFlux`는 기존 `WebMVC`의 annotation을 그대로 지원하기에 `WebFlux`서비스를 구현하는것 자체는 전혀 어려움이 없으실거라 생각됩니다.

---
### WebFlux 통신

`WebFlux`에서는 reactive 통신을 위해 더이상 `RestTemplate`을 사용 할 수 없습니다.  
`blocking` 기반으로 구현된 `RestTemplate`을 reactive에 사용 할 수 없기때문에 대안으로, `WebFlux`에서는 Reactive 기반으로 구현된 `WebClient`를 제공합니다.  

```java
    Mono<Page> page = WebClient.create()
        .get()
        .uri("http://taes-k.github.io")
        .retrieve()
        .bodyToMono(Page.class);

    page.subscribe(p -> (...))
```

`Controller Test`를 할때, `MockMvc`를 사용했다면 이 또한 `Blocking` 기반으로 구현되어 있기 때문에 대안으로, `WebTestClient`를 사용해야 합니다.

```java
    WebTestClient testClient = WebTestClient.bindToController(
        new PageController(pageService))
        .build();
    
    testClient.get().uri("/page")
        .exchange()
        .expectStatus().isOk()
        .expectBody()
            .jsonPath("$").isArray()
            . ...
    
```


---


### 마무리

`spring-reactive`라는 주제로 3편으로 나누어 포스팅을 모두 작성하였습니다. 최대한 `실무에서 사용하는 Reactive`가 될 수 있도록 스스로 공부 및 정리 할 겸 포스팅으로 정리하면서 공유를 드렸는데 내용이 잘 전달 됬을지는 모르겠습니다만 개인적으로는 `Spring Reactive system`이 실무에서 사용 하기에 충분히 매력적으로 느껴졌습니다.

`Reactive system`은 이제 하나의 트렌드를 넘어 메인 스트림으로 자리잡았기 때문에 앞으로도 계속해서 성장해 갈 것이라 생각됩니다.

지금 당장 `reactive`를 실무에 적용하지 못한다고 해도, 미리 익혀보시기를 추천드립니다.


