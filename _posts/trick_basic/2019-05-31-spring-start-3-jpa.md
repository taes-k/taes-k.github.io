---
layout: post
comments: true
title: Spring 프로젝트 시작하기 (3) - JPA
tags: [요령과 기본, Spring]
date: 2019.05.30
---

### 2.3 Spring 프로젝트 시작하기 (3) - JPA
{: .no_toc }
아직 JPA를 모르시는 분들은 'JDBC, MyBatis 했으면 다 되는데 JPA는 또 무엇인가?', JPA를 알고 계시지만 사용해보시지 않았거나 매력을 못느끼셨던 분들은 'JPA의 큰 장점을 모르겠다.', '굳이 JPA?' 하는 의문을  가지시고 계실것 같습니다. 저 또한 가지고있던 의문이었는데, 이번챕터에서는 JPA에대해 알아보도록 하겠습니다.  
  

### Contents List
{: .no_toc .text-delta }

1. TOC
{:toc}

---

### 2.3.1) ORM
  
ORM(Object Relational Mapping)이란 RDB(관계형 데이터베이스) 테이블을 객체지향으로 사용하기위한 기술입니다. 즉, 데이터베이스의 테이블을 객체화시켜 OOP다운 프로그래밍을 할수잇도록 도와주는 툴 입니다. 이를통해 객체지향적인 코드로써 더 직관적으로 서비스를 표현할수 있으며 비즈니스 로직에 더 집중 할 수 있도록 도와줍니다.   

  
---

### 2.3.2) Spring JPA
  
JPA는 자바 ORM에 대한 API 표준 명세로써, ORM을 사용하기위한 인터페이스를 모아둔 것입니다. JPA를 직접적으로 사용 하기위해서는 Hibernate 같은 ORM Framework 를 사용해야합니다.   
  
이전 챕터에서 JDBC로 연결했던 프로젝트를 JPA 수정하면서 JPA의 구조에대해서 알아보도록 하겠습니다.   
가장먼저, 데이터베이스를 연결하기 위해 설정을 진행해 주도록 하겠습니다.

---

#### JPA 동작 구조  

![1]({{ site.images | relative_url }}/posts/trick_basic/2019-05-31-spring-start-3-jpa/1.png)   

jpa는 jdbc와 별개로 동작하는것이 아닌, JDBC를 내포하고 있으며, 데이터베이스와의 연결에 JDBC를 사용하게 됩니다.  

![2]({{ site.images | relative_url }}/posts/trick_basic/2019-05-31-spring-start-3-jpa/2.png)   

jpa는 Entity Manager를 통해 테이블과 매핑된 Entity들의 저장,수정,삭제,조회 등의 동작들을 수행시키고 관리합니다. 이때 영속성 컨텍스트(persistence context )라는 어플리케이션과 데이터베이스 사이에 존재하는 context에 entity들을 저장시키고 관리하는데, 이 persistence context의 생명주기를 이용해 캐시, 트랜잭션 등의 기능들을 구현시킵니다. 

---

### 2.3.3) Spring JPA 적용하기  

자 이제 이전에 작업하던 프로젝트에 JPA를 적용하는 예제를 진행해보도록 하겠습니다.  

---

#### dependency 추가

```c
// build.gradle
// dependency 추가
    compile'org.springframework.boot:spring-boot-starter-data-jpa'
    compile 'mysql:mysql-connector-java'
```
  
---
  
#### datasource 설정

JPA도 JDBC와 마찬가지로 datasource 설정을통해 데이터베이스와의 연결설정을 진행합니다. 

```c
// application.properties

spring.datasource.url=jdbc:mysql://127.0.0.1:3306/TAES_NEWS
spring.datasource.username=root
spring.datasource.password=*****
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

---

#### Entity 설정

위에서 ORM 설명을 보면서 떠오르셨던것이 있나요? 바로 이전 프로젝트까지 사용하던 VO가 비슷한 역할을 했었습니다. 데이터를 객체화시켜 전달하는 역할을 해주었는데 사실 데이터베이스 테이블과 정확히 매핑된 객체라기보다는 조금더 느슨한 객체였습니다. JPA에서는 VO를 설정했던것 처럼 Entity를 설정하여 최대한 정확히 매핑시켜 사용합니다.  
  
이전에 사용했던 NewsVo 클래스를 수정해서 Entity로 사용보도록 하겠습니다.   
  
|Collumn|Datatype|
|:--:|:--:|
|id|INT|
|title|VARCHAR(200)|
|contents|VARCHAR(2000)|
|date|DATETIME|
|author|VARCHAR(50)|
|type|VARCHAR(45)|
  
```java
// News.java

@Entity
@Table(name="T_NEWS")
public class News {

    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private int id;

    @Column(length = 200, nullable = false)
    private String title;
    @Column(length = 2000, nullable = false)
    private String contents;
    @Column(nullable = false)
    private Date date;
    @Column(length = 50, nullable = false)
    private String author;
    @Column(length = 45, nullable = false)
    private String type;


    public int getId(){
        return this.id;
    }
    public String getTitle(){
        return this.title;
    }
    public String getContents(){
        return this.contents;
    }
    public Date getDate(){
        return this.date;
    }
    public String getAuthor(){
        return this.author;
    }
    public String getType(){
        return this.type;
    }
}
```

위와 같이 어노테이션을 이용하여 테이블 칼럼들과 매핑하여 Entity를 만들어주게되면, JPA객체로써 바로 사용이 가능해집니다. 이때 칼럼 값들에대해 getter, setter를 모두 지정해 주어야하는데 롬복(Lombok)을 이용하여 더욱 간단히 처리해주도록 하겠습니다.  

```c
// build.gradle

    compile 'org.projectlombok:lombok:1.18.6'
```  
  
```java
// News.java

@Entity
@Table(name="T_NEWS")
@Data
public class News {

    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private int id;

    @Column(length = 200, nullable = false)
    private String title;
    @Column(length = 2000, nullable = false)
    private String contents;
    @Column(nullable = false)
    private Date date;
    @Column(length = 50, nullable = false)
    private String author;
    @Column(length = 45, nullable = false)
    private String type;

}
```  
  
먼저, 롬복 dependency를 추가후, 기존 News Entity에 @Data 어노테이션 추가만으로 getter, setter, toString 등의 기본적인 메소드들을 자동으로 추가해 구성이 가능합니다.  
   
#### DAO,Service 설정

JDBC를 사용할때는 JDBC, SqlSession Template을 사용하여 SQL문은 직접 실행시킨것과는 다르게 JPA는 이전단계에서 진행한 Entity 설정만 해주면 findById, findAll, save 등의 기본적인 메서드형 데이터처리가 가능하며 원하는 메서드 또한 추가가 가능합니다.  
  
```c
// NewsDao.java

public interface NewsDao extends JpaRepository<News, Integer> {

    List<News> findByType(String type);

    @Query("select n from News n where n.type = :type")
    List<News> findCategoryNews(String type);

}
```  

```c
@Service
public class MainNewsServiceImpl implements NewsService {

    @Autowired
    NewsDao newsDao;

    @Override
    public List<News> getNews() {
        List mainNews = new ArrayList();
        mainNews = newsDao.findAll();

        return mainNews;
    }
}
```

위와같이 JPA를 사용하여 JDBC, MyBatis를 사용하는것보다 좀더 간편하고 JAVA Object화시켜 데이터베이스에서 원하는 데이터들을 조회하는것을 확인 할 수 있습니다. 새로운 메서드를 작성하고 싶을때는 Sql을 작성하는것처럼 메소드 이름에 키워드를 조합하여 원하는 데이터 관리가 가능하며, SQL처럼 사용 가능한 JPQL 쿼리문을 사용하여 데이터관리도 가능합니다. 

---

### 마무리

이번챕터에서는 스프링 프로젝트에 JPA를 통해 데이터베이스 통신을 하기위한 예제를 실행해보며 JPA의 동작원리에대해 알아보았습니다. JPA를 사용하게되면 데이터를 객체화할수 있으며 메서드 호출만으로 쿼리가 수행되어 생산성이 높아짐과 동시에 유지보수에도 큰 장점이 있습니다. 하지만 직접 SQL을 실행시키는것과 비교하면 성능상 조금 떨어질수도 있으며 복잡한 데이터 처리과정에 있어서 메서드만으로 처리하는데에는 문제가 있을수도 있습니다.  
JPA를 적용하는데 어느정도 러닝커브가 있지만, 프로젝트에 따라서 큰 생산성을 기여할 수도 있기에, 관심을 가지고 알아두시면 좋을것 같습니다.

--- 

### 샘플 프로젝트 
{: .no_toc }

위 프로젝트는 다음 링크에서 확인하실수 있습니다.  
<https://github.com/taes-k/spring-example/tree/master/spring-mvc-start-jpa>


---
