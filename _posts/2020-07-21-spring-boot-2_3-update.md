---
layout: post
comments: true
title: Spring boot 2.3.x update
tags: [spring, springboot]
---

### 요약

Spring boot 2.3 주요 패치사항을 요약해보자면, 다음과 같은 기능들이 강화되었습니다.
- Docker Image
- Custom Layer
- k8s monitoring
- Graceful shutdown

전반적으로 Container 기술을 효율적으로 사용하기 위한 기능들이 이번버전에서 업데이트 되었다고 할 수 있을것 같습니다.  

자세한 내용은 다음 url에서 확인 하실수 있습니다.

https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.3-Release-Notes

---
### 상세 업데이트 내용

#### Spring Data Major update
- JPA Bootstrap mode 지원
- Cassandra v4 update
- Couchbase v3 update
- Native Elasticsearch, jest library deprecate
- MongoDB 4 update
- Neo4j default interceptor update

#### Micrometer
- v1.5 update

#### Jackson
- v2.11 update
- default formatting 변경 (java.util.date, java.util.Calender)

#### Spring cloud connectors starter
- deprecate

#### Embedded Servlet web server threading configuration
- embedded servlet thread 설정 그룹이 변경 (server.tomcate.threads, server.jetty.threads ...)

#### Changes to the Default Error Page’s Content
- 더이상 에러 메세지 및 error에 default error 페이지를 포함하지 않습니다.

#### Disk space health indicator
- 설정시 path가 필요하지 않습니다.

#### Automatic creation of developmentOnly Gradle configuration
- Spring Boot’s DevTools 에 필요했던 'developmentOnly' 속성이 Spring boot gradle plugin에서 자동으로 생성해줍니다.
- gradle build script에 'developmentOnly'로 수동 설정되어있는 속성들은 삭제해도 됩니다.

#### Removal of Maven Site Plugin management
#### Removal of Maven Exec Plugin custom configuration
- Spring Boot’s 는 더이상 빌드에 'maven-site-plugin'를 사용하지 않습니다.

#### ApplicationContextRunner disables bean overriding by default
- 'SpringApplication', 'ApplicationContextRunner' 빈을 override 할 수 없습니다.
- test를 위해 override가 필요할 시 'withAllowBeanDefinitionOverriding'를 사용해야합니다.

#### Activating multiple profiles with @ActiveProfiles
- @ActiveProfiles("p1,p2") -> @ActiveProfiles({"p1","p2"})

#### WebServerInitializedEvent and ContextRefreshedEvent
- 정상종료를 위해, Web server 초기화는 어플리케이션 컨텍스트의 refresh가 완료될때 수행됩니다. (기존에는 refresh process 끝난 직후)

#### Deprecations from Spring Boot 2.2
- Spring boot 2.2에서 deprecate 였던 class, method들이 삭제되었습니다.

#### Java 14 support
- Java 14를 지원합니다.

#### Build OCI images with Cloud Native Buildpacks
- Docker image build를 지원합니다.
- spring-boot:build-image goal && bootBuildImage task

#### Build layered jars for inclusion in a Docker image
- 컨텐츠가 레이어로 분리된 jar 파일을 빌드한느 기능이 maven/gradle plugin 에 추가되었습니다. (변경된 레이어만 빌드)
- Docker image build에 효과적입니다.

#### Predictable Classpath Ordering When Exploding Fat Jars
- Fat Jar 에 인덱스가 추가되어 class경로의 순서를 보장합니다.

#### Support of wildcard locations for configuration files
- config file의 wild card를 지원합니다. (/config/*/)
- k8s에서 여러 config property를 구성할때 효과적입니다.

#### Graceful shutdown
- Jetty, Reactor Netty, Tomcat, Undertow 에서 정상종료를 지원합니다.
- server.shutdown=graceful 옵션
- 종료하는동안 request를 허용하지 않습ㄴ디ㅏ.

#### Liveness and Readiness probes
- 어플리케이션 가용에 대한 정보가 내장됩니다. (app 존재여부, 트래픽 처리 ready 상태 확인)
- management.health.probes.enabled = true
- 쿠버네티스를 연결하면 바로 확인이 가능합니다.

#### Spring Data Neumann
- R2DBC 지원

#### Configurable base path for WebFlux applications

#### Date-Time conversion in web applications
- 날짜 - 시간값 변환을 속성 설정으로 구성 할 수 있습니다.

#### Actuator Improvements
- End-to-end Traceability for Configuration Properties
- Names in metrics endpoint are ordered alphabetically
- Query-less datasource health indicator
- Contributing additional tags to Web MVC and WebFlux metrics
- Auto-configuration of Wavefront’s Sender
- Native Kafka metrics

#### RSocket support for Spring Integration
- Spring Integration RSocket을 지원합니다.

#### Binding to Period
- property에서 java.util.period를 사용 할 수 있습니다.

#### Slice test for Web Services
-  @WebServiceClientTest 에서 slice test를 지원합니다.
