---
layout: post
comments: true
title: Spiring boot 3.0.0 update
tags: [spring, springboot]
---

### Spiring boot 3.0.0 update  

4.5 년만에 메이저 업데이트
Spring boot 3.0.0 update 사항 정리 (2022. 11. 24)

원문 
- https://spring.io/blog/2022/11/24/spring-boot-3-0-goes-ga
- https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Release-Notes

---

### 내기준 주요 변경사항

- [Java 버전 최소기준치 java17 사용](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Release-Notes#java-17-baseline-and-java-19-support)
    - 스프링 버전업을 위해서라도 java 버전을 함께 올리는 프로젝트들이 많아질듯
- [SpringBatch 5.0](https://github.com/spring-projects/spring-batch/releases/tag/v5.0.0)
    - 스프링 배치 5.0 도 함께 버전업으로 들어감
    - metaTable 정의가 달라져서 기존 배치 테이블 마이그레이션 필요함
    - https://github.com/spring-projects/spring-batch/releases/tag/v5.0.0
- Java EE -> Jakarta EE
    - jakarta api 으로 마이그레이션 필요
- [GraalVM 지원](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Release-Notes#graalvm-native-image-support)
    - 여러 행사에서 이미 공개되왔던 graalVM 을 지원함
    - https://docs.spring.io/spring-boot/docs/3.0.0/reference/html/native-image.html#native-image

---

### summary by chatGPT

#### Spring Boot 2.7에서의 업그레이드
- Spring Boot 3.0은 Java 17을 최소 버전으로 요구하며, Java 8 또는 Java 11을 사용 중인 경우 JDK를 업그레이드해야 합니다.
- Spring Boot 3.0은 JDK 19와도 잘 작동하며 테스트되었습니다.
- Spring Boot는 이제 GraalVM 22.3 이상 및 Native Build Tools Plugin 0.9.17 이상을 요구합니다.

#### 써드파티 라이브러리 업그레이드
- Spring Boot 3.0은 Spring Framework 6를 기반으로 하며, Spring Framework 6.0의 새로운 기능을 활용할 수 있습니다.
- 이 릴리스에서 다음 Spring 프로젝트도 업그레이드되었습니다: Spring AMQP 3.0, Spring Batch 5.0, Spring Data 2022.0, Spring GraphQL 1.1, Spring HATEOAS 2.0, Spring Integration 6.0, Spring Kafka 3.0, Spring LDAP 3.0, Spring REST Docs 3.0, Spring Retry 2.0, Spring Security 6.0 등.
- Spring Boot 3.0은 Java EE 대신 Jakarta EE API를 모든 종속성에 대한 사용합니다.

#### GraalVM Native Image 지원
- Spring Boot 3.0 애플리케이션은 GraalVM Native Image로 변환할 수 있으며, 메모리 및 시작 성능 향상을 제공할 수 있습니다.
- GraalVM Native Images 지원은 Spring 포트폴리오 전체에 걸쳐 수행된 주요 엔지니어링 작업 중 하나입니다.
- GraalVM Native Images 사용을 시작하려면 Spring Boot 참조 문서를 참조하십시오.

#### Log4j2 업데이트
- Log4j2 지원이 새로운 확장 기능과 함께 업데이트되었습니다. 프로필별 구성, 환경 프로퍼티 조회, Log4j2 시스템 프로퍼티 등을 제공합니다.
- 자세한 내용은 업데이트된 문서를 확인하십시오.

#### @ConstructorBinding 검색 개선
- 생성자 바인딩 @ConfigurationProperties를 사용할 때 이제 클래스에 단일 매개변수화 생성자가 있는 경우 @ConstructorBinding 어노테이션이 더 이상 필요하지 않습니다.
- 두 개 이상의 생성자가 있는 경우 Spring Boot에 사용할 생성자를 알려주기 위해 여전히 @ConstructorBinding를 사용해야 합니다.

#### Micrometer 업데이트
- Micrometer 1.10에서 소개된 새로운 관찰 API를 지원합니다.
- Micrometer 트레이싱을 자동으로 구성합니다. Brave, OpenTelemetry, Zipkin 및 Wavefront을 지원합니다.
- Micrometer의 OtlpMeterRegistry를 자동으로 구성합니다.

#### Prometheus 지원
- Prometheus 훅과 관련된 지원이 추가되었습니다.
- 종료 훅을 수행하기 위해 management.prometheus.metrics.export.pushgateway.shutdown-operation 프로퍼티가 추가되었습니다.

#### Spring Data JDBC 자동 구성 향상
- Spring Data JDBC의 자동 구성이 더 유연해집니다. Spring Data JDBC에 필요한 여러 자동 구성된 빈을 조건부로 대체할 수 있습니다.

#### Apache Kafka에서 Async Acks 활성화
- spring.kafka.listener.async-acks 설정을 사용하여 Kafka에서 Async Acks를 활성화하는 데 사용합니다.

#### Elasticsearch Java Client
- Elasticsearch Java Client를 위한 자동 구성이 도입되었습니다.
- ObjectMapper를 사용자 정의하려면 JacksonJsonpMapper 빈을 정의하면 됩니다.

#### JdkClientHttpConnector 자동 구성
- Reactor Netty, Jetty의 리액티브 클라이언트 및 Apache HTTP 클라이언트가 없는 경우 JdkClientHttpConnector가 자동으로 구성됩니다.

#### @SpringBootTest에서 Main 메서드 지원
- @SpringBootTest 어노테이션은 이제 사용 가능한 경우 모든 @SpringBootConfiguration 클래스의 main 메서드를 사용할 수 있습니다.

#### 기타
- 기타 변경 및 개선 사항
- Spring Boot 3.0에서의 폐기 사항
- 기능과 관련된 몇 가지 변경사항
- 업그레이드 지침에 대한 참고사항
