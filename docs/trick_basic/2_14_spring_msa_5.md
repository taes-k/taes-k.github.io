---
layout: default
comments: true
title: 2.14 Spring MSA (5) -  리액티브(RabbitMQ)
parent: 요령과 기본(Spring)
date: 2019.06.27
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

## 2.14 Spring MSA (5) -  리액티브(RabbitMQ)
{: .no_toc }
처음 MSA 포스팅에서 일체형 서비스와 비교해 서비스간의 통신으로 인한 오버헤드가 단점으로 지적된다고 말씀드렸습니다. 이 단점을 해결하기위한 스프링 MSA에서의 리액티브는 어떻게 사용되는지 알아보도록 하겠습니다.

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
  <img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_5/asyncronous_nonblocking.png" style="height:400px;">
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
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_5/message_queue.png" style="height:400px;">
</div>   
  
메시지큐를 이용한 구조는 위의 다이어그램과 같이 메시지큐를 구독하고있는 서버에서는 데이터가 들어오면 해당 데이터를 받아볼수 있는 시스템으로 구현할수 있습니다.  
  
특히 이 '메시지큐'를 지원하는 많은 오픈소스 미들웨어(RabbitMQ, ActiveMQ, Kafka 등)들이 제공되고있는데 이번예제에서는 RabbitMQ를 활용하여 진행해보도록 하겠습니다.  


## rabbitMQ

<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_5/rabbitmq_1.png" style="width:100%">
</div>   

rabbitMQ는 AMQP (Advanced Message Queueing Protocol) 표준 MQ 프로토콜을 구현한 오픈소스 미들웨어로써, 위와같이 메시지 전달의 중간과정을 담당해 주면서 메시지 브로커 소프트웨어라고 불립니다.  

대부분의 MQ들의 처리과정이 비슷하지만, rabbitMQ의 좀 더 자세한 메시지 처리과정을 살펴보면 다음과 같습니다.  

<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_5/rabbitmq_2.png" style="width:100%">
</div>   

메시지를 전달하는 프로듀서는, Queue로 메시지를 바로 전달하는것이 아니라 Exchange에 전달하여 일련의 설정에 따라 전달하고자 하는 Queue로 라우팅 시켜주는 역할을 하게되고 해당 Queue에 전달된 메시지는 Consumer에의해 consume되는 과정으로 메시지가 전달되게 됩니다.

## 스프링 클라우드 스트림

<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_5/spring_cloud_stream.png" style="width:100%">
</div>   

스프링 클라우드에서는 메시지기반 마이크로 서비스를 구현하기위한 '스프링 클라우드 스트림'을 지원하고있습니다. 물론 해당 프레임워크를 사용안하고 자바 라이브러리를 통해 메시지큐를 연결시킬수 있지만, 선언적으로 편리하고 통합된 환경으로 연결을 지원해 줍니다. 위의 다이어그램을 보시면 발신자의 Source(입구)의 OUTPUT이 RabbitMQ와 연결되어 메세지가 RabbitMQ로 전달되고 Sink(출구)의 Input이 RabbitMQ와 연결되어 RabbitMQ에서 나온 메세지를 받을수 있도록 해줍니다.   
이제 RabbitMQ와 스프링 클라우드 스트림을 이용해 메시지기반 MSA 서비스를 구축해보도록 하겠습니다.  

## 리액티브 MSA 프로젝트 - 메시지 브로커 서버

이제 마이크로서비스간에 메세지브로커를 구축하여 동작시켜보도록 하겠습니다. 먼저 메시지 브로커 역할을 할 Rabbit MQ를 먼저 설치해 주어야 합니다.
  
해당 설치과정은 따로 설명드리지 않겠습니다. 하단 url을 통해 손쉽게 설치후 구동 하실수 있습니다.  
<https://www.rabbitmq.com/download.html>
  
  rabbitMQ를 실행하시면 rabbitMQ manager plugin으로 MQ 상태를 확인하실수 있습니다. localhost:15672로 접속해보시면 접근이 가능합니다.  (default ID/PW : guest/guest)  
  
  <div style="text-align:center; margin:50px 0;">
  <img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_5/rabbitmq_manager.png" style="height:400px;border:1px solid #d0d0d0;">
  </div>   
  

***프로젝트 구성***

|서비스|프로젝트|포트|
|:--:|:--:|:--:|
|웹|web-site|8080|
|유레카 서버|eureka-server|8761|
|API 게이트웨이|api-gateway|8090|
|인증 서버|auth-server|8095|
|유저 api|user-api|8000|
|뉴스 api|news-api|8005|
|리뷰 api|reviewe-api|8010|
|광고 api|advertising-api|8015|
|메세지 브로커서버|message-broker-server|5672


## 리액티브 MSA 프로젝트 - producer

메시지 큐 방식으로 동작할 트랜잭션을 다시 확인해보도록 하겠습니다.   

> 뉴스 리뷰 작성 -> 뉴스 점수 증가   
  
위의 트랜잭션을 만든다고 할때, review api가 producer 역할로서, 리뷰가 작성되면 MQ로 메시지를 보내 news api에서 뉴스의 점수를 증가시켜주는 consumer  역할을 수행 할 수 있어야합니다. 그렇다면 review api에서 producer 역할을 할 수 있도록 구성해 보도록 하겠습니다.  

***review-api(:8010)***  

```c
// build.gradle

implementation 'org.springframework.cloud:spring-cloud-stream-reactive'
implementation 'org.springframework.cloud:spring-cloud-stream-binder-rabbit'
```

spring cloud stream rabbitMQ를 사용하기위해 dependency를 추가해줍니다.  

```c
// application.properties

spring.cloud.stream.bindings.write-review.destination=write_review
spring.cloud.stream.bindings.write-review.group=review
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```
위와같이 rabbitMQ 설정값들을 지정해줍니다. 최상단 설정의 경우, queue 이름을 지정해줌으로써 producer - consumer가 같은 이름의 큐를 link할수 있도록 해 줄 수 있습니다. 해당 설정은 write_review.review 라는 큐의 이름으로 설정이 됩니다.   
  
```java
// MessageProducer.java

@EnableBinding(MessageProducer.WriteReviewSource.class)
public class MessageProducer {

    public interface WriteReviewSource {
    String writeReview = "write-review";

        @Output(writeReview)
        MessageChannel writeReview();
    }
}

```   
  
실제 설정은 훨신 더 간단합니다. 위의 설정에서 지정해주었던 'write-review' 바인딩네임으로 Output message  채널 인터페이스를 만들어주고 바인딩해줌으로써 메시지 프로듀서의 설정으 끝입니다.  
  
```java
// ReviewController.java

@RestController
public class ReviewController {

    @Autowired
    WriteReviewSource writeReviewSource;

    @RequestMapping("/")
    public String getReview() {

        return "review Information";
    }

    @RequestMapping(value="/", method = RequestMethod.POST)
    public String setReview() {

    // set new review
        writeReviewSource.writeReview().send(MessageBuilder.withPayload("{seq : 13322}").build());
        return "write review";
    }
}

```
 
편의상, Controller에서 비즈니스 로직이 실행된다고 가정하고 진행하도록 하겠습니다. 위의 설정을통해 이제  `POST /` api를 호출하게되면 리뷰작성 로직이 완료된 이후  `application.properties`에서 설정한 queName의 큐에 메시지를 보낼것입니다. 한번 직접 확인해보도록 하겠습니다.  
  
(테스트에는 이전단계에 작성했던 인증과, API-GATEWAY 를 모두 이용합니다.)  

<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_5/call_api.png" style="height:300px;border:1px solid #d0d0d0">
</div>   
 
 <div style="text-align:center; margin:50px 0;">
 <img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_5/rabbitmq_manager_2.png" style="height:400px;border:1px solid #d0d0d0;">
 </div>   
 <div style="text-align:center; margin:50px 0;">
 <img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_5/rabbitmq_manager_3.png" style="height:400px;border:1px solid #d0d0d0;">
 </div>   
   
기존에 아무 연결이 없었던 상황에서 api 호출 이후 connection 과 channel이 생성된것을 확인 할 수 있습니다.  

## 리액티브 MSA 프로젝트 - consumer

자, 이제 consumer 측 작성을 진행해보도록 하겠습니다. 위에서 확인한대로  consumer는 news api 입니다.  

***news-api(:8005)***  

```c
// build.gradle

implementation 'org.springframework.cloud:spring-cloud-stream-reactive'
implementation 'org.springframework.cloud:spring-cloud-stream-binder-rabbit'
```  

```c
// application.properties

spring.cloud.stream.bindings.write-review.destination=write_review
spring.cloud.stream.bindings.write-review.group=review

spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```

```java
// messageConsumer.java

@EnableBinding(MessageConsumer.writeReviewSink.class)
public class MessageConsumer {

    @StreamListener(writeReviewSink.writeReview)
    public void writeReviewHandler(String message) {

        //add news point service logic
        System.out.println("add service : "+message);
    }


    public interface writeReviewSink {
        String writeReview = "write-review";

        @Input(writeReview)
        SubscribableChannel writeReviewListener();
    }
}

```

위의 producer 과정이 이해되셨다면  consumer는 더 간단하게 이해 되셨으리라 믿습니다. 해당 Input 으로 바인딩되어 있는 Listener를 통해 메시지가 큐에 들어온다면 해당 로직을 수행할것입니다. 제대로 메시지를 받아와서 동작하는지 확인해보도록 하겠습니다.  
   
   
<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_5/message_queue_result.png" style="border:1px solid #d0d0d0;">
</div>   
   
review-api 서버의 review 작성 api를 수행하면 news-api서버의 리스너가 동작하여 콘솔에 메시지를 출력하는것을 확인 할 수 있습니다. 이 메시지에 뉴스의 id값을 넘겨 로직내에서 해당 뉴스의 점수를 올린다면 원하는 로직을 구현 할 수 있을 것입니다.   

---

## 전체구조

<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_5/total_diagram.png" style="height:150px;border:1px solid #d0d0d0;">
</div>  

---

## <마무리>
MSA를 더 마이크로하게 만들어주는 메시지큐를 통한 리액티브시스템을 구현해 보았습니다. 해당 구현을 통해 단순히 비동기-논블럭  서비스를 구현한것 뿐만 아니라 해당 서버간의 의존도가 확실히 분리되었습니다. 다만 해당 시스템의 문제점이 한가지 있다면, 에러에대한 트랜잭션인데 해당 내용은 다음 포스팅때 다루어보도록 하겠습니다.   


---

## 샘플 프로젝트 
{: .no_toc }

위 프로젝트는 다음 링크에서 확인하실수 있습니다.  
<https://github.com/taes-k/spring-example/tree/master/spring-msa>


---
