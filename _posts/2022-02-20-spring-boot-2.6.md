---
layout: post
comments: true
title: Spiring boot 2.6.0 update
tags: [spring, springboot]
---

### Spiring boot 2.6.0 update  

Spring boot 2.6.0 update 사항 정리 (2021. 11. 19)

원문 
- https://spring.io/blog/2021/11/19/spring-boot-2-6-is-now-available
- https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.6-Release-Notes

---

### 내기준 주요 변경사항

- [bean 순환참조가 default 옵션으로 금지처리됨.](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.6-Release-Notes#circular-references-prohibited-by-default)
    - 순환참조 여부를 좀 더 빠르게 확인 가능해질듯.
    - 순환참조를 의도적으로 사용 할 일이 그렇게 많지는 않겠지만, 옵션설정을 통해 가능하도록 처리 가능.
- [Kafka 지원 version up (3.0.0)](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.6-Release-Notes#kafka-30)


---

### summary by chatGPT

#### 순환 참조 금지
- 빈 간의 순환 참조가 기본적으로 금지되며, 순환 참조로 인해 애플리케이션이 시작에 실패하면 의존성 순환을 해결하도록 설정을 업데이트해야 합니다.

#### Spring MVC 요청 경로 일치 전략 변경
- Spring MVC 핸들러 매핑에 대한 요청 경로 일치 전략이 `AntPathMatcher`에서 `PathPatternParser`로 변경되었습니다.

#### Actuator 엔드포인트 변경
- Actuator 엔드포인트는 이제 PathPattern을 기반으로하는 URL 일치 전략을 사용합니다.

#### Actuator Env InfoContributor 기본 비활성화
- Env InfoContributor는 이제 기본적으로 비활성화되며, "info"로 시작하는 환경 속성을 환경에서 노출하려면 설정을 업데이트해야 합니다.

#### Embedded MongoDB 버전 설정
- Embedded MongoDB를 사용하려면 `spring.mongodb.embedded.version` 속성을 설정해야 하며, 이는 내장 지원에서 사용하는 MongoDB 버전을 프로덕션에서 사용할 MongoDB 버전과 일치시키는 데 도움이 됩니다.

#### Oracle 데이터베이스 드라이버 종속성 변경
- Oracle 데이터베이스 드라이버 종속성 관리가 개선되었으며, `com.oracle.ojdbc` 그룹 대신 `com.oracle.database.jdbc` 그룹을 사용해야 합니다.

#### Elasticsearch 속성 통합
- Elasticsearch 클라이언트 설정 속성이 통합되었습니다.

#### Apache Kafka 3.0 업그레이드
- Apache Kafka 3.0으로 업그레이드되었으며, idempotence 기능을 비활성화해야 할 필요가 있는 문제가 발생할 수 있습니다.

#### Spring Boot 2.6의 종속성 업데이트
- Spring Boot 2.6은 여러 Spring 프로젝트 및 서드파티 의존성의 새로운 버전으로 이동합니다.
