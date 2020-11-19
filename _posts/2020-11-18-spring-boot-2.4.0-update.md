---
layout: post
comments: true
title: Spring boot 2.4.0 update
tags: [spring, springboot]
---

### Spiring boot 2.4.0 update

Spring boot 2.4.0 update 사항 정리 (2020. 11. 12)

원문 : https://spring.io/blog/2020/11/12/spring-boot-2-4-0-available-now

---
### versioning 형식의 변경

- MAJOR.MINOR.PATCH[-MODIFIER] 형식으로 변경됨
- `2.4.0`, `2.4.0-M1`, `2.4.0-SNAPSHOT`, ...
- 기존 (2.3.5.RELEASE) -> 변경 (2.4.0)

---

### JUnit 5’s Vintage Engine 삭제

- `spring-boot-starter-test`에서 `JUnit 5’s Vintage Engine`이 제거됨
- JUnit4로 작성된 Test가 있다면 JUnit5로 변경하거나 `JUnit 5’s Vintage Engine` dependency를 별도로 추가해주어야 함

```
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```
---

### Config 파일 처리방식의 변경 (properties, yml)

- multi-document yml file(---로 구분된 yml 파일)을 사용시, 선언된 순서대로 추가되는점을 유의해야함 (기존에는 프로필 활성화 순서 기반)
- 속성 오버라이딩의 승자를 결정할때는 문서내 선언 순서를 확인하여 정렬해주어야 함
- `spring.config.use-legacy-processing=true` 활성화로 legacy mode를 사용 할 수 있음
- 기존에는 외부 jar 사용시 config 파일을 override 하지 않았지만, `2.4.0` 부터는 패키징된 외부 jar의 config  파일을 재정의함
- `spring.profiles` & `spring.config.activate.on-profile` 으로 마이그레이션 필요함
- config file import 시, 기존에는  not found error 가 발생했지만 `2.4.0`부터는 skip 함

---

### embedded database logic 변경

- 데이터베이스가 메모리에 있는경우에만 임베디드 DB로 간주하도록 개선됨
- 임베디드 DB로 간주되지 않는다면 시작시 초기화되지 않음 주의 
- `spring.datasource.initialization-mode` 설정으로 변경 가능
- H2 사용시 `default user name = sa`는 더이상 설정되지 않음
- `spring.datasource.username = sa` 지정 필요

---

### Logback property 이름 변경

Spring boot property
- logging.pattern.rolling-file-name → logging.logback.rollingpolicy.file-name-pattern
- logging.file.clean-history-on-start → logging.logback.rollingpolicy.clean-history-on-start
- logging.file.max-size → logging.logback.rollingpolicy.max-file-size
- logging.file.total-size-cap → logging.logback.rollingpolicy.total-size-cap
- logging.file.max-history → logging.logback.rollingpolicy.max-history

system environment property
- ROLLING_FILE_NAME_PATTERN → LOGBACK_ROLLINGPOLICY_FILE_NAME_PATTERN
- LOG_FILE_CLEAN_HISTORY_ON_START → LOGBACK_ROLLINGPOLICY_CLEAN_HISTORY_ON_START
- LOG_FILE_MAX_SIZE → LOGBACK_ROLLINGPOLICY_MAX_FILE_SIZE
- LOG_FILE_TOTAL_SIZE_CAP → LOGBACK_ROLLINGPOLICY_TOTAL_SIZE_CAP
- LOG_FILE_MAX_HISTORY → LOGBACK_ROLLINGPOLICY_MAX_HISTORY

---

### Default 서블릿 등록 제외

- `2.4.0`부터는 서블릿 컨테이너에서 제공하는 `DefaultServlet`을 등록하지 않음
- 필요하다면 `server.servlet.register-default-servlet=true` 설정

---

### HTTP traces 쿠키헤더 제외

---

### Undertow Path on Forward

---

### Neo4j

- Neo4j 지원의 대폭 개편

---

### Elasticsearch RestClient

- 저수준의 Elasticsearch RestClient bean이 자동생성되지 않음

---

### R2DBC

- R2DBC가 `spring-r2dbc`으로 지원

---

### Flyway

- Flyway 7 업그레이드 지원

---

### Flatten Maven Plugin 제거

---

### exec-maven-plugin 제거

---

### Spring boot gradle main class 지정 변경

기존
```
bootJar {
	mainClassName 'com.example.ExampleApplication'
}
```
변경후
```
bootJar {
	mainClass 'com.example.ExampleApplication'
}
```

---

### @SpringBootTest 변경점

- `@SpringBootTest` 모니터링 시스템을 구성하지 않음
- `in-memory meterEgistry`만 제공함
- `@AutoConfigureMetric` 설정으로 복원 가능

---

### Spring Boot 2.2 and 2.3 지원중단

---

### Spring framework 5.3 지원

- Upgrade to ASM 9.0 and Kotlin 1.4.
- `ReactiveAdapterRegistry` 에서 RxJava3 지원, RxJava1 지원 중지
- Java14/15 record class binding support
- ...

---

### JAVA15 완벽한 지원
