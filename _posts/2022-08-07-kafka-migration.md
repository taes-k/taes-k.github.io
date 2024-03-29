---
layout: post
comments: true
title: 운영중인 Kafka 브로커를 안전하게 전환한 사례 공유
tags: [kafka, migration, broker]
---

### 카프카 마이그레이션 시도의 이유

최근 일 3,000만 이상의 대용량 Kafka 메세지가 발행되는 신규 Producer 시스템을 리빌딩하는 프로젝트를 성공적으로 마무리했습니다.  
이 프로젝트를 진행하면서 Kafka 또한 운영중인 상태에서 멈춤 없이 신규 Cluster로 전환 할 수 있었는데, 안전하게 전환 할 수 있었던 사례를 공유드리고자 합니다.  

먼저 단순히 Producer 시스템만 전환한게 아니라 Kafka Cluster까지 전환 한 이유에 대해서 말씀드리자면, 기존에 사용하고 있던 Kafka Cluster는 여러 토픽들이 함께 사용되는 고용 Kafka-pool 이었는데, 해당 메세지시스템이 초기설계와는 다르게 초 대용량으로 사용됨에 따라 공용풀에 부담을 주고 있었습니다. 

또한 앞으로 해당 메세지를 연계한 추가적인 메세지가 만들어 질 수 있는 가능성이 있었기에 확장성을 고려하여 전용 Kafka-pool을 구성 할 필요성이 대두되었고 무엇보다도 대용량 메세지인 만큼 운영환경에서의 안정적이고 점진적인 전환을 위해 기존 클러스터와 분리된 별도의 클러스터를 활용해 메세지 발행 시스템 전체를 리빌딩하는 방법을 시도했습니다.

### 전환 Step

운영환경에서 메세지 발행과 소비가 활발하게 수행되고 있는 환경에서도 안정적인 전환을 위해서는 `한번에 전환` 같은 매직은 어렵습니다. 따라서 아래와 같은 계획을 가지고 Step 을 나누어 부분적으로 전환을 시도했습니다.

1. 신규 클러스터 구축 & 메세지 미러링환경 구축
  - ![2]({{ site.images | relative_url }}/posts/2022-08-07-kafka-migration/2.jpg)  
  - 미러링 환경을 구축하는것은 MirrorMaker 등의 툴을 이용한다면 메세지 미러링 환경을 구축하는것을 그다지 어렵지 않습니다. 다만 주의할것은 단순히 메세지만 복제하면 되는것이 아니라, Consumer들을 전환할때 중복 메세지 소비 혹은 메세지 누락이 일어나지 않도록 Consumer Group Offset까지도 같이 복제를 해줘야합니다. consumer gorup offset이 복제되지 않는다면 consumer가 새 broker에 붙을때 consumer offset 정책에 따라 `latest`, `earliest` 를 수행하면서 메세지 누락 혹은 중복이 발생 할 수 있습니다.

2. Consumer 전환
  - ![3]({{ site.images | relative_url }}/posts/2022-08-07-kafka-migration/3.jpg)
  - Consumer 전환시 일반적인 배포정책인 blue-green 배포 정책으로 신규 broker으로 전환하게 될시 동일 컨슈머그룹이 `기존 Kafka - Blue Consumer`, `신규 Kafka - Green Consumer` 연결이 일어 날 수 있습니다. 이경우 Consumer Group Offset 동기화가 비정상적으로 처리될 수 있으므로 기존 Consumer를 완전히 종료후 신규 Consumer를 띄우는 방식으로 전환을 시도해야합니다.
  - 추가로 Consumer Group Offset 복제는 실시간으로 동기화되지 않을수있습니다. MirrorMaker2 의 경우 설정에따라 다르나 동기화에 최소 1초의 시간이 소요될 수 있으므로 consumer 전환시 적절한 텀을 가지고 전환이 필요 할 수 도 있습니다.

3. Producer 전환
  - ![4]({{ site.images | relative_url }}/posts/2022-08-07-kafka-migration/4.jpg)  
  - 일반적으로 Producer들 또한 scale-out 구성으로 서비스되고 있는 경우가 많을텐데, Producer 전환시에 모든 서버에서 동시에 전환되도록 해주어야합니다. 그렇지 않는다면 일부는 기존 Kafka로 발행, 일부는 신규 Kafka로 발행되면서 메세지 순서가 꼬이게 될 수 있습니다. 기존 Kafka으로 발행되더라도 미러링 구성을 통해 복제되서 신규 Kafka으로 메세지가 들어오겠지만 만약 동일 ID의 메세지가 발행되는 경우에 순서보장이 되지 않을수 있으니 주의해야합니다.
  

### 신규 시스템을 어떻게 신뢰 할 수 있을까?

위에서 공유드린 방법을 통해 신규시스템으로 안정적인 전환을 하더라도, 신규시스템에서 발행된 메세지에 대한 신뢰도를 가졌는지 여부는 또 다른 이야기가 될 수 있습니다.

![5]({{ site.images | relative_url }}/posts/2022-08-07-kafka-migration/5.jpg)  

일반적으로 Producer 리빌딩 하면서 발생 할 수 있는 문제들은 아래와 같습니다.

- 메세지 스펙이 달라졌어요
- 메세지 발행이 누락됬어요
- 메세지 중복 발행 되요
- 모르는 이상한 메세지가 발행되고 있어요

#### 정합성 검증기 운영

신규시스템으로 완전 전환하기 전에 먼저 검증기를 수행시킴으로서 위 문제들을 사전에 검증시켜 신규 시스템에 대한 신뢰도를 높여줄 수 있습니다.  
먼저 정합성검증기르 통해 메세지 스펙에 대해 검증해줄수 있습니다.

![6]({{ site.images | relative_url }}/posts/2022-08-07-kafka-migration/6.jpg)  

신규 Producer 에서 구)메세지를 consume 하고 해당 메세지 바탕으로 신)메세지를 새로 만들어 비교하는 방식으로 신규 메세지에 대한 정합성을 체크 할 수 있습니다.

#### 발행 검증기 운영

![7]({{ site.images | relative_url }}/posts/2022-08-07-kafka-migration/7.jpg)  

old/new 발행검증기를 통해서 단순 메세지 스펙 비교로만으로는 확인 할수 없는 케이스들을 추가로 검증 할 수 있습니다.  

#### 실제 검출된 케이스

실제로 위 검증기를 통해 아래와 같은 케이스들이 전환전에 미리 검출되어 수정 할 수 있었습니다.

- 필드 누락
- 잘못된 필드값
- 암호화 누락 필드값
- 신규스펙 미대응 필드값
- 특정 케이스 메세지 발행누락

### 마무리

일 3,000만건의 대량의 메세지를 중단없이 안전하게 전환하는것이 굉장히 도전적인 프로젝트라고 생각하고 초기에 많은 검색을 했으나 저희 상황과 부합하는 사례가 없어 많은 시행착오를 겪었습니다. 혹시나 비슷한 작업을 생각하고 계신 분들께 좋은 사례가 될수있기를 바랍니다.

좀 더 자세한 내용은 아래 링크에서 확인 가능합니다.

- https://d2.naver.com/helloworld/9581727