---
layout: post
comments: true
title: JPA의 대안으로 Spring Data JDBC 
tags: [spring, jdbc, springdatajdbc]
---

### 단순한 Spring Data JDBC

이전 포스팅 내용에 이어지는 포스팅입니다. 먼저 단순한 사용을 지향하는 Spring Data JDBC에 대해서 조금더 알아보도록 하겠습니다.

#### 객체매핑

테이블 구조에 대응된 객체 매핑 기능들을 제공합니다.  
JPA와 비슷한 방식으로 `@Table, @Id, @Column`등의 어노테이션을 활용해 객체 매핑을 정의 할 수 있습니다.

```java
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

#### 객체 persistence

객체자체로 실제 데이터와의 영속성을 유지하지않습니다. 엔티티를 저장하기 위해서는 `CrudRepository.save(…)` 메서드 호출을 통해 저장시켜야합니다.

```java
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;

    public int createNewUser(UserParam userParam){
        User newUser = User.from(userParam);

        return userRepository.save(newUser); // user insert
    }
}
```

또한 캐시와 LazyLoading을 지원하지 않기때문에 Entity가 로드되었을때 해당 객체는 완전히 로드가 완료된 Entity 입니다. 추가적인 지연쿼리등이 호출되지 않습니다.

#### 커스텀 쿼리

커스텀 쿼리를 사용하기위한 방식도 간단하게 사용가능합니다.   
JPA와 비슷하게 `@Query` 어노테이션을 활용해 사용하거나 MyBatis와 같은 방식으로 직접 쿼리 스트을 전달해 사용하는방식도 쓸수 있습니다.

```java
public interface UserRepository extends CrudRepository<User, Long> {
    @Query("SELECT * FROM user WHERE lst_nm = :lastname")
    List<User> findByLastName(String lastName);

    @Query("SELECT * FROM user WHERE lst_nm = :lastname")
    List<User> findByLastName2(String lastName);
}
```
```java
@RequiredArgsConstructor
public class CustomUserRepositoryImpl extends UserRepository {
    
    private final NamedParameterJdbcTemplate jdbcTemplate;

    @Override
    public List<User> findByLastName2(String lastName){
        String sql = "SELECT * FROM user WHERE lst_nm = :lastname";
        Map<String, Object> params = Map.of("age", age);

        return jdbcTemplate.query(
            sql, 
            params, 
            new BeanPropertyRowMapper<>(User.class)
        );
    }ß
}
```

---


### JPA의 알려진 이슈들

다들 알고 계시겠지만 `JPA`는 객체관계 매핑을 통해 자바 객체와 데이터베이스 테이블 간의 매핑을 수월하게 처리해 SQL 쿼리 작성 및 데이터베이스 접근과 관련된 복잡한 작업을 줄여 현재 개발에서 필수로 사용되는 필수 프레임워크으로 자리잡았습니다. 

`JPA`의 핵심 개념 중 하나는 영속 컨텍스트(Persistence Context)인데, 영속 컨텍스트는 엔터티 객체의 생명 주기를 관리하고, 데이터베이스와의 효율적인 통신을 지원하는 핵심 메커니즘입니다. 그러나 영속 컨텍스트를 활용함으로서 발생하는 잘 알려진 몇 가지 이슈들이 있는데 이러한 이슈들을 이해하고 적절히 다루는 것이 JPA를 효과적으로 활용하는 핵심입니다.

#### N+1 문제

`N+1` 문제는 가장 잘 알려진 이슈라 다들 잘 알고 계실것이라고 생각됩니다. 연관된 엔티티를 지연로딩으로 조회시 매번 추가 쿼리호출을 하게되는 이슈입니다. 

```java
@Data
@Entity
public class ParentEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "parent", fetch = FetchType.LAZY)
    private List<ChildEntity> children;
}

@Data
@Entity
public class ChildEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String childName;

    @ManyToOne
    @JoinColumn(name = "parent_id")
    private ParentEntity parent;
}
```
```java
@RequiredArgsConstructor
@Service
public class SampleService {

    private final ParentRepository parentRepository;

    @Transactional
    public void doSomething(){
        ParentEntity parent = parentRepository.getParent();
        List<ChildEntity> children = parent.getChildren(); // 추가 N번 쿼리 호출
        for (ChildEntity child : children) {
            // doSomething
        }
    }
}
```

#### LazyInitializationException 이슈

`JPA`를 사용하면서 자주 발생 할 수 있는 이슈로써, 영속성 컨텍스트를 벗어난 상태의 Entity 내 지연 로딩 Entity에 접근하려고 할 때 발생하는 exception입니다.

```java
@RequiredArgsConstructor
@Service
public class SampleService {

    private final ParentRepository parentRepository;

    public void doSomething(){
        ParentEntity parent = parentRepository.getParent(); // 영속성 컨텍스트 종료
        List<ChildEntity> children = parent.getChildren(); // LazyInitializationException
        for (ChildEntity child : children) {
            // doSomething
        }
    }
}
```

#### 그외에도...

`JPA`는 영속성 Context를 유지하는것이 핵심이기때문에 이에 따른 다양한 부수효과들을 잘 알고 고려하여 개발 해야합니다.  
- 쓰기지연 기능
- 영속성 컨텍스트 용량관리
- entity 변경을 위한 `DirtyChecking` 기능

---

### 대안으로서의 Spring Data JDBC

`Spring Data JDBC`의 가장 큰 장점을 말하자면, 단순하게 데이터를 조회하고, entity를 수정하고, 저장합니다.  
JPA와 비슷하게 객체 매핑 기능을 지원해 간편한 데이터 관리를 할 수 있게 하면서도, 영속성 컨텍스트는 사용하지 않아 JPA보다는 단순한 방법으로 데이터를 관리 할 수 있다는점 일 것 입니다.  

연관된 객체 매핑을 lazyLoading으로 처리하기보다는 필요할때 직접 조회하는 방식을 통해 `JPA`에서 발생하는 `N+1 쿼리 문제`, `LazyInitializationException` 이슈들을 자연스럽게 회피 할 수도 있습니다.

```java
@Data
@Entity
public class ParentEntity {
    @Id
    private Long id;

    private String name;
}

@Data
@Entity
public class ChildEntity {
    @Id
    private Long id;

    private String childName;

    private Long parentId;
}
```
```java
@RequiredArgsConstructor
@Service
public class SampleService {

    private final ParentRepository parentRepository;
    private final ChildRepository childRepository;

    public void doSomething(){
        ParentEntity parent = parentRepository.getParent();
        List<ChildEntity> children = childRepository.getChildrenByParentId(parent.getId());
        for (ChildEntity child : children) {
            // doSomething
        }
    }
}
```

중간 영속 컨텍스트 없이 단순하게 필요할때 DB에 직접 접근해서 데이터를 조작한다는 관점에서 훨씬 단순하고 직관적이라는 점이 JPA의 좋은 대안이 될 수 있다고 생각합니다.  

추가로 객체 변경에 대해서도 entity 객체를 직접 수정할 필요는 없기 때문에 `불변 객체`를 사용 가능하다는 장점이 있습니다. 만약 `Kotlin`언어를 사용하고 계시다면 더욱이 시너지가 날 수 있는 프레임워크가 될 것입니다.


```kotlin
@Entity
data class ParentEntity(
    @Id
    val id: Long,

    val name: String
) {
}

@Entity
data class ChildEntity(
    @Id
    val id: Long,

    val parentId: Long,

    val name: String
) {
}
```
```kotlin
@RequiredArgsConstructor
@Service
public class SampleService(
    parentRepository: ParentRepository,
    childRepository: ChildRepository
) {
    fun doSomething(){
        val parent = parentRepository.getParent()
        val modifiedParent = parent.copy(
            name = "modifiedName"
        )
        
        parentRepository.save(modifiedParent)
        ...
    }
}
```

---

### 마무리

혹시나 오해가 있을수 있어 한번더 말씀드리자면 이 포스팅의 목적이 `JPA` 대신에 `spring data JDBC` 를 사용하자는 의미는 아닙니다. 다만 `JPA`를 사용하시면서 어려움을 느끼고 계신분이나 조직이 있다면 좋은 대안이 될 수 있을것이라 생각합니다.

좀더 자세한 내용을 아래 영상들이 조금 더 도움이 될 수 있으리라 생각됩니다.

- [우아한테크세미나-우아한CRUD](https://www.youtube.com/watch?v=cflK7FTGPlg)
- [The Complexity of JPA](https://www.youtube.com/watch?v=RGz-Vn9fw04)

---

### reference
- [스프링 Document](https://docs.spring.io/spring-data/jdbc/docs/3.1.0/reference/html/#reference)