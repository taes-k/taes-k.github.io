---
layout: post
comments: true
title: 메세지 발행 트랜잭션 보장을 위한 outbox pattern
tags: [kafka, message, transaction, outboxpattern]
---

### 메세지 발행 트랜잭션 보장 필요상황

메세지를 발행하는 Producer를 구축할때 일반적으로 고민하는 이슈중 하나는 DB 와 메세지 발행의 트랜잭션 보장 방법 입니다. 단순히 메세지 발행만 하는 시스템일 경우에는 크게 고민이 되지 않을수 있겠지만 아래와 같은 예시상황에서는 DB 상태와 메시지 브로커에 발행된 메시지의 상태를 일관되게 유지해야 합니다.

1. 주문 처리  
고객이 주문 생성시, 주문 데이터는 DB에 저장시키고 동시에 주문발생 이벤트를 메시지 브로커에 발행하여 다른 시스템에서도 주문 상태를 업데이트할 수 있도록 해야 합니다.

2. 회원 가입 및 이메일 발송  
사용자가 회원 가입을 요청시, 사용자 정보를 DB에 저장시키고 동시에 회원 가입 이벤트를 메시지 브로커에 발행하여 다른 시스템에서도 추가적인 작업을 수행할 수 있도록 합니다.

3. 결제 처리  
고객이 결제 요청시, 결제 내역을 데이터베이스에 저장하고 동시에 결제발생 이벤트를 메시지 브로커에 발행하여 다른 시스템에서도 추가적인 결제 작업을 수행할 수 있도록 합니다.

위와같은 예시상황외에도 이벤트기반으로 구성되어있는 많은 시스템에서 DB와 메세지 발행 트랜잭션이 보장을 필요로하는 경우가 많습니다.

---

### DB와 메세지 발행이 트랜잭션 보장이 안된다면?  

앞에서 트랜잭션보장이 필요한 상황에 대해서 알아보았는데, 우선 트랜잭션 보장이 되지 않는다면 발생할 수 있는 상황에 대해서 알아보도록 하겠습니다. 우선 위에서 보여드린 예시중 주문처리 케이스에서 발생할수 있는 문제들로 예를 들어보겠습니다.

```java
@RequiredArgumentConstructor
public class OrderService {
    private final OrderRepository repository;
    private final KafkaTemplate<String, String> kafkaTemplate;

    @Transactional
    public void processOrder(Order order) {
        // 데이터베이스에 주문 정보 저장
        repository.saveOrder(order);

        // Kafka 메시지 브로커에 주문 완료 이벤트 발행
        publishOrderCompleteEvent(order.getOrderId());

        // 데이터베이스에 주문 상태 업데이트
        repository.updateOrderStatus(order);
    }

    private void publishOrderCompleteEvent(String orderId) {
        String kafkaTopic = "order-complete-topic";
        String messageKey = orderId;
        String messageValue = "OrderCompleted";

        kafkaTemplate.send(kafkaTopic, messageKey, messageValue);
    }
}
```

위 코드 수행을 하면서 발생할수 있는 문제에 대해 알아보도록 하겠습니다.  
먼저 `데이터베이스에 주문 상태 업데이트` 라인에서 에러가 발생하게된다면 `@Transactional`에 의해 DB에 저장된 주문정보는 rollback 되지만 kafka에 발행된 메세지는 그대로 남아있기때문에 해당 메세지를 사용하는 시스템에서는 실제로는 생성되지 않은 주문이 생성되었다고 메세지를 받게되면서 이후 처리에 장애를 일으킬수 있습니다.

다른 이슈로는 `Kafka 메시지 브로커에 주문 완료 이벤트 발행`시 `timeout` 오류가 발생하는 경우에도 마찬가지로 DB에 저장된 주문정보는 rollback 되지만 kafka에 메세지는 발행될수 있습니다. 

이와같이 db와 message를 함께쓰는 경우에는 트랜잭션을 보장하기 위한 방법을 고민 해야 합니다.

---

### Kafka 트랜잭션을 활용한 보장방법

가장 간단한 방법으로는 메세지 시스템에서 트랜잭션을 사용하는 방법입니다. 위 코드 예시에서 사용한 Kafka 0.11.0 version 이후부터 트랜잭션 기능을 제공합니다. 

다만, Kafka 에서 제공하는 트랜잭션은 DB 에서 사용하는 트랜잭션의 개념과는 조금 다른점이 있습니다. `commit-rollback`의 개념과는 다르게 Kafka 메세지는 발행자체가 되지 않도록 보장되는것이 아닌 추가 metaData message를 추가로 발행하여 해당메세지의 commit 여부를 알려주는것으로 Consumer가 commit상태의 메세지가 맞는지 판단 할 수 있도록 해주는 방식입니다.

![1]({{ site.images | relative_url }}/posts/2022-11-20-outbox-pattern/1.jpeg)  

따라서 트랜잭션을 제대로 적용하기 위해서는 consumer 에서도 별도 처리가 필요합니다. Kafka consumer의 `isolation.level`을 `read_committed`를 설정하여 commit된 메세지만 읽어들이도록 처리가 되어야하며 `read_uncommitted`를 사용하면 아직 commit 되지 않은 상태의 메세지도 읽을수 있기때문에 트랜잭션 처리가 정상적으로 되지 않을 수 있습니다.

따라서 카프카 트랜잭션을 사용할때에는 producer, consumer 양쪽에 설정값을 모두 신경 써줘야하고 처리 성능 하락과 브로커의 오버헤드 증가 라는 부수효과가 발생할수 있음을 고려해줘야합니다.

---

### outbox pattern을 이용한 트랜잭션 보장방법

위의 Kafka 트랜잭션 기능을 활용하지 않고도 확실하게 트랜잭션을 보장할수있는 다른 방법도 있습니다. `outbox pattern` 아키텍쳐 패턴을 이용한 방법인데 아래 그림을 보시면 바로 이해하실수 있으실것이라 생각됩니다.

![2]({{ site.images | relative_url }}/posts/2022-11-20-outbox-pattern/2.jpeg)  

DB에 event 발행을 위한 별도의 테이블을 두고, 트랜잭션 처리에서 같이 Insert를 해주는 방식으로 메세지 발행 필요한 데이터를 동일 DB 상에서 트랜잭션 보장되게 묶어서 처리해 보장해줄수 있습니다. 해당 테이블에 추가된 데이터는 별도 Producer를 통해 Polling 혹은 CDC 등을 통해 메세지 발행으로 연계 시킬 수 있습니다.

이와같은 방식의 장점은 아래와 같습니다.

- 간단하고 확실한 일관성 유지  
  DB 트랜잭션 기능을 사용하기때문에 사용하기도 쉽고 확실하게 일관성을 유지시켜줄수 있습니다.
- 어플리케이션 책임 분리  
  메세지 발행에대한 책임을 별도의 어플리케이션으로 분리시켜 비즈니스로직을 단순화 할 수 있습니다.
- 메세지 보장
  메세지 발행 대상 데이터가 DB에 저장되어있는 상태로 남아있기때문에 혹시나 메세지 발행이 실패하더라도 재시도 등의 처리를 통해 메세지 발행에 대한 보장을 해줄수 있습니다.

outbox 패턴을 통해 위와같은 장점을 갖고 트랜잭션으 보장하는 메세지 발행시스템을 구축 할 수 있습니다. 다만 아키텍쳐의 특성상 약간의 지연 발생이나 별도 producer의 추가 구현 필요성이 있기때문에 이러한점을 고려하여 적용해볼수 있을것입니다.

---

### 마무리

마이크로 서비스 아키텍쳐를 구현하면서 DB는 물론이고, 메세지 시스템은 빠져서는 안될 주요한 시스템이 되었습니다. 이 둘에 대한 트랜잭션 보장 필요성도 당연하게도 요구되는데 위 소개드린 방법에서 적절한 해결책을 찾으실수 있기를 바랍니다.

