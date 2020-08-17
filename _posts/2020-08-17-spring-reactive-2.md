---
layout: post
comments: true
title: R2DBC mysql (Spring reactive-2)
tags: [spring, reactive, r2dbc, mysql]
---

### R2DBC
포스팅 작성일 기준 (2020.08.17) Spring boot 최신 버전인 `spring-boot-2.3.x` 에서는 신기술을 탐닉하는 스프링진영의 개발자들이라면 기다려왔을 `R2DBC`의 `GA (General Availability)`버전이 `Spring-data` 프로젝트에 포함되었습니다.

The Reactive Relational Database Connectivity (`R2DBC`) 는 이름 그대로 RDB를 Reactive하게 사용 할 수 있도록 API를 제공하는 프로젝트입니다.  

기존에 실무에서 Reactive programming을 적용하지 못했던 가장 큰 이유를 들자면, 'Reactive를 제공하지않는 대부분의 Blocking 기반의 RDB'를 꼽아볼수 있을것입니다.  

`R2DBC`는 완전한 Reactive RDB 사용을 위해 non-blocking I/O layer 위에 Database 유선 프로토콜 계층까지 완전히 새롭게 구현한 드라이버를 제공합니다.  
이를통해서 Reactive를 RDB로 서비스할때, Reactive 한 척을 하는것이 아닌, 실제로 Reactive하게 동작 할 수 있도록 해줍니다.

현재 공식적으로 지원하는 DB는 `H2`, `MSSQL`, `Postgre` 3개의 DB를 지원하고 있으나, 별도의 추가 프로젝트를 통해 `MySQL`, `MariaDB`까지도 지원 하고 있습니다.

`R2DBC`에 대한 자세한 사항은 공식홈페이지에서 확인하실수 있습니다.  
(https://r2dbc.io)

---

### R2DBC 사용해보기

실제 Spring에서 R2DBC를 적용하는 코드를 알아보도록 하겠습니다.
```
// build.gradle

implementation 'org.springframework.boot:spring-boot-starter-data-r2dbc'
compile 'dev.miku:r2dbc-mysql:0.8.0.RELEASE'
annotationProcessor 'org.projectlombok:lombok'
compile 'org.projectlombok:lombok:1.16.20'

```

```
// application.properties

spring.r2dbc.url=r2dbc:mysql://localhost:3306/test_schema?useUnicode=true&characterEncoding=utf8
spring.r2dbc.username=root
spring.r2dbc.password=root123
```

```java
// User.java

import lombok.Data;

@Data
public class User {
    private String id;
    private String name;
    private String phoneNumber;
    private String address;
    private String registDate;
}
```

```java
// UserRepository.java

import com.taes.r2dbc.entity.User;
import org.springframework.data.r2dbc.repository.R2dbcRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface UserRepository extends R2dbcRepository<User, String> {
}

```

```java
// UserService.java

import com.taes.r2dbc.entity.User;
import com.taes.r2dbc.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Flux;

@Service
public class UserService {

    private final UserRepository userRepository;

    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public Flux<User> getAllUser()
    {
        return userRepository.findAll();
    }
}
```
위와같이 매우 간단하게, DB로 부터 쿼리 결과를 `Reactive stream`의 `Flux` 혹은 `Mono`로 return 받아 올 수 있습니다.  

![1]({{ site.images | relative_url }}/posts/2020-08-17-spring-reactive-2/1.jpg)   
(참 쉽죠?)

`JPA`를 사용해 보신 분이라면, 사용법이 거의 비슷해 `R2DBC`도 `ORM`이라고 생각 하실 수도 있겠으나, `R2DBC`에서는 다음과같이 명시하고 있습니다.

> Spring Data R2DBC aims at being conceptually easy. In order to achieve this it does NOT offer caching, lazy loading, write behind or many other features of ORM frameworks. This makes Spring Data R2DBC a simple, limited, opinionated object mapper.

`R2DBC`를 쉽게 사용 할 수 있도록 `ORM`의 사용법을 참고하여 설계를 하였으나, `ORM Framework`에서 제공하는 다양한 기능들을 제공하지 않습니다.


---

### R2DBC 성능 비교

쓰는방법을 설명 드렸으니, 이제 실제 실무에서 쓸만 할까를 검증하기 위해 성능비교를 한번 해봐야겠죠?

테스트에 사용된 스펙과 버전 정보는 다음과 같습니다.

- Spring boot 2.3.2
- Java8
- MySQL5.8

테스트를 위해, 다음과 같은 구조의 테이블 구조가 있다고 생각해주시면 되겠습니다.

![2]({{ site.images | relative_url }}/posts/2020-08-17-spring-reactive-2/2.png)   

R2DBC의 비교군으로, JDBC를 사용한 프로젝트를 사용하도록 하겠습니다.  
테스트 측정 대상은 다음과 같습니다.
- Spring에서 첫번째 데이터 접근 시간
- Spring에서 데이터 처리 완료 시간


수행 쿼리 (select 약 1,000,000건)

```sql
select p.id product_id, p.product_name, p.product_detail, u.id user_id, u.name user_name, l.id location_id, l.name location_name 
from 
	PRODUCT_SALE ps 
    INNER JOIN PRODUCT p 
		ON ps.product_id = p.id
	INNER JOIN USER u
		ON ps.user_id = u.id 
	INNER JOIN LOCATION l
		ON ps.location_id = l.id
```

`JDBC` 수행 결과

![3]({{ site.images | relative_url }}/posts/2020-08-17-spring-reactive-2/3.png)   

`R2DBC` 수행결과

![4]({{ site.images | relative_url }}/posts/2020-08-17-spring-reactive-2/4.png)   

위 수행결과로 알수있는 내용은, 전체 수행 결과로는 `JDBC`가 빠르지만 첫 데이터 처리는 당연하게도 `R2DBC`가 빠르게 처리게 된 것을 확인 할 수 있습니다. 


`R2DBC`가 `Reactive stream`을 만들어내면서 내부적으로 `Flux`와 `Mono`의 Wrapping 과정으로인해 SELECT에서 벤치마킹상 `JDBC`의 약 60%의 성능을 가질수 있다고 개발팀에서 안내 하고 있습니다. (https://github.com/spring-projects/spring-data-r2dbc/issues/203)

위 퍼포먼스 테스트는 `JDBC`와 `R2DBC`간의 비교를 한 내용으로, 프로그램의 성능을 테스트한 내용이 아닌, 단순히 DB 조회 성능을 비교한 것 입니다.  
따라서 `Reactive programming`의 성능이 더 좋지 않게 나왔다 라고 오해하지 않으시길 바랍니다.

---

### 마무리

점점 더 `Reactive programming`의 생태계를 확장할 수 있도록 개발 업데이트 사항들이 지원되고 있습니다.  
특히 가장 큰 걸림돌이었던 RDB에서의 `Reactive`를 지원하는 `R2DBC`의 `GA` 버전 런칭으로 고가용성의 `Reactive`한 서비스 개발을 실무에서도 고려 해 볼수 있을것 같습니다.  

하지만 위 성능 테스트에서 보셨듯이, 모든 프로그램을 단순하게 `Reactive`하게 바꾼다고 성능이 좋아지지는 않을것입니다. (오히려 마이너스가 될 수 있습니다.)  

성능에 초점을 두기 보다는 말 그대로 `Stream`을 만들 수 있는 서비스에서, 고가용적인 프로그램을 만들고자 할 때 `R2DBC`를 활용해 `Reactive programming`을 적용하신다면 좋은 선택이 되실 수 있을것 같습니다.


