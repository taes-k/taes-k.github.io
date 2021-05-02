---
layout: post
comments: true
title: Spring testcontainer를 이용한 독립 테스트환경 구축
tags: [spring, mysql]
---

### 독립 통합테스트 환경 구축하기 

통합 테스트를 하다보면, 외부 요인으로 인해 기대하던 데이터 혹은 메세지를 전달 받지 못해 테스트가 실패하는 경우가 빈번하게 발생 할 수 있습니다.  

이전에 비슷한 내용으로 포스팅 한 적이 있습니다. ([Spring H2를 이용한 독립 테스트환경 구축](https://taes-k.github.io/2021/04/05/spring-test-isolation-datasource/))

테스트 수행시 `H2`를 DB로 사용함으로서, 테스트를 수행 할 때 마다 독립적인 DB 환경을 구축해주는 내용의 포스팅이었는데 실무에 적용해보면서 우려했던 단점이 확인되었습니다.  

`H2` 에서 호환모드를 지원한다고 하지만, 100% 보장이 되지 않기때문에 실제 `MySQL`에서 동작하던 쿼리가 테스트 환경에서 에러를 발생시키는 몇가지 미호환 쿼리들이 존재합니다.

- MySQL `IF` 문 지원 하지 않음 -> `CASW WHEN THEN ELSE` 구문으로 치환 가능
- MySQL `DATE_FORMAT` 함수 지원하지 않음 -> `cast (date as datetime)`으로 어느정도 치환 가능
- MySQL `json_object('id', 123, 'pw', 123)` / H2 `json_object('id' : 123, 'pw' : 123)` 문법 다름 -> `concat ()` 구문으로 치환 가능
- MySQL `group_concat( xx orderby f1, f2)` -> suborder 기능 지원안함
- ...

최대한 `ANSI` 문법으로 통일하거나 우회하는 방법으로 호환을 맞출수는 있었으나, 테스트를 위해 기존의 코드를 수정해야 하고, 쿼리사용에 제약이 생긴다는 점 및 미상의 불호환 쿼리들이 있다는 점 등이 결국 큰 단점이 되었습니다.

따라서 결국에는 `MySQL`을 독립 통합테스트 환경을 구축하기 위해 테스트 수행시 `docker`를 이용해 `MySQL`을 띄우고 데이터소스로 사용하는 방법을 서치하던 중에 `Testcontainer` 라이브러리를 만나게 되었습니다.

---

### Testcontainer

> `Testcontainers` 
> 
> Testcontainers is a Java library that supports JUnit tests, providing lightweight, throwaway instances of common databases, Selenium web browsers, or anything else that can run in a Docker container.

`Testcontainer`는 Junit test 에서 docker 컨테이너를 사용 할 수 있도록 해주는 오픈소스 라이브러리입니다. 이 `Testcontainer`를 사용하게되면 우리는 다음과 같은 테스트 환경을 사용 할 수 있게 됩니다.

- 테스트 실행
- `MySQL` 컨테이너 실행
- 테스트 수행
- 테스트 종료
- `MySQL` 컨테이너 종료


![1]({{ site.images | relative_url }}/posts/2021-05-02-spring-test-container/1.png)   

`MySQL` 이외에도, 다양한 Database를 지원하고 있으니 프로젝트 에 맞는 DB를 선택해 사용 하실 수 있습니다.

![2]({{ site.images | relative_url }}/posts/2021-05-02-spring-test-container/2.png)   

Database 뿐만 아니라, `Localstack` module을 이용해 `AWS S3, SQS`등의 테스트 전용 서비스들을 구축해 사용 할 수도 있고, `Mockserver` module을 통해 테스트가 어려운 `MSA` 환경에서도 독립적인 테스트를 수행 할 수 있습니다. 이 외에도 `Kafka`, `Elasticsearch` 등등 많은 module들을 지원하기때문에  

---
### Testcontainer MySQL 환경 구축하기

이제 실제 프로젝트에 `Testcontainer MySQL`을 적용해서 수행해보도록 하겠습니다.

```groovy
// build.gradle

    testImplementation "org.testcontainers:testcontainers:1.15.3"
    testImplementation "org.testcontainers:junit-jupiter:1.15.3"
    testImplementation "org.testcontainers:mysql:1.15.3"
```

```yml
## application.yml

...


---

spring:
  profiles: junit-test

datasource:
    driver-class-name: org.testcontainers.jdbc.ContainerDatabaseDriver
    url: jdbc:tc:mysql:5.7.22:///wsin_dev/wsin_dev?user=root?password=;

...

---

```

```java
// TCIntegrationTest.java

@Testcontainers
@ActiveProfiles("junit-test")
@SpringBootTest
@AutoConfigureMockMvc
public abstract class TCIntegrationTest
{
    @Autowired
    protected MockMvc mockMvc;
}
```

사용법은 정말 간단합니다. Test 코드에 `@Testcontainers` 어노테이션만 추가후 `@ActiveProfiles`를 통해 `Testcontainer`에서 띄워진 `MySQL`을 사용 할 수 있도록 'driver-class-name'과 'url'을 지정해주시면 실제 DB 사용하는것과 동일하게 DB를 사용 할 수 있습니다.

이때 주의하셔야 할 것은 `org.testcontainers.jdbc`에서 제공하는 driver-class-name을 사용하셔야 한다는 것 입니다.


---
### Testcontainer Docker-compose 환경 구축하기

위 방법을 사용하게되면 Default 설정으로 `MySQL`이 실행되어 `encoding`, `TimeZone` 등의 설정이 불가능하고 여러개의 Datasource를 사용할때에 설정이 어렵기 때문에 `docker-compose`를 사용해 설정하는 방법을 하나 더 공유드립니다.


```groovy
// build.gradle

    testImplementation "org.testcontainers:testcontainers:1.15.3"
    testImplementation "org.testcontainers:junit-jupiter:1.15.3"
```

```yml
## test/resources/docker-compose.yml

version: '3.2'

services:
  wsin-mysql:
    image: mysql/mysql-server:5.7
    environment:
      MYSQL_ROOT_HOST: '%'
      MYSQL_DATABASE: 'test_db'
      MYSQL_ALLOW_EMPTY_PASSWORD: 'yes'
      TZ: Asia/Seoul
    ports:
      - '33006:3306'
    volumes:
      - ./init-schema.sql:/docker-entrypoint-initdb.d/init.sql

    command:
      - 'mysqld'
      - '--character-set-server=utf8mb4'
      - '--collation-server=utf8mb4_unicode_ci'
```

일반적인 `docker-compose`를 설정하는 방법으로 똑같이 구성하면 됩니다. init script를 미리 `docker-entrypoint-initdb.d/` 하단에 위치시켜 초기 스키마 및 데이터를 구성 할 수도 있습니다.

```yml
## application.yml

...


---

spring:
  profiles: junit-test

datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:33006/test_db?serverTimezone=UTC

...

---

```

위에서 `testcontainer` 전용 driver를 사용한 것과 다르게, 일반적으로 사용하는 driver와 url도 `docker-compose`에서 매칭시킨 `localhost:port`를 사용해서 작성 할 수 있습니다. 


```java
// TCIntegrationTest.java

@Testcontainers
@ActiveProfiles("junit-test")
@SpringBootTest
@AutoConfigureMockMvc
public abstract class TCIntegrationTest
{
    @Autowired
    protected MockMvc mockMvc;

    static final DockerComposeContainer composeContainer;

    static
    {
        composeContainer = new DockerComposeContainer(new File("src/test/resources/docker-compose.yml"));
        composeContainer.start();
    }
}
```

```java
// SomethingIntegrationTest.java

public class SomethingIntegrationTest extends TCIntegrationTest
{
    @Test
    void some_test()
    {
        ...
    }
}

```

위와같이 구성을하게되면, 통합 테스트수행시 `container`를 공유해서 사용 할 수 있습니다.  
`docker-compose` 를 이용하는 것 이기 때문에, 예제에서는 `MySQL` 만을 예제로 들었지만, 이외에 `docker-image`를 사용하는 어떤 것 이든 `container`에 띄워서 독립된 테스트를 수행 하실수 있습니다.

---

### 마무리

`docker container`를 사용하기때문에 시간이 좀 더 걸릴 수 있다는점, `CI`등의 서버에서 이용시 메모리 사용량에 주의해야 한다는 점 등의 고려해야할 사항이 있지만, `testcontainer`를 통해 이전에 고민했던 ([Spring H2를 이용한 독립 테스트환경 구축](https://taes-k.github.io/2021/04/05/spring-test-isolation-datasource/)) 환경구축 내용을 보완 할 수 있는 완전 독립적인 테스트 환경을 구축 할 수 있었습니다.  

게다가 DB 뿐만아니라 연결되어있는 시스템이 많다면 외부 시스템 고려없이 테스트를 할 수 있다는 점에서 앞으로도 효용성이 높게 사용 될 수 있는 오픈소스 라이브러리 인것 같습니다.


---

### Reference

- https://www.testcontainers.org/
- https://woowabros.github.io/tools/2019/07/18/localstack-integration.html
- https://riiidtechblog.medium.com/testcontainer-%EB%A1%9C-%EB%A9%B1%EB%93%B1%EC%84%B1%EC%9E%88%EB%8A%94-integration-test-%ED%99%98%EA%B2%BD-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0-4a6287551a31
