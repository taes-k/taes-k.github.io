---
layout: post
comments: true
title: spring boot에서 다중 datasource 환경에서의 트랜잭션 
tags: [spring]
---

## datasource와 transaction
트랜잭션은 일반적으로 단일 datasource단위로 관리됩니다.  
그렇다면 다중 datasource 환경에서는 transaction을 어떻게 구축할수 있는지 알아보도로고 하겠습니다.

## Transaction propagation 설정을 통한 환경
Datasource 마다 TransactionManager가 설정되어, 양측의 데이터소스 트랜잭션을 확실하게 체크 할 수 없습니다.  
Propagation을통해 한쪽의 Transaction을 따라가 줄 수 있으나, 양방향으로는 불가능합니다. (한쪽에서  Select만 하는 상황이라면 써볼 수 있을듯)

참조문서 ) https://supawer0728.github.io/2018/03/22/spring-multi-transaction/


## ChainedTransactionManager
PlatformTransactionManager 구현체로 transaction 생성, commit, rollback을 위임자 패턴으로 구성합니다. 이 구현체를 사용하는 것의 전제되는 가정은, transaction의 rollback을 야기하는 오류는 대부분 transaction이 완료되기 전, 혹은 가장 안쪽의 PlatformTransactionManager에서 발생한다는 것입니다.  
  
이 구현체의 인스턴스는 지정된 순서대로 transaction을 시작하고, 역순으로 commit/rollback 하는 특성을 가집니다. 즉, transaction을 중단시킬 가능성이 가장 큰 PlatformTransactionManager가 마지막에 설정되어야 합니다.  
commit 중에 예외를 던지는 PlatformTransactionManager는 자동으로 다른 transaction manager의 rollback을 일으킨다.

참조문서 ) https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/transaction/ChainedTransactionManager.html


### 요약
- 기본적으로 for문을통해 datasource들의 TransactionManasger들을 PlatformTransactionManager에 등록시켜 사용합니다.
- 각각의 트랜잭션 매니저들을 연쇄 체인으로 묶어주어 상황에 따라 트랜잭션이 필요한 데이터 트랜잭션들만을 묶어서 사용 할 수 있습니다. 
- 이 과정에서 Datasource 커넥션풀을 많이 차지할 수 있게되는데, 커넥션이 일어나는 시점에서 사용하도록 LazyConnectionDataSourceProxy로써 등록 해준다.

### 구현
1. Dependency  

```
// build.gradle

dependencies {
    compile('org.springframework.data:spring-data-commons')
}
```

2. Datasource  

```
// DatasourceFirst.java

Public class DataSourceFirst {
    @Bean
    public DataSource dataSourceFirst(DataSourceProperties properties)
    {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl(properties.getUrl());
        dataSource.setUsername(properties.getUsername());
        dataSource.setPassword(properties.getPassword());
        dataSource.setDriverClassName(properties.getDriverClassName());
 
        return new LazyConnectionDataSourceProxy(dataSource);
    }
 
    @Bean(name="firstTxManager")
    @Primary
    public PlatformTransactionManager firstTxManager(@Autowired @Qualifier("dataSourceFirst") DataSource dataSource)
    {
        return new DataSourceTransactionManager(dataSource);
    }
}


// DatasourceSecond.java

Public class DataSourceSecond {
    @Bean
    public DataSource dataSourceSecond(DataSourceProperties properties)
    {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl(properties.getUrl());
        dataSource.setUsername(properties.getUsername());
        dataSource.setPassword(properties.getPassword());
        dataSource.setDriverClassName(properties.getDriverClassName());
 
        return new LazyConnectionDataSourceProxy(dataSource);
    }
 
    @Bean(name="secondTxManager")
    @Primary
    public PlatformTransactionManager secondTxManager(@Autowired @Qualifier("dataSourceSecond") DataSource dataSource)
    {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

3. ChainedTransactionManager  

```
// DatasourceSecond.java

public class ChainedTxConfig
{
    @Bean
    @Primary
    @Autowired
    public PlatformTransactionManager transactionManager(@Qualifier("firstTxManager") PlatformTransactionManager firstTxManager, @Qualifier("secondTxManager") PlatformTransactionManager secondTxManager)
    {
        return new ChainedTransactionManager(cmsTxManager, enuriTxManager);
    }
}
```

## JtaTransactionManager

JTA (JAVA Transaction API)  
JTA(Java Transaction Api)는 자바 표준으로써, 분산 transaction을 가능하게 해준다. 매우 간단하게 설명하자면, JTA를 지원하는 자원을 가리키는 XA Resource 인터페이스의 구현체들을 등록하면, 해당 구현체들에 대해서 전역 transaction을 지원해줍니다.  
Java EE Application Server에서는 전역 tranction을 지원하기 위해서, JTA를 사용하기도 합니다.   
Spring에서는 JNDI에서 Java EE Container가 사용 중인 DataSource를 가져와, JtaTransactionManager에 설정할 수도 있습니다.

### 구현
1. Dependency  

```
// build.gradle

dependencies {
    compile('org.springframework.data:spring-boot-starter-jta-atomikos')
}
```

2. Datasource  

```
// Datasource.java

@Bean(name = "dataSourceFirst")
public DataSource cmsDataSource()
{
    java.util.Properties properties = new Properties();
    properties.setProperty("user", "root");
    properties.setProperty("password", "aaaaa");
    properties.setProperty("url", "jdbc:mysql://localhost:11111/search_cms_dev");
 
    AtomikosDataSourceBean dataSource = new AtomikosDataSourceBean();
    dataSource.setXaDataSourceClassName("com.mysql.cj.jdbc.MysqlXADataSource");
    dataSource.setXaProperties(properties);
 
    return dataSource;
}
``` 
  
참고)  https://d2.naver.com/helloworld/5812258   


## ChainedTransactionManager VS JtaTransactionManager  
두가지 종류의 TransactionManager모두 PlatformTransactionManager를 기본으로 사용합니다.   
ChainedTransactionManager의 경우 기존 Datasource마다 가지고있는 TransactionManager를 체인형태로 등록, 연결하여 트랜잭션을 순차적으로 실행시켜준다는 특징이 있으며 이러한 특징으로 인해 하나의 트랜잭션 커밋 완료후, 다음 트랜잭션 커밋과정에서 오류가 발생할 경우 이전에 커밋된 내용에 대해서는 롤백을 보장 할 수 없습니다. 이러한 상황을 줄여주기 위해서는 실행순서를 고려한 Chain 등록이 필요합니다.   
그와는 다르게 JTATransactionManager는 하나의 트랜잭션 매니저로 묶어서 관리하여 한번의 커밋/롤백을 진행하여 완전한 트랜잭션을 보장할수있습니다. 다만 실행속도가 비교적 느릴수 있다는 단점이 존재합니다.