---
layout: post
comments: true
title: Spring DB 커넥션
tags: [spring]
---

### 구동환경
- Spring boot 2.1.3

### Spring DB Connection
Spring을 사용해서 Database와의 커넥션을위한 Session  DAO 커넥션을위한 설정을 알아보도록 하자.  
먼저, DB 커넥션을 위해 Session factory 설정이 필요하다. 이를위해 java Configuration 파일 `DataSourceConfig.java`를 생성해주도록 하자.  

```c
// DataSourceConfig.java

@Configuration
public class DatasourceConfig { 

    @Autowired
    ApplicationContext applicationContext;

    @Primary
    @Bean(name="dataSource")
    @ConfigurationProperties(prefix = "spring.datasource") 
    public DataSource dataSource() { 
        return DataSourceBuilder.create().build(); 
    } 

    @Primary
    @Bean(name = "sqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory() throws Exception { 
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean(); 
        sqlSessionFactoryBean.setDataSource(dataSource()); 
        sqlSessionFactoryBean.setMapperLocations(applicationContext.getResources("classpath:mybatis/mapper/*/*-sql.xml"));
        return sqlSessionFactoryBean.getObject(); 
    } 

    @Primary
    @Bean(name = "sqlSessionTemplate")
    public SqlSessionTemplate sqlSessionTemplate() throws Exception { 
        return new SqlSessionTemplate(sqlSessionFactory()); 
    } 
}

```

위 설정중 prefix인 'spring.datasource'의 설정값은 application.porperties에서 설정해두고 자동으로 DB 커넥션을 설정 해 줄수 있다.  

`application.properties`
```c
// application.properties

spring.datasource.url= DB 주소
spring.datasource.username= ID
spring.datasource.password= PW
```

DAO에서는 Autowired를 통해 위에서 생성해둔 SessionTemplate을 주입해줌으로써 간단하게 DBConnection을 이용 할 수 있다.
```c
// test.dao
public class TestDao{

    @Autowired
    @Qualifier("sqlSessionTemplate")
    private SqlSessionTemplate sqlSession;

    public int getTest( Map<String, Object> _param ){
        return sqlSession.selectOne( "test.getTest", _param );
    }
}
```

Session factory 설정시 설정해둔 매퍼 위치에 sql mapping을 통해  DB Sql을 작성해 결과를 받아온다.
```c
<mapper namespace="test">
    <select id="getTest" parameterType="java.util.concurrent.ConcurrentHashMap" statementType="CALLABLE" > 
        SELECT  *
        FROM    TEST
        WHERE   TEST_Idx = 1
    </select>
</mapper>
```

위와같이 DAO와 분리된 DB Connection 코드를 통해 DB 설정을 독립적으로 관리 할 수 있으며 DAO의 확장에서도 문제없이 사용 할 수 있다.
