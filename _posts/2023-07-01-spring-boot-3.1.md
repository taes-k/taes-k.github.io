---
layout: post
comments: true
title: Spiring boot 3.1.0 update
tags: [spring, springboot]
---

### Spiring boot 3.1.0 update  

Spring boot 3.1.0 update 사항 정리 (2023. 05. 18)

원문 
- https://spring.io/blog/2023/05/18/spring-boot-3-1-0-available-now/
- https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.1-Release-Notes

---

### 내기준 주요 변경사항

- [Git 커밋 ID Maven 플러그인 버전 속성 변경](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.1-Release-Notes#git-commit-id-maven-plugin-version-property)
    - maven id 변경사항 수정 필요
- [테스트 컨테이너 종속성 관리](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.1-Release-Notes#dependency-management-for-testcontainers)
    - 테스트 컨테이너가 spring boot 에서 종속성 직접 관리
- Java EE -> Jakarta EE
    - jakarta api 으로 마이그레이션 필요
- [GraalVM 지원](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Release-Notes#graalvm-native-image-support)
    - 여러 행사에서 이미 공개되왔던 graalVM 을 지원함
    - https://docs.spring.io/spring-boot/docs/3.0.0/reference/html/native-image.html#native-image

---

### summary by chatGPT


#### Spring Boot 3.0에서의 업그레이드

- Apache HttpClient 4에 대한 종속성 관리:
  - Spring Boot 3.0에서는 HttpClient 4 및 5의 종속성 관리가 포함되어 있었습니다.
  - Spring Boot 3.1에서는 HttpClient 4의 종속성 관리가 제거되어 HttpClient 5로 이동하도록 권장합니다.

#### Servlet 및 Filter 등록

- `ServletRegistrationBean` 및 `FilterRegistrationBean`은 이제 등록이 실패하면 `IllegalStateException`을 throw합니다.

#### Git Commit ID Maven Plugin Version Property 변경

- `git-commit-id-maven-plugin` 버전을 재정의하기 위한 프로퍼티가 업데이트되었습니다.

#### Spring Kafka Retry Topic 자동 구성 변경

- Apache Kafka에 대한 자동 구성된 재시도 가능한 토픽 구성의 변경사항.

#### Testcontainers의 Dependency Management 추가

- Spring Boot에 이제 Testcontainers가 종속성 관리에 포함되었습니다.

#### Hibernate 6.2

- Spring Boot 3.1에서는 Hibernate 6.2로 업그레이드되었습니다.

#### Jackson 2.15

- Spring Boot 3.1에서는 Jackson 2.15로 업그레이드되었습니다.

#### Spring Cloud Stream Binder Kafka

- Spring Cloud Stream Binder Kafka를 지원하는 새로운 모듈이 추가되었습니다.

#### Spring Authorization Server

- Spring Authorization Server 프로젝트 지원과 함께 새로운 스타터가 추가되었습니다.

#### Docker 이미지 빌딩

- Docker 이미지의 생성 날짜 및 애플리케이션 디렉터리를 설정하기 위한 옵션들이 추가되었습니다.

#### Spring for GraphQL

- 예외 처리 개선 및 페이징/정렬 지원이 추가되었습니다.

#### Testcontainers

- 개발 시간에 Testcontainers를 사용하는 지원이 추가되었습니다.

#### Service Connection 개념 도입

- 새로운 서비스 연결 개념이 도입되었습니다.

#### Property-based ConnectionDetails Beans

- 자동 구성들은 이제 구성 프로퍼티를 기반으로 자체 ConnectionDetails를 정의합니다.

#### Docker Compose

- Docker Compose 통합을 위한 새로운 모듈이 추가되었습니다.

#### SSL Configuration

- SSL 신뢰 자료 구성이 개선되었습니다.

#### Spring Authorization Server를 위한 Auto-configuration

- Spring Authorization Server를 지원하기 위한 새로운 스타터가 추가되었습니다.

#### Dependency Upgrades

- 다양한 Spring 프로젝트 및 서드파티 종속성들의 업데이트가 이루어졌습니다.

#### Miscellaneous

- 여러 작은 튜크 및 개선사항들이 추가되었습니다.

#### Spring Boot 3.1.0에서의 폐기 예정 항목

- 폐기 예정된 프로퍼티 및 클래스들이 있습니다.