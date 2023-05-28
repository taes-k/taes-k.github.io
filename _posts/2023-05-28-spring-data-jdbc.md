---
layout: post
comments: true
title: Spring Data JDBC 소개
tags: [spring, jdbc, springdatajdbc]
---

### Spring Data JDBC 소개

이번 포스팅에서는 저희 팀에서도 활발하게 사용하고 있는 `Spring Data JDBC`에 대한 가벼운 소개를 해보려고 합니다. 혹시 처음 들어보셨다고 하더라도 이름만 보면 `spring` 에서 공식적으로 지원하는 `data`를 다루기위한 `jdbc` 프레임워크이겠구나 라고 알수 있으실듯한데요, 간단하게 소개드리자면 Spring에서 JDBC 기반의 간단하고 가벼운 데이터 액세스 계층을 제공하는 프레임워크라고 소개 할 수 있겠습니다.

먼저 이해를 돕기위해 비슷한 프레임워크로 거론되는 프로젝트로는 `MyBatis`,`JPA` 등이 빠지지않고 이야기되는데요, Spring Data JDBC의 특징은 아래와 같다고 할 수 있습니다.
- `ORM`을 사용하지 않고 JDBC를 통해 데이터베이스와 상호작용
- 최소한의 매핑을 통해 `단순성`과 `생산성`을 높임
- 쿼리의 `명확성`과 성능을 유지

---

### MyBatis, JPA 그리고 Spring Data JDBC

위에서 소개드린 내용만으론느 아직은 `Spring Data JDBC`가 정확히 어떤 이점이 있는지 긴가민가 하실수 있으실듯해서 조금더 이해하기 쉽도록 `MyBatis`, `JPA`와의 차이를 코드를 통해 설명을 드려보도록 하겠습니다.

- MyBatis
    ```java
    // MyBatis

    @Data
    public class User {
        private Long id;
        private String firstName;
        private String lastName;
    }

    public interface UserMapper {
        List<User> findByLastName(@Param("lastName") String lastName);
    }

    <!-- MyBatis XML Query -->
    <select id="findByLastName" resultType="User">
        SELECT * FROM users WHERE last_name = #{lastName}
    </select>
    ```
- JPA
    ```java
    // jpa

    @Entity
    @Table(name = "user")
    @Data
    public class User {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
        private String firstName;
        private String lastName;
    }

    public interface UserRepository extends JpaRepository<User, Long> {
        List<User> findByLastName(String lastName);
    }
    ```
- Spring Data JDBC
    ```java
    // Spring Data JDBC

    @Table("user")
    @Data
    public class User {
        @Id
        private Long id;
        private String firstName;
        private String lastName;
    }

    public interface UserRepository extends CrudRepository<User, Long> {
        List<User> findByLastName(String lastName);
    }
    ```

위 코드들로만 비교해보았을때는 `MyBatis`와 `Spring Data JDBC`의 차이가 매핑로직의 유무로 확실하게 보이는것을 확인하실수 있습니다. MyBatis는 SQL 매핑을 위해 XML 파일에 쿼리를 직접 정의하여 사용함으로써, 객체와 테이블 간의 매핑을 선언적으로 정의하고 사용합니다.  

Spring Data JDBC는 최소한의 매핑을 지향하여 객체와 테이블 간의 매핑을 선언적으로 정의하지 않고 사용 가능하며 자동생성 쿼리를 사용할수 있어 코드가 더 간결한 코드수행이 가능합니다.

위에서 지향하는 방향은 사실 `JPA`코드에서도 동일하게 나타나는것을 확인하실수 있습니다. 따로 설명드리지 않아도 `JPA`는 주요한 `ORM(Object-Relational Mapping)` 프레임워크로서 다음과 같은 주요 특징들을 갖고 있다고 볼 수 있습니다.

- 객체와 테이블 관계 매핑
- 자동화된 SQL 생성
- 포괄적인 영속성 관리
- 캐싱 기능
- SQL 추상화

위 예시 코드로만 봐서는 `Spring Data JDBC`역시 `ORM 프레임워크 인가?` 라고 보실수도 있겠으나 `Spring Data JDBC`는 `ORM` 프레임워크와는 다르게 순수한 JDBC를 사용하여 데이터베이스 액세스를 지원하는 프레임워크라는 점에서 차이를 가집니다.

---

### Spring Data JDBC 의 지향방향

`Spring Data JDBC`는 `JPA`와는 설계적으로 다른 지향점을 갖음으로서 조금더 단순한 사용을 목표로 하고있는것을 [공식문서](https://docs.spring.io/spring-data/jdbc/docs/3.1.0/reference/html/#jdbc.why)를 통해 확인하실수 있습니다.

> - If you load an entity, SQL statements get run. Once this is done, you have a completely loaded entity. No lazy loading or caching is done.
> - If you save an entity, it gets saved. If you do not, it does not. There is no dirty tracking and no session.
> - There is a simple model of how to map entities to tables. It probably only works for rather simple cases. If you do not like that, you should code your own strategy. Spring Data JDBC offers only very limited support for customizing the strategy with annotations.

layLoading 없고, 캐싱 없고, 영속성 객체 없고, 복잡한 객체매핑 어노테이션이 없이 단순하게 도메인기반으로 데이터-객체 매핑을 지원하는 설계방향에서 ORM 프레임워크인 JPA와의 차이점을 알 수 있으실것으로 생각됩니다.

---

### reference
- [스프링 Document](https://docs.spring.io/spring-data/jdbc/docs/3.1.0/reference/html/#reference)
