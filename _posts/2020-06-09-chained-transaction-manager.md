---
layout: post
comments: true
title: ChainedTransactionManager 데이터소스 트랜잭션 연결하기
tags: [spring, transaction]
---

### 다중 데이터소스 트랜잭션 연결
요새는 MSA로 시스템을 구축하여 서버와 DB도 모두 각각 분리하여 마이크로하게 시스템 설계를 하는 추세이지만, 서비스에 따라 여러가지 이유로인해 여러개의 Database를 연결하고 하나의 트랜잭션으로 묶어야 할 상황이 생길 수 있습니다.  

이번 포스팅에서는 Spring 에서 다중 데이터소스를 가장 쉽게 트랜잭션으로 묶어주는 Chained Transaction Manager를 소개해보려고합니다.

---

### ChainedTransactionManager  
`ChainedTransactionManager`는 org.springframework.data(Spring Data Commons)에서 공식으로 지원하는 기술입니다.  

이 기술을 아주 간단하게 설명하자면, 말 그대로 여러개의 트랜잭션 매니저를 하나로 묶어(Chain)사용하는 방식인데 트랜잭션의 시작과 끝에서 연결된 트랜잭션들을 순차로 Start/Commit 시킴으로써 하나의 트랜잭션으로 실행되는것 처럼 동작하게 됩니다. 아래 그림을 보시면 단번에 어떤 구조인지 이해가 되실거라 생각합니다.

![1]({{ site.images | relative_url }}/posts/2020-06-09-chained-transaction-manager/1.png) 

먼저, `ChaineedTransactionManager`는 '완벽한' 트랜잭션을 제공하지 않습니다. 위 그림에서 눈치 채신분도 있겠지만, 이 트랜잭션을 구현시키는 구조적인 이유로인해 완벽한 트랜잭션의 롤백을 보장 할 수는 없습니다. 이해를 돕기위해 아래에서 예시를 들어보겠습니다.  

```
- 트랜잭션1 (Tx1)
- 트랜잭션2 (Tx2)

Tx1 (Start) -> Tx2 (Start) -> Logic -> Tx2 (COMMIT) -> Tx1 (COMMIT) 


Tx1 (Start) -> Tx2 (Start) -> Logic -> Tx2 (ROLLBACK) -> Tx1 (ROLLBACK) 
```
위 프로세스처럼 동작한다면, 아주 이상적으로 동작을 한 트랜잭션입니다. 하지만, 아래와같은 프로세스가 발생 할 수도 있습니다.

```
Tx1 (Start) -> Tx2 (Start) -> Logic -> Tx2 (COMMIT) -> Tx1 (ROLLBACK) 
```

`Tx2`이 COMMIT 된 이후에 `Tx1`가 ROLLBACK이 된다면 `Tx2`은 이미 COMMIT이 되었기에 ROLLBACK 되지 않아 데이터에 문제가 발생 할 것입니다.

이러한 이유로 인해서 `ChaiendTransactionManager`를 구성할때에는 체이닝 순서가 중요합니다. 예제에서 보시다시피 더 나중에 선언된 트랜잭션에서 에러가 날 경우에는 트랜잭션 ROLLBACK이 보장되기에, 에러가 날 확률이 높은 트랜잭션을 후순위 chain으로 묶어주셔야 조금 더 안전한 트랜잭션을 구성 할 수 있습니다.


약간 헷갈리실수도 있는데 한번더 확인하셔야 할 점은 로직이 수행되는 시점이 아닌, 트랜잭션이 `종료`되는 시점에서에서의 에러임을 확인해주셔야합니다. 

![2]({{ site.images | relative_url }}/posts/2020-06-09-chained-transaction-manager/2.png)  

SQL으로 따지자면 트랜잭션이 위와 같이 동작하는것이아닌, 아래와같이 동작한다는점을 기억해주셔야 합니다.

![3]({{ site.images | relative_url }}/posts/2020-06-09-chained-transaction-manager/3.png)  

쿼리는 위와 같이 실행되고 Commit 되기에, 트랜잭션 로직상에서의 데이터소스 에러는 모든 롤백이 보장됩니다. 위에서 말씀드린 문제는, 트랜잭션이 끝나는 시점에서의 COMMIT시 에러시에 문제가 생길 수 있다는점을 말씀드리는 내용입니다. 

---

### ChainedTransactionManager 구현

처음에도 말씀드렸다시피 `ChainedTransactionManager`의 구현은 매우 쉽습니다. 우선 다중 Datasource를 먼저 구현되어 있다고 가정해보겠습니다.  

```java
// tx1

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
 
    @Primary
    @Bean(name="firstTxManager")
    public PlatformTransactionManager firstTxManager(@Autowired @Qualifier("dataSourceFirst") DataSource dataSource)
    {
        return new DataSourceTransactionManager(dataSource);
    }
}
```


```java
// tx2
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
    public PlatformTransactionManager secondTxManager(@Autowired @Qualifier("dataSourceSecond") DataSource dataSource)
    {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

위와 같이 두개의 Datsource와 TransactionManager가 선언되어 있다고할때, 아래와같이 `ChainedTransactionManager`를 등록 할 수 있습니다.

```java
// ChainedTransactionManager

public class ChainedTxConfig
{
    @Bean
    @Primary
    @Autowired
    public PlatformTransactionManager chainedTransactionManager(   
        @Qualifier("firstTxManager") PlatformTransactionManager firstTxManager
        , @Qualifier("secondTxManager") PlatformTransactionManager secondTxManager)
    {
        return new ChainedTransactionManager(firstTxManager, secondTxManager);
    }
}
```

여기서 중요한 점은 위에서 말씀드렸던 `등록순서` 입니다. 에러의 영향도가 큰 트랜잭션일수록 체인의 뒤쪽에 연결시켜야 보다 안전한 트랜잭션을 구성 할 수 있습니다. 
이렇게 선언한 트랜잭션 매니저는 서비스에서 다음과 같이 사용 될 수 있습니다.

```java
// service

@Transactional("chainedTransactionManager")
public void doTxExample()
{
    ...
}

```



