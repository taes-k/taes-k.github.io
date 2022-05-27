---
layout: post
comments: true
title: Spiring boot 2.5.0 update
tags: [spring, springboot]
---

### Spiring boot 2.7.0 update  

Spring boot 2.7.0 update 사항 정리 (2022. 05. 19)

원문 
- https://spring.io/blog/2022/05/19/spring-boot-2-7-0-available-now
- https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.7-Release-Notes

---

### 내기준 주요 변경사항

- [ES RestHighLevelClient 가 deprecated 됨](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.7-Release-Notes#support-for-elasticsearchs-resthighlevelclient-is-deprecated)
    - low-level RestClient 사용해야해서 간단한 호출들대한 자잘한 코드 사이즈가 증가하겠음
    - 복잡한 쿼리 사용시에는 이미 low-level RestClient 를 사용하고있어 큰 이슈사항은 아닐듯함.
- [WebSecurityConfigurerAdapter deprecated](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.7-Release-Notes#migrating-from-websecurityconfigureradapter-to-securityfilterchain)
    - SecurityFilterChain 으로 마이그레이션 필요
- [AutoConfiguration 설정변경](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.7-Release-Notes#changes-to-auto-configuration)
    - autoConfiguration 파일 선언 방식 변경 필요


---

### summary by chatGPT

#### Spring Boot 2.6에서의 업그레이드 사항
- Spring Boot 2.6에서 업그레이드하는 경우, @SpringBootTest를 사용하여 추가한 테스트 프로퍼티 소스가 명령행 프로퍼티 소스보다 먼저 추가됩니다. 만약 @SpringBootTest가 properties 및 args를 모두 사용하고 같은 프로퍼티 이름을 가지고 있다면 조정이 필요할 수 있습니다.

#### Flyway 새 모듈
- Spring Boot 2.7에서는 Flyway 8.5로 업그레이드되며, 8.0 이후 버전부터 여러 데이터베이스에 대한 지원이 새 모듈로 분리되었습니다.

#### H2 2.1 업그레이드
- Spring Boot 2.7은 H2 2.1.120로 업그레이드되었으며, H2 2.x는 역호환성이 없으며 여러 보안 취약점을 수정했습니다.

#### jOOQ
- Java 8 및 H2 2.x와 호환되는 오픈 소스 버전의 jOOQ가 없습니다. Java 11을 사용하는 경우 jooq.version 프로퍼티를 사용하여 jOOQ 3.16 이상으로 업그레이드하는 것을 고려하세요.

#### Microsoft SQL Server JDBC 드라이버 10
- Spring Boot 2.7은 MSSQL 드라이버를 v9에서 v10으로 업그레이드했습니다. 업데이트된 드라이버는 기본적으로 암호화를 활성화하므로 기존 응용 프로그램에 영향을 줄 수 있습니다. 서버에 신뢰할 수 있는 인증서를 설치하거나 JDBC 연결 URL에 encrypt=false를 포함시키는 것이 권장됩니다.

#### OkHttp 4
- OkHttp 3의 지원이 중단되었으므로 Spring Boot 2.7은 OkHTTP 4로 업그레이드했습니다.

#### netty-tcnative의 별도의 의존성 관리 제거
- netty-tcnative의 별도의 의존성 관리가 제거되었습니다. 대신 Netty의 bom에서 제공하는 의존성 관리를 사용하게 되며 netty-tcnative의 버전을 더 이상 무시하는 데 netty-tcnative.version 프로퍼티를 사용할 수 없습니다.

#### spring.mongodb.embedded.features 구성 프로퍼티 제거
- 임베디드 Mongo 3.4에서 Mongo 피처를 구성하는 지원이 삭제되었습니다. 이에 따라 spring.mongodb.embedded.features 구성 프로퍼티가 제거되었습니다.

#### 서블릿별 Mustache 프로퍼티
- 서블릿별 Mustache 관련 프로퍼티가 사용 중지되었으며 대체 프로퍼티가 도입되었습니다.

#### Auto-configured ReactiveElasticsearchTemplate의 기본 인덱스 옵션
- Auto-configured ReactiveElasticsearchTemplate의 기본 인덱스 옵션이 변경되어 이전의 strictExpandOpenAndForbidClosed에서 strictExpandOpenAndForbidClosedIgnoreThrottled로 변경되었습니다.

#### MongoDB 프로퍼티 우선순위
- 이제 spring.data.mongodb.uri가 설정되어 있는 경우 spring.data.mongodb.host 및 spring.data.mongodb.port와 같은 별도의 프로퍼티가 무시됩니다.

#### Maven 프로세스에서 애플리케이션 실행
- Maven 플러그인의 spring-boot:run 및 spring-boot:start 목표는 기본적으로 분기된 프로세스에서 애플리케이션을 실행합니다. 이 동작을 비활성화하는 데 사용되는 fork 속성은 이제 사용이 중단되었으며 대체될 사항이 없습니다.

#### Ordered Exit Code Generators
- ExitCodeGenerators는 이제 Ordered 구현 및 @Order 주석에 따라 순서가 지정되며 생성된 첫 번째 비제로 exit 코드가 사용됩니다.

#### Metric Tag Keys 이름 변경
- Micrometer의 권장사항에 따라 CamelCase의 메트릭 태그 키가 모두 소문자 및 . 구분 기호를 사용하도록 변경되었습니다.

#### Elasticsearch의 RestHighLevelClient 지원 폐지
- Elasticsearch가 RestHighLevelClient를 지원 중지했습니다. 이와 일치하게 Spring Boot의 RestHighLevelClient의 자동 구성도 폐지되었습니다.

#### R2DBC 드라이버 변경
- Borca 릴리스에서 PostgreSQL용 드라이버인 r2dbc-postgresql의 그룹 ID가 io.r2dbc에서 org.postgresql로 변경되었습니다.

#### WebSecurityConfigurerAdapter에서 SecurityFilterChain으로 마이그레이션
- Spring Boot 2.7은 Spring Security 5.7로 업그레이드되었으며, WebSecurityConfigurerAdapter가 폐기되었습니다. Spring Security를 WebSecurityConfigurerAdapter를 사용하지 않고 구성하고 @WebMvcTest와 같은 Spring Boot의 슬라이스 테스트를 사용하는 경우 보안 필터 체인 빈을 테스트에서 사용 가능하게 하기 위해 보안 구성 클래스를 @Import해야 할 수 있습니다.

#### Maven Shade Plugin과 Gradle Shadow Plugin을 사용한 JAR 빌드
- Spring Boot 2.7에서 자동 구성 및 관리 컨텍스트 클래스가 검색되는 방식이 변경되었습니다. 이제 이러한 클래스들은 "META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports" 및 "META-INF/spring/org.springframework.boot.actuate.autoconfigure.web.ManagementContextConfiguration.imports"이라는 파일에 선언됩니다.

#### Maven Shade Plugin 구성
- maven-shade-plugin을 사용하고 spring-boot-starter-parent에 종속성이 없는 경우 다음 AppendingTransformer 구성을 추가해야 합니다.

### MIMEPull
- org.jvnet.mimepull:mimepull의 의존성 관리가 제거되었습니다. mimepull에 대한 의존성을 선언한 경우 필요한 버전을 해당 선언에 추가해야 합니다.

#### Hazelcast 3.0 지원 폐지
- Hazelcast 3 지원이 폐지되었습니다. 여전히 Hazelcast 3로 다운그레이드해야 하는 경우 클라이언트를 구성하는 데 hazelcast-client를 클래스패스에 추가해야 합니다.

#### Spring MVC의 requestMappingHandlerMapping가 더 이상 기본이 아님
- Spring Framework 5.1부터 Spring MVC는 여러 RequestMappingHandlerMapping 빈을 지원합니다. 따라서 Spring Boot 2.7에서는 Spring MVC의 주요 requestMappingHandlerMapping 빈이 더 이상 @Primary로 정의되지 않습니다. RequestMappingHandlerMapping을 주입했던 경우, 컨텍스트에 여러 개의 해당 빈이 있는 경우 @Qualifier를 사용하여 주입하려는 후보를 선택해야 합니다.

#### MySQL JDBC 드라이버
- MySQL JDBC 드라이버의 좌표가 변경되었습니다. 8.0.31 버전은 com.mysql:mysql-connector-j와 mysql:mysql-connector-java에 발행되며, 8.0.32 이후 버전은 com.mysql:mysql-connector-j에만 발행됩니다. Spring Boot 2.7.8에서는 8.0.32로 업그레이드되었습니다. MySQL JDBC 드라이버를 사용하는 경우 Spring Boot 2.7.8 이상으로 업그레이드할 때 좌표를 업데이트해야 합니다.

#### 새로운 기능 및 주목할만한 사항
- Spring GraphQL 프로젝트 지원을 위한 새로운 spring-boot-starter-graphql 스타터 추가
- RabbitStreamTemplate 지원 추가
- Hazelcast @SpringAware 지원 추가
- Info 엔드포인트에서 운영 체제 정보 노출
- Info 엔드포인트에서 Java 공급자 정보 노출
- RSocket 핸들러 메서드에서 인증된 Principal에 액세스
- Opaque Token Introspection을 위한 종속성 제거
- Spring Data Couchbase 지원을 위한 @DataCouchbaseTest 추가
- Spring Data Elasticsearch 지원을 위한 @DataElasticsearchTest 추가
- SAML2 로그아웃을 위한 자동 구성 지원 추가

#### 자동 구성 변경 사항
- 자동 구성 등록을 위한 spring.factories에서 META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports로 이동
- @AutoConfiguration 주석 추가
- 테스트 슬라이스 구성을 META-INF/spring/<테스트 슬라이스 어노테이션 이름>.imports로 이동
- FailureAnalyzer 주입 변경

#### Redis Sentinel 사용자 이름 지원
- Sentinel을 인증하기 위한 사용자 이름 지정 지원 추가

#### 내장 웹 서버 SSL 구성을 위한 PEM 인코딩 인증서 사용
- PEM 인코딩 인증서 및 개인 키 파일을 사용한 내장 웹 서버의 SSL 구성을 지원
