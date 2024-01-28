---
layout: post
comments: true
title: Spiring boot 3.2.0 update
tags: [spring, springboot]
---

### Spiring boot 3.2.0 update  

Spring boot 3.2.0 update 사항 정리 (2023. 11. 23)

원문 
- https://spring.io/blog/2023/11/23/spring-boot-3-2-0-available-now/
- https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.2-Release-Notes

---

### 내기준 주요 변경사항

- [RestClient 지원](hhttps://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.2-Release-Notes#restclient-support)
    - webClient와 비슷한 `RestClient` 제공
    - `RestTemplate`를 대체할수 있는 기능이 될듯
- [가상 스레드 지원](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.2-Release-Notes#support-for-virtual-threads)
    - java19 부터 지원
    - 경량의 가상스레드 사용
    - https://openjdk.org/jeps/444
- [Micrometer Tracing 기능 업데이트](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.2-Release-Notes#observability-improvements)
    - `@Coutned`, `@NewSpan`, `@ContinueSpan`, `@Observed` 어노테이션 제공


---

### summary by chatGPT

### 1. Spring Framework 3.2 사용 변경
- Spring Boot 3.2에서는 `spring-security-oauth2-client`, `spring-security-oauth2-resource-server`, `spring-security-saml2-service-provider` 등이 클래스패스에 있을 때, `InMemoryUserDetailsManager`의 동작이 변경됨.

### 2. OTLP 추적 엔드포인트 변경
- `management.otlp.tracing.endpoint`의 기본값이 제거되었으며, 값이 있는 경우에만 `OtlpHttpSpanExporter`가 자동 구성됨.

### 3. H2 버전 2.2 기본 사용
- Spring Boot가 기본적으로 H2의 2.2 버전을 사용하도록 변경됨.

### 4. Oracle UCP DataSource 변경
- Oracle UCP DataSource가 더 이상 기본적으로 `validateConnectionOnBorrow`를 `true`로 설정하지 않음.

### 5. Jetty 12 지원
- Spring Boot가 이제 Jetty 12를 지원함.

### 6. Kotlin 1.9.0 및 Gradle
- Kotlin 1.9.0 Gradle 플러그인의 버그로 인해 추가 리소스 디렉토리가 손실되는 문제가 있으며, 이를 해결하기 위해 Kotlin의 Gradle 플러그인을 먼저 적용해야 함.

### 7. 중첩된 Jar 지원 변경
- Spring Boot의 "Uber Jar" 로딩을 지원하는 기본 코드가 Java 8을 더 이상 지원하지 않으므로 업데이트됨. Gradle 사용자는 `bootJar.loaderImplementation`을 `org.springframework.boot.loader.tools.LoaderImplementation.CLASSIC`로 설정하여 이전 동작으로 복원할 수 있음.

### 8. Spring for Apache Pulsar 지원
- Spring Boot에는 이제 Spring for Apache Pulsar 프로젝트에 대한 자동 구성 지원 및 스타터 POM이 포함됨.

### 9. 로깅 상관 ID
- Micrometer 추적을 사용할 때 Spring Boot는 이제 자동으로 상관 ID를 로깅함.

### 10. RestClient 지원
- Spring Boot 3.2에는 Spring Framework 6.1에서 도입된 RestClient 인터페이스 지원이 포함됨.

### 11. RestTemplate HTTP Clients
- Jetty의 HttpClient가 클래스패스에 있는 경우, Spring Boot의 HTTP 클라이언트 자동 감지는 이제 RestTemplateBuilder가 Spring Framework 6.1에서 도입된 JettyClientHttpRequestFactory를 사용하도록 구성함.

### 12. JdbcClient 지원
- JdbcClient에 대한 자동 구성이 추가되었으며, NamedParameterJdbcTemplate이 자동 구성되면 spring.jdbc.template.* 속성이 고려됨.

### 13. 가상 쓰레드 지원
- Spring Boot 3.2에서는 가상 쓰레드 지원이 추가되었으며, Java 21에서 실행하고 `spring.threads.virtual.enabled` 속성을 `true`로 설정해야 함.

### 14. Servlet 웹 서버
- 가상 쓰레드가 활성화된 경우, Tomcat 및 Jetty는 요청 처리에 가상 쓰레드를 사용함.

### 15. 블로킹 실행(Spring WebFlux)
- Spring WebFlux의 블로킹 실행 지원이 `applicationTaskExecutor`가 `AsyncTaskExecutor`인 경우 자동으로 구성됨.

### 16. 작업 실행(Task Execution)
- 가상 쓰레드가 활성화된 경우, `applicationTaskExecutor` 빈은 `SimpleAsyncTaskExecutor`로 구성되어 가상 쓰레드를 사용함.

### 17. 작업 스케줄링(Task Scheduling)
- 가상 쓰레드가 활성화된 경우, `taskScheduler` 빈은 `SimpleAsyncTaskScheduler`로 구성되어 가상 쓰레드를 사용함.

### 18. JVM 유지
- `spring.main.keep-alive` 속성이 추가되었으며, `true`로 설정하면 모든 다른 쓰레드가 가상(또는 데몬) 쓰레드 일지라도 JVM을 유지함.

### 19. 기술별 통합
- 가상 쓰레드가 활성화된 경우, RabbitMQ 리스너 및 Kafka 리스너에 대해 가상 쓰레드 Executor가 자동 구성됨.

### 20. JVM Checkpoint Restore 초기 지원
- Spring Boot 3.2에서는 JVM Checkpoint Restore(Project CRaC)에 대한 초기 지원이 추가되었음.

### 21. SSL 번들 리로딩
- SSL 번들이 변경될 때 SSL 번들을 자동으로 리로딩할 수 있음. 단, 번들은 리로딩을 지원하도록 `reload-on-update` 속성을 `true`로 설정해야 함.

### 22. 감시 기능 개선
- Micrometer의 `@Timed`, `@Counted`, `@NewSpan`, `@ContinueSpan`, 및 `@Observed` 어노테이션을 사용할 수 있음. B3 추적 전파의 기본 형식이 `single-no-parent`에서 `single`로 변경됨.

### 23. OpenTelemetry
- OpenTelemetry MeterProvider 빈이 발견되면 BatchSpanProcessor에 자동으로 등록됨.

### 24. Micrometer 1.12에서 더 넓은 Exemplar 지원
- Micrometer 1.12에는 새로운 Exemplar 지원이 포함되어 있으며, Prometheus 2.43 이상 버전이 필요함.

### 25. 테스트에서 감시 기능
- 이제 Micrometer Tracing, Brave 및 OpenTelemetry 인프라가 감지 기능이 비활성화된 상태로 통합 테스트를
