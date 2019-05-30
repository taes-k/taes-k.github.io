---
layout: default
comments: true
title: (요령과 기본) 2.2 Spring 프로젝트 시작하기 (2) - JDBC
parent: 요령과 기본
date: 2019.05.28
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### 2.2 Spring 프로젝트 시작하기 (2) - JDBC
{: .no_toc }
이번챕터에서는 지난 프로젝트에서 서비스단에서 넣어주었던 데이터들을 데이터베이스로 옮기고 JDBC를 통해 데이터베이스와 연동하는 예제를 진행해보도록 하겠습니다.   
  

## Contents List
{: .no_toc .text-delta }

1. TOC
{:toc}

---

### 데이터베이스
  
이번 예제에 사용할 데이터베이스는 MySql입니다. 데이터베이스 및 SQL에 대해서는 잘 아실거라 믿고 따로 설명을하지는 않겠습니다. 
가장 먼저 진행할사항은 이전프로젝트에서 사용했던 '뉴스'데이터를 데이터베이스로 옮기는작업을 진행하도록 하겠습니다.   
  
***1\. 테이블 만들기***  
  
```java

News news1 = new News();
news1.setTitle("중고차 매매 후 한 달 내 고장나면 보험 보상받는다");
news1.setContents("중고차 구입 소비자의 피해를 구제하기 위한 중고차 보험이 다음 달부터 의무화되면서 구입 한 달 이내의 중고차 고장은 보험으로 보장을 받을 수 있게 된다.");
news1.setDate("2019-05-27 13:30");
news1.setAuthor("Taes");
news1.setType("financial");

News news2 = new News();
news2.setTitle("하늘 참 신기하네");
news2.setContents("오전 대구 달서구청 옥상 정원에서 시민들이 하늘에 펼쳐진 이색적인 구름을 관찰하고 있다. 파란 하늘에서 잘라낸 듯 넓게 펼쳐진 구름대와 차고 습한 대기 속을 비행하는 항공기가 남기는 가늘고 긴 구름 ‘비행운(오른쪽 가늘고 긴 구름)’이 겹쳐지며 진기한 풍경을 연출하고 있다. ");
news2.setDate("2019-05-27 09:30");
news2.setAuthor("Taes");
news2.setType("social");

News news3 = new News();
news3.setTitle("커쇼 11연승, 다저스 4연승");
news3.setContents("커쇼가 11연승에 성공했다. 벨린저는 19호 홈런과 함께 팀을 구해낸 두 개의 외야 어시스트를 기록했다. 오클랜드는 10연승을 질주. ");
news3.setDate("2019-05-27 11:35");
news3.setAuthor("Taes");
news3.setType("sports");
```  
  
위와같은 샘플 데이터를 저장하기 위해 [제목, 내용, 날짜, 저자, 타입]을 저장하기위한 데이터베이스 테이블을 생성하도록 하겠습니다.  

```sql
CREATE TABLE `TAES_NEWS`.`T_NEWS` (
    `id` INT NOT NULL AUTO_INCREMENT,
    `title` VARCHAR(200) NOT NULL,
    `contents` VARCHAR(2000) NOT NULL,
    `date` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `author` VARCHAR(50) NOT NULL,
    `type` VARCHAR(45) NOT NULL,
PRIMARY KEY (`news_id`)
);

```  

위의 CREATE 쿼리로 다음과 같은 테이블이 만들어졌습니다.   

|Collumn|Datatype|
|:--:|:--:|
|id|INT|
|title|VARCHAR(200)|
|contents|VARCHAR(2000)|
|date|DATETIME|
|author|VARCHAR(50)|
|type|VARCHAR(45)|
  
***2\.샘플데이터 삽입***

위에서 생성한 테이블에 이제 샘플데이터를 넣어보도록 하겠습니다.  

```sql
INSERT INTO `TAES_NEWS`.`T_NEWS` (`title`, `contents`, `date`, `author`, `type`) VALUES ('중고차 매매 후 한 달 내 고장나면 보험 보상받는다', '중고차 구입 소비자의 피해를 구제하기 위한 중고차 보험이 다음 달부터 의무화되면서 구입 한 달 이내의 중고차 고장은 보험으로 보장을 받을 수 있게 된다.', '2019-05-27 13:30', 'Taes', 'financial'),
('하늘 참 신기하네', '오전 대구 달서구청 옥상 정원에서 시민들이 하늘에 펼쳐진 이색적인 구름을 관찰하고 있다. 파란 하늘에서 잘라낸 듯 넓게 펼쳐진 구름대와 차고 습한 대기 속을 비행하는 항공기가 남기는 가늘고 긴 구름 ‘비행운(오른쪽 가늘고 긴 구름)’이 겹쳐지며 진기한 풍경을 연출하고 있다.', '2019-05-27 09:30', 'Taes', 'social'),
('커쇼 11연승, 다저스 4연승', '커쇼가 11연승에 성공했다. 벨린저는 19호 홈런과 함께 팀을 구해낸 두 개의 외야 어시스트를 기록했다. 오클랜드는 10연승을 질주.', '2019-05-27 11:35', 'Taes');
```  

---

### Spring JDBC

스프링에서는 데이터베이스와 연결을 위해서 별도의 API가 필요합니다. 그중 먼저, JDBC를 사용해서 연결을 해보도록 하겠습니다. JDBC(Java Database Connectivity)란 자바에서 데이터베이스에 접속할수 있도록 만든 자바 API로써 데이터베이스의 종류에따라 별도의 JDBC 드라이버로 연결하여 사용 가능합니다.  
  
프로젝트에서 직접 설정을 진행하면서 JDBC의 구조를 알아보도록 하겠습니다.  
***1. dependency 추가하기***

jdbc와 mysql driver를 사용하기위해 gradle에 dependecy를 추가합니다.  
```c
// build.gradle
// dependency 추가

    compile 'org.springframework.boot:spring-boot-starter-jdbc'
    compile 'mysql:mysql-connector-java'
```
  
***2. datasource 설정***

스프링에는 JDBC 설정을 편리하게 할 수있도록 datasource라는 연결 팩토리를 가지고 있어, DB 연결에 필요한 몇몇 인자들만 등록해주면 간단하게 DB와의 연결을 획득 할 수 있습니다.  
  
Spring boot에서는 `application.properies`에서 설정을 해주면 가장 간단하게 사용이 가능합니다.  

```c
// application.properties

spring.datasource.url=jdbc:mysql://127.0.0.1:3306/TAES_NEWS
spring.datasource.username=root
spring.datasource.password=*****
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

***3. DAO, service 설정***

지난 프로젝트에서는 작성하지 않았던 DAO를 작성해주도록 하겠습니다. DAO(Data Access Object)는 실제로 DB에 접근하는 객체로써 Service 레이어와 데이터베이스를 연결해주는 역할을 해주는 클래스입니다.  
  
사전에 넣어둔 샘플 뉴스 데이터를 불러오기위해 DAO를 작성해보도록 하겠습니다. 이때 DAO는 Service를 작성한것과 마찬가지로 인터페이스와 구현체를 따로 나누어 작업하도록 하고, 데이터를 직접 넣어주었던 서비스 부분도 함께 수정하도록 하겠습니다.
```java
// NewsDao.java

public interface NewsDao {

    public List<News> getNews();

}
```  
  
```java
// MainNewsDaoImpl.java

@Repository
public class MainNewsDaoImpl implements NewsDao {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Override
    public List<News> getNews() {
        List mainNews = new ArrayList();
        String query = "select * from T_NEWS";
        return jdbcTemplate.query(query, new BeanPropertyRowMapper<News>(News.class));
    }
}
```   
  
```java
// MainNewsServiceImpl.java

@Service
public class MainNewsServiceImpl implements NewsService {

    @Autowired
    MainNewsDaoImpl mainNewsDao;

    @Override
    public List<News> getNews() {
        List mainNews = new ArrayList();
        mainNews = mainNewsDao.getNews();
        return mainNews;
    }
}

```  

dao에서는 이미 bean으로 등록되어있는 jdbcTemplate을 autowired로 주입 해와서 query문을 직접 입력하여 원하는 데이터 조작을 할 수 있습니다. 이때 result값은 이전에 미리 설정해둔 NewsVO 객체에 바로 주입시켜 결과로 받아 오도록 설정 할 수 있습니다.  

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_start_2_jdbc/jdbc_result.png" style="height:300px; border:1px solid #d0d0d0;">
</div>   

데이터베이스를 사용하기 전의 결과와 동일한 결과를 얻어내어, JDBC가 잘 동작 된 것을 확인 할 수 있습니다.  

---

### JDBC 동작 구조

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_start_2_jdbc/jdbc_structure.png" style="height:250px; border:1px solid #d0d0d0;">
</div>   
  
JDBC는 Interface와 드라이버로 구성되어 있습니다. 위의 구조도를 보면 JDBC드라이버는 DBMS의 종류에따라 구분되어지며 JDBC Interface는 어떤 DBMS 와의 연결에서도 공통된 API 기능을 제공해줌을 알 수 있습니다.  
  
---

### MyBatis

위와같이 JDBC만 사용하더라도 원하는 대부분의 결과를 얻으실수 있을실겁니다. 하지만 DAO JAVA파일에 직접적으로 SQL문을 작성하고 파라미터를 하나씩 직접 매핑시켜주는 등  편리성이 떨어지는데, MyBatis는 이를 편하게 해주기 위해 사용하는 라이브러리 입니다. 위 프로젝트에 MyBatis까지 연동해 보도록 하겠습니다.  
  
***1. Dependency 추가***

역시나 가장먼저 할일은 dependency를 추가하는 일 입니다.  
  
```c
//build.gradle
// dependency 추가

    compile 'org.mybatis.spring.boot:mybatis-spring-boot-starter:1.3.2'
```

***2. Mapper 등록***

MyBatis에서는 Mapper를 따로두어 SQL문을 xml로 따로 관리 할 수 있습니다.  
  
```c
//application.properties

mybatis.mapper-locations=/mybatis/mapper/*-sql.xml
```

```xml
//News-sql.xml

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="news">

    <select id="getMainNews" parameterType="java.util.Map" resultType="java.util.Map" statementType="CALLABLE" >
    SELECT     *
    FROM    T_NEWS
    </select>

</mapper>
```  
  
mapper의 위치 등록을 위해 `application.properties`에 mapper 파일들을 등록해주고, mapper들을 위한 디렉토리(/src/main/resources/mybatis/mapper/)를 만들어주어 그안에 mapper들을 등록해줍니다.  
    
***3. SQL SessionFactory 설정***

이제 위에서 작성한 매퍼들을 DAO와 연결시켜 사용해 주어야합니다. MyBatis에서는 query 스트링으로 질의하는 JDBC Template 대신 객체로써 질의를 할 수 있도록 SqlSessionTemplate을 사용하는데, 이에대한 설정은 위에서 Datasource설정 기반으로 자동으로 설정이 됩니다.  
  
```java
@Repository
public class MainNewsDaoImpl implements NewsDao {

    @Autowired
    private SqlSessionTemplate sqlSession;

    @Override
    public List<News> getNews() {
        List mainNews = new ArrayList();

        return sqlSession.selectList("news.getMainNews");
    }
}
```  
  
MyBatis는 SqlSession 객체를 만들어 질의를 하게되는데,  원하는 데이터 질의문에 따라 `selectList`,`update`,`selectOne` 등의 메소드들을 적절히 활용하고, mapper들과 잘 매칭시켜 사용하면 굉장히 효율적으로 프로젝트를 관리 할 수 있습니다. 

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_start_2_jdbc/mybatis_result.png" style="height:350px; border:1px solid #d0d0d0;">
</div>   

mybatis 까지 적용을 해서, 동일한 결과를 얻어내어, MyBatis까지 잘 적용된 것을 확인 할 수 있습니다.  

---
### Mybatis 동작 구조  
  
  
  <div style="text-align:center;">
  <img src="https://taes-k.github.io/assets/images/trick_basic/spring_start_2_jdbc/mybatis_structure.png" style="height:250px; border:1px solid #d0d0d0;">
  </div>   

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_start_2_jdbc/mybatis_process.png" style="height:250px; border:1px solid #d0d0d0;">
</div>   

Mybatis는 JDBC 이전 단계에서 실행됩니다. 클라이언트의 요청이 있을때 SqlSessionFactory에서 SqlSession을 만들어 매퍼를 찾아 SQL을 실행하는 구조로 동작을 하게 됩니다. SQL 쿼리를 별도로 구성하여 필요할때마다 참조하여 사용하여 JDBC 에 비해 코드 및 매개변수의 중복 작업들을 제거 할 수 있습니다.  

---

### <마무리>

스프링 프로젝트에 JDBC를 통해 데이터베이스를 연결하여 데이터를 직접 불러오는 예제와 함께 JDBC 및 Mybatis의 구조를 알아보았습니다. 사용하는 것은 간단한 datasource 설정만으로 연결 후 쉽게 사용이 가능하지만, 그 동작 구조까지 잘 알고계신다면 좋을것 같습니다. 

--- 

### 샘플 프로젝트 
{: .no_toc }

위 프로젝트는 다음 링크에서 확인하실수 있습니다.  
<https://github.com/taes-k/spring-example/tree/master/spring-mvc-start-jdbc>


---
