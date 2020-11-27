---
layout: post
comments: true
title: 매일 1억건씩 변경되는 상품 DB를 Column-oriented DBMS로 변경한 좌충우돌 마이그레이션 경험기
tags: [db, database, migration, column, vertica]
---

### API 조합 패턴

하나의 DB에 여러 데이터를 저장하던 모놀리식 아키텍처에서는 여러 도메인의 데이터들를 조회하고, DB에서 쿼리로 조인하여 조합하기도 비교적 쉽게 수행 할 수 있었을 것 입니다.  

하지만 MSA 환경에서는 여러 도메인의 조합된 데이터를 조회하는것은 여간 쉬운일이 아닐것 입니다. 예를들어 `A 카탈로그`에 매칭된 상품들을 조회한다고 했을때, 프론트엔드에서는 `카탈로그 서비스`, `일반 상품 서비스`, `EP 상품 서비스`를 별도로 모두 호출해야 할 것입니다.

![1]({{ site.images | relative_url }}/posts/2020-11-22-msa-5/1.png) 

API 조합 패턴의 개념은 어렵지 않습니다. 클라이언트 측에서 각각의 MSA 서비스들을 호출하는것이 아닌, MSA로 나누어진 여러 서비스들의 데이터를 조합해주는 `API 조합기`를 두어 클라이언트는 `API 조합기`만을 호출함으로써 원하는 조합된 데이터를 얻는형태의 패턴입니다.

![2]({{ site.images | relative_url }}/posts/2020-11-22-msa-5/2.png) 

이 `API 조합기`의 경우 MSA 분리방식, 사용되는 데이터등에 따라 가변적 이기 때문에 정해진 구현 방식이 있는것은 아니지만 일반적으로 모놀리식 아키텍처에서 `DB Join`이 수행하던 작업들을 어느정도 수행해야하기에 In-memory join을 수행하게 될 것입니다.  

이와 같은 경우에는 메모리 데이터 로드에 한계가 있음으로 만양 대용량 데이터의 join이 필요한 경우에는 일반적인 Join 방식 대신에 다른 형식의 데이터 join을 고안 해 봐야 할 필요가 있습니다.  

---

### API 조합기를 어디에 둘것인가?

API 조합기의 역할을 할 수 있는 서버는 3가지의 후보가 있습니다.

첫번째로는 클라이언트 웹서버에 `API 조합기`를 구현한다면 같은 LAN을 사용하여 가장 효율적으로 서비스 호출을 할 수 있을것입니다.

두번째로 만약 현재 서비스에 `API 게이트웨이`가 적용되어 있다면 `API 게이트웨이`를 `API 조합기`로 사용 할 수도 있습니다.  

세번째로 `API 조합기`를 위한 별도의 서비스를 구축하는것입니다. 운영중인 MSA 서비스가 많고 데이터 처리로직이 복잡하다면 별도로 서비스를 구축하는것이 좋은 방법 입니다.

---

### API 조합기 유의점

첫번째로, `API  조합기`에서 제공하는 API은 대부분 클라이언트에서 직접 호출하는 내용들이 대부분이기에 `빠른 응답속도`이 보장 되어야 하는데요, `API 조합기`는 여러 서비스들을 호출하여 데이터들을 가져와 조합을 하기때문에 여러 호출이 `Blocking` 수행이된다면 응답시간이 그만큼 길어질 것입니다. 가능하다면 `Reactive model`을 사용하여 여러 서비스들을 `Non-blocking`으로 병렬 호출하여 데이터를 조합해준다면 클라이언트 응답시간에 개선이 있을 수 있습니다.  

둘째로, 모놀리식과 비교하면 DB에서 간단하게 한번의 호출로 끝나는것과 다르게 한번의 호출에서 여러서비스를 호출하고 여러 DB 쿼리를 수행하게 되기에 서비스들 전체적으로 오버헤드가 증가하게 됩니다.   

셋째로, 서비스가 많아질수록 당연하게도 가용성을 떨어지게 됩니다. 가용성을 높이기 위해 호출한 서비스가 불능이더라도 미리 만들어둔 캐시를 사용하게 한다던지 미완성 데이터라도 응답하는등의 방식으로 가용성을 높여 줄 수 있습니다.

넷째, 데이터 일관성이 맞지 않을수 있습니다. 데이터 조회, 변경 작업들이 하나의 DB 트랜잭션으로 수행되지 않아 `isolation`이 보장되지 않기 때문에 부정확한 데이터를 응답 받을 수 있다는점에 유의해야 합니다.

---

### CQRS 패턴

`Command and Query Resposibility Segregation` 이하 `CQRS`은 말 그대로 Command (명령)과 Query (조회)를 분리하는 패턴입니다.   

일반적인 엔터프라이즈 어플리케이션에서 데이터는 RDBMS에서 관리를 하고 텍스트 서치 들을 위해 ES(Elastic search)나 Solar등을 이용하는것 처럼 명령과 조회를 분리하는 패턴을 `CQRS`라고 합니다.

위 `API 조합 패턴`의 경우 매번 호출 할 때 마다 여러 서비스들로부터 데이터를 가져와 조합하면서 성능상의 이슈나 복잡한 데이터 조합 사용의 어려움등이 일어났는데, `CQRS`는 조합이 필요한 데이터들을 DB나 ES 등의 데이터저장소에 미리 구성해두고 사용하여 해당 이슈를 해결시킬수 있습니다.

또한 `CQRS`의 의미대로 명령과 조회의 관심사가 분리되기 때문에  데이터가 저장되는 DB가 변경되어도 조회에 필요한 데이터가 변경되지않는다면 해당 Reader 데이터를 사용하는 코드의 변경은 없게끔 될 수도 있습니다.

![3]({{ site.images | relative_url }}/posts/2020-11-22-msa-5/3.png) 

---

### CQRS 장단점

CQRS를 사용하면 다음과같은 장점이 있습니다.

- 효율적인 쿼리 사용이 가능하다
- 다양한 쿼리 사용이 가능하다
- 이벤트 소싱 패턴에서 쿼리 사용이 가능하다
- 관심사를 분리 시킬 수 있다

물론 단점도 존재합니다.

- Read 전용 데이터소스(view)를 구성하기 위해 별도의 서비스를 개발해야 한다
- 아키텍처가 복잡해진다
- 데이터 복제에 시차가 발생한다

---

### CQRS view 구축하기

`CQRS View`를 구축하기로 결정했다면, `CQRS View`의 구성요소 세가지를 결정하고 구축해야합니다.

첫번째로, 데이터를 저장할 저장소를 선택 하는것입니다. `CQRS View`는 서비스에서 `Read`만 한다는 특징이 있기때문에 전통적인 `RDB`만을 고집하는것 보다는 상황에따라 유연하게 선택 할 수 있다는 강점이 있습니다.

- JSON 객체 검색 -> 문서형 스토어 (MongoDB, DynamoDB ...)
- 키값 검색 -> Key-value 스토어 (Redis ...)
- 텍스트 쿼리 -> 검색엔진 (Elastic search, Solar ...)
- 그래프 쿼리 -> 그래프 DB (Neo4j ...)
- SQL 리포팅, BI -> RDB (MySQL, Oracle ...)

둘째로, 데이터 동기화를 위한 이벤트 핸들러 작업을 효율적으로 수행시켜야 합니다. 이 이벤트 핸들러작업이 실제 데이터의 변경 처리건을 실시간으로 반영하지 못하기 때문에 실제 데이터와의 시차가 발생 할 수 있습니다.

셋째로, 데이터 조회를 위한 API서비스를 구축해야합니다.

 