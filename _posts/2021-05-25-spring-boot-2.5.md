---
layout: post
comments: true
title: Spiring boot 2.5.0 update
tags: [spring, springboot]
---

### Spiring boot 2.5.0 update  

Spring boot 2.5.0 update 사항 정리 (2021. 05. 20)

원문 : https://spring.io/blog/2021/05/20/spring-boot-2-5-is-now-ga

---

#### SQL Script 초기화 re-design

- property 패키지가 변경 됩니다. `'spring.datasource.*'` -> `'spring.sql.init.*'` 
- `R2DBC`에서도 사용 가능합니다.
- default option으로, `'data.sql'` script가 hibernate initialize 하기 전에 수행됩니다.

---

### Flyway, Liquibase JDBC URL

- `'spring.flyway.url'` 혹은 `'spring.liquibase.url'` 사용중일 경우 `'username'`, `'password'`를 추가해서 사용해야 합니다.

---

### Spring Data Solr

- 이번 release에서 `auto-configuration for Spring Data Solr`가 제거되었습니다.

---

### Secure Info Endpoint

- spring actuator `'/info'`가 web에서 더이상 default로 노출도디지 않습니다.

---

### Default Expression Language (EL) Implementation

- `glassfish.jakarta.el` 대신 `tomcat-embed-el`이 사용됩니다.

---

### Messages in the Default Error View

- default error view 에서 `message` 항목이 비어있을경우 공백으로 나타내는 대신, 제거됩니다.
- json으로 메세지를 파싱하는 경우 해당 코드를 수정 해야 할 수 있습니다.

---

### Logging shutdown hook

- JVM 종료시 로깅 자원이 해제되도록 logging shutdown hook을 등록합니다.
- `'logging.register-shutdown-hook'` property 설정으로 수정할수 있습니다.

---

### Gradle Default jar and war Tasks

- `Spring boot gradle plugin`이 더이상 자동으로 `standard gradle jar ,war task`를 disable 시키지 않습니다.

---

### Cassandra Throttling Properties

- `Spring boot`가 더이상 `spring.data.cassandra.request.throttle` 의 default value를 제공하지 않습니다.
- `max-queue-size`, `max-concurrent-requests`, `max-requests-per-second`, `drain-interval` property를 직접 설정해야합니다.


---

### Customizing jOOQ’s DefaultConfiguration

- JOOQ의 `DefaultConfigurationCustomizer` Bean를 직접 정의 할 수 있습니다.

---

### Groovy 3

- `groovy` default 버전이 3.x 으로 업그레이드 됬습니다.

---

### Minimum Requirements Changes

- 그래들 최소 요구사항이 `Gradle 6.8` 으로 변경되었습니다.


---

### Environment Variable Prefixes

- 환경변수 prefix 설정이 가능합니다.

```java
SpringApplication application = new SpringApplication(MyApp.class);
application.setEnvironmentPrefix("myapp");
application.run(args);
```

---

### HTTP/2 over TCP (h2c)

- 모든 embedded web container 에서 `HTTP/2 over TCP (h2c)` 사용이 가능합니다.
- `server.http2.enabled = true`

---

### Generic DataSource Initialization

- datasource initilize 코드 작업시 `Flyway`, `Liquibase` 기반으로 수행됩니다.

---

### Layered WARs

- Docker image 사용을 위한 `layered war`를 제공합니다.


---

### Docker image build supports

- custom `builderpacks` 설정이 가능합니다.
- `buildpack` builder 에서 volume binding이 가능합니다.
- war file package가 가능합니다.

---

### OpenMetrics for Prometheus

- actuator endpoint `'/actuator/prometheus'`는 표준 `prometheus`와 `openmetircs`  응답 모두를 제공할 수 있습니다.

---

### Metrics for Spring Data Repositories

- actuator는 spring data repository에 대한 micrometer 매트릭을 생성합니다.

---

### @Timed Metrics with WebFlux

- `@Timed`를 사용해 Webflux 컨트롤러, 핸드러가 처리하는 request 타이밍을 수동으로 활성화 할 수 있습니다.
- `management.metrics.web.server.request.autotime.enabled = false`

---

### MongoDB Metrics

- actuator 사용시 자동으로 mongodb metric 연결과 command가 수행됩니다.

---

### Actuator Endpoint for Quartz

- actuator endpoint `/quartz`가 추가되었습니다.
- quartz job, trigger 상세 정보를 볼 수 있습니다.


---

### GET requests to actuator/startup

- actuator endpoint `/startup`이 `GET` request를 지원합니다.
- `POST` request와는 다르게 이벤트 버퍼를 비우지 않고, 메모리에서 지속적으로 이벤트가 유지됩니다.

---

### Abstract Routing DataSource Health

- actuator health endpoint 에서 `AbstractRoutingDatasource`의 health 상태도 보여집니다.


---

### update support

- JAVA 16 support
- Gradle 7 support
- Jetty 10 support

---

### 정리

- 이번 minor 버전 releasee 에서는 `Datasource`와 `Actuator`, `Metric` 강화에 대한 내용이 많이 포함되어 있는것으로 보입니다.
















