---
layout: post
comments: true
title: Spring, master-slave dynamic routing datasource 사용하기
tags: [spring, mysql]
---

### DB Replication

서비스를 운영하면서 DB의 부하를 줄이기 위한 방법으로 DB Replication을 통해 쿼리의 대부분을 차지하는 read 작업을 Slave DB를 사용하게끔 하여 부하를 분산 시킬수 있습니다. Read의 부하가 큰 서비스의 경우 Slave를 여러개 만들어 사용하는것도 가능합니다.

---

### Spring multi-datasource

위와 같은 이유로써 Master/ Slave를 나누어진 DB를 Spring에서 사용하기 위해서는 가장 간단한 생각으로는, 두개를 별개의 Datasource로 등록하여 Create, Update, Delete 작업은 Master Datasource에서, Read작업은 Slave Datasource에서 작업하도록 환경을 구축 할 수 있습니다.  
  
아래는 multi-datsource를 설정한 예제입니다.  

```java
// MasterDataSourceConfig.java

@Configuration
@MapperScan("com.example.api.mapper.master")
public class MasterDataSourceConfig
{
    @Primary
    @Bean("masterDataSource")
    public DataSource masterDataSource() {
        DataSourceBuilder<?> dataSourceBuilder = DataSourceBuilder.create();
        dataSourceBuilder.driverClassName("mysql.cj.jdbc.driver");
        dataSourceBuilder.url("master-db-url");
        dataSourceBuilder.username("id");
        dataSourceBuilder.password("password");
        return dataSourceBuilder.build();
    }

    @Primary
    @Bean("masterSqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory(@Qualifier("masterDataSource") DataSource dataSource, ApplicationContext applicationContext)
    {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSource);
        factoryBean.setMapperLocations(applicationContext.getResources("classpath:mybatis/mapper/master/**/*.xml"));
        return factoryBean.getObject();
    }

    @Bean
    public PlatformTransactionManager transactionManager(@Qualifier("masterDataSource") DataSource dataSource) {
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
        transactionManager.setDataSource(dataSource);
        return transactionManager;
    }
```

```java
// SlaveDataSourceConfig.java

@Configuration
@MapperScan("com.example.api.mapper.slave")
public class SlaveDataSourceConfig
{
    @Bean("slaveDataSource")
    public DataSource masterDataSource() {
        DataSourceBuilder<?> dataSourceBuilder = DataSourceBuilder.create();
        dataSourceBuilder.driverClassName("mysql.cj.jdbc.driver");
        dataSourceBuilder.url("slave-db-url");
        dataSourceBuilder.username("id");
        dataSourceBuilder.password("password");
        return dataSourceBuilder.build();
    }

    @Primary
    @Bean("slaveSqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory(@Qualifier("slaveDataSource") DataSource dataSource, ApplicationContext applicationContext)
    {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSource);
        factoryBean.setMapperLocations(applicationContext.getResources("classpath:mybatis/mapper/slave/**/*.xml"));
        return factoryBean.getObject();
    }
```

하지만 위와같은 방법을 사용했을때 다음과같은 문제점이 있습니다.  

1. 트랜잭션의 분리 문제
2. 동일한 트랜잭션내에서 Insert(Master) -> Select(Slave) 작업시 정상 동작이 보장되지 않음  
3. CRUD 목적에 따른 소스의 분리 -> 파일 관리가 어려워지고, 사용상의 불편함 초래   
  
---

### Spring Routing-datasource

사실 '같은 데이터'를 가지고있는 master - slave DB를 위에서 소개한 방법과같이 단순히 SQL 목적에 따라서 나누어 사용하게되면 문제가 생길 가능성이 있고 Slave가 늘어남에 따라 관리하기가 거의 불가능해 지게 됩니다. 이번에는 똑같이 Datasource를 여러개 등록하지만, Routing-datasource + LazyLoading을 이용하여 Transaction의 read-only 여부에 따라 datasource를 분리하여 사용해보도록 하겠습니다.  

아래는 ReplicationRoutingDataSource 예제입니다.  

```java
// ReplicationRoutingDataSource.java

public class DynamicRoutingDataSource extends AbstractRoutingDataSource
{
    @Override
    protected Object determineCurrentLookupKey() {
        boolean isReadOnly = TransactionSynchronizationManager.isCurrentTransactionReadOnly();
        if(isReadOnly) {
            return "slave";
        } else {
            return "master";
        }
    }
}
```

위의 determineCurrentLookupKey 메서드는 DataSource 연결이 필요할때마다 DataSourceMap에서 어떤 DataSource를 사용할지 Key를 찾아주는 역할을 하는데 여기에 `TransactionSynchronizationManager.isCurrentTransactionReadOnly()`을 통해서 현재 진행중인 트랜잭션이 ReadOnly 인지 여부를 판단하여 Master / Slave DataSource를 분기해줄수 있습니다.  


```java
// DatasourceConfig.java


@Configuration
@MapperScan("com.example.api.mapper")
public class DataSourceConfig
{
    public DataSource createDataSource(DataSourceProperty.Base dataSourceProperty) {
        DataSourceBuilder<?> dataSourceBuilder = DataSourceBuilder.create();
        dataSourceBuilder.driverClassName(dataSourceProperty.getDriverClassName());
        dataSourceBuilder.url(dataSourceProperty.getUrl());
        dataSourceBuilder.username(dataSourceProperty.getUsername());
        dataSourceBuilder.password(dataSourceProperty.getPassword());

        log.info(String.format("DB Setting // username: %s", dataSourceProperty.getUsername()));
        return dataSourceBuilder.build();
    }

    @Bean
    public DataSource routingDataSource(
        DataSourceProperty.Master masterDataSourceProperty
        , DataSourceProperty.Slave slaveDataSourceProperty) {

        DataSource masterDataSource = createDataSource(masterDataSourceProperty);
        DataSource slaveDataSource = createDataSource(slaveDataSourceProperty);

        DynamicRoutingDataSource routingDataSource = new DynamicRoutingDataSource();

        Map<Object, Object> dataSourceMap = new HashMap<>();
        dataSourceMap.put("master", masterDataSource);
        dataSourceMap.put("slave", slaveDataSource);
        routingDataSource.setTargetDataSources(dataSourceMap);
        routingDataSource.setDefaultTargetDataSource(masterDataSource);

        return routingDataSource;
    }

    @DependsOn({"routingDataSource"})
    @Bean
    public DataSource dataSource(DataSource routingDataSource) {
        return new LazyConnectionDataSourceProxy(routingDataSource);
    }

    @Bean
    public SqlSessionFactory sqlSessionFactory(
        DataSource dataSource,
        ApplicationContext applicationContext)
        throws Exception
    {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSource);
        factoryBean.setMapperLocations(applicationContext.getResources("classpath:mybatis/mapper/**/*.xml"));

        return factoryBean.getObject();
    }

    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource) {
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
        transactionManager.setDataSource(dataSource);
        return transactionManager;
    }

}
```

위 DataSourceConfig 핵심사항은 `RoutingDataSource`로 선언되어있는 DataSource를 `LazyConnectionDataSourceProxy`에 등록시켜, lazy loading을 통해 연결할때마다 `ReplicationRoutingDataSource`를 통해 master/slave datasource를 분기시키는 것입니다. 
  
```java
// SampleService.java

@Service
public class SampleService
{
    private final SampleMapper SampleMapper;

    @Autowired
    public SampleService(com.wmp.wsin.cms.api.sample.mapper.SampleMapper sampleMapper)
    {
        SampleMapper = sampleMapper;
    }

    @Transactional
    public Sample.Param setSample(Sample.Param param)
    {
        SampleMapper.setSample(param);
        SampleMapper.setSampleDtl(param);
        return SampleMapper.getSample(param.getTestId());
    }

    @Transactional
    public Sample.Param updateSample(Sample.Param param, int seq)
    {
        SampleMapper.updateSample(param, seq);
        return SampleMapper.getSample(seq);
    }

    @Transactional(readOnly = true)
    public Sample.Param getSample(int seq)
    {
        return SampleMapper.getSample(seq);
    }
}
```

위와같이 Create/Update Transaction이 필요한곳에는 `readOnly = false (default)` 를 사용해주고, 단순 Read만 필요할때는 `readOnly = true`를 명시해주어 Slave Datasource를 사용 할 수 있습니다.

---

#### 주의사항

위 datasource를 설정하면서 주의해야 할 내용이 있습니다.  
RoutingDatasource를 LazyConnectionDataSourceProxy에 등록하게되는데, 이때 Bean을 초기화 하는 과정에서 등록 순서에 문제가 생기면 다음과같은 오류가 발생 할 수 있습니다.  

![1]({{ site.images | relative_url }}/posts/2020-03-11-sprinig-master-slave-dynamic-routing-datasource/1.png)  

>  nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'dataSource' defined in class path resource [com/wmp/wsin/cms/config/datasource/DataSourceConfig.class]: Unsatisfied dependency expressed through method 'dataSource' parameter 0; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'routingDataSource' defined in class path resource [com/wmp/wsin/cms/config/datasource/DataSourceConfig.class]: Initialization of bean failed; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'org.springframework.boot.autoconfigure.jdbc.DataSourceInitializerInvoker': Invocation of init method failed; nested exception is org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'dataSource': Requested bean is currently in creation: Is there an unresolvable circular reference?

dataSource bean을 등록하는과정에서 위 이미지와 같이 무한 loop를 돌면서 bean 등록에 실패하게되어 생기는 오류입니다. 해당 오류를 방지하기 위해서 Datasource를 설정하는 코드에서 `@DependsOn({"routingDataSource"})`를 선언해주어 의존 관계를 명시 해주는 이유 입니다.  
  
---

#### datasource 분기 테스트  

DynamicRoutingDatasource를 다양한 테스트케이스들을 통해서 직접 테스트한 결과를 공유합니다.  
  

***Test Case1***  

- @Transactional(readOnly=true) 설정시 insert/update/delete block 되는지 여부? or Exception 발생?  

> → Exception 발생 
```
### Error updating database.  Cause: java.sql.SQLException: Connection is read-only. Queries leading to data modification are not allowed\n### The error may exist in file [/Users/we/Documents/git_repo/wsin-cms/wsin-catalog-cms/build/resources/main/mybatis/mapper/sample/SampleMapper.xml]\n### The error may involve defaultParameterMap\n### The error occurred while setting parameters\n### SQL: INSERT INTO tx_test_table(test_cd, test_nm, test_dtl, reg_id)         VALUES (?, ?, ?, ?)\n### Cause: java.sql.SQLException: Connection is read-only. Queries leading to data modification are not allowed\n; Connection is read-only. Queries leading to data modification are not allowed; nested exception is java.sql.SQLException: Connection is read-only. Queries leading to data modification are not allowed
```

@Transactional(readOnly=true) → JDBC Connection.setReadOnly(true)를 호출합니다.  
이는 단지 힌트일 뿐이며 실제 DB를 read-only 트랜잭션으로 생성할지 여부는 JDBC 드라이버에 따라 달라지고 행위도 데이터 계층 프레임워크의 구현여부에 따라 달라질 수 있습니다.
MySQL의 경우 5.6.5 버전 부터 지원합니다.


***Test Case2***  

- @Transactional() 메소드에서 @Transactional(readOnly = true) 메소드를 호출.
- case 2-1 : 서로 다른 클래스에서 호출시 @Transactional(readOnly = true)  메소드가 Slave DB로 연결되는지 여부?  

> → X, master datasource로 모두 실행됨

- case 2-2 : 같은 클래스내에서 호출시 @Transactional(readOnly = true)  메소드가 Slave DB로 연결되는지 여부?  

> → X, master datasource로 모두 실행됨


***Test Case3***  

- @Transactional(readOnly = true) 메소드가 @Transactional() 메소드 호출.  
- case 3-1 : 서로 다른 클래스에서 호출시  @Transactional() 메소드가 Master DB로 연결되는지 여부?  
> → X, slave datasource로 모두 실행됨 → read-only 에러발생

- case 3-2 : 같은 클래스내에서 호출시  @Transactional() 메소드가 Master DB로 연결되는지 여부?  
> → X, slave datasource로 모두 실행됨 → read-only 에러발생  


***Test Case4***  

- @Transactional() 메소드에서 @Transactional(propagation = REQUIRED_NEW, readOnly = true) 메소드 호출  
- case 4-1 : 서로 다른 클래스에서 호출시  @Transactional(readOnly = true)  메소드가 Slave DB로 연결되는지 여부?
> → O, 자식 Transaction에서 에러 발생시 부모 Transaction 까지 에러 전달되어 Rollback됨.

- case 4-2 : 같은 클래스내에서 호출시  @Transactional(readOnly = true)  메소드가 Slave DB로 연결되는지 여부?
> → X, master datasource로 모두 실행됨


***Test Case5***  

- @Transactional(readOnly = true) 메소드에서 @Transactional(propagation = REQUIRED_NEW) 메소드 호출
- case 5-1 : 서로 다른 클래스에서 호출시 Datasource 분기 일어나는지 여부?  
> → O, read : slave datasource, Insert : master datasource

- case 5-2 : 같은 클래스내에서 호출시 Datasource 분기 일어나는지 여부?  
> → X, slave datasource로 모두 실행됨 → read-only 에러발생

***Test Case6***  

- @Transactional() Service메소드에서 @Transactional(propagation = REQUIRED_NEW, readOnly = true) Mapper 메소드 호출시 readOnly 메서드만 Slave DB로 연결되는지 여부?
> → O, read : slave datasource


**테스트 결과**  

원하는대로 Master/Slave를 분기하여 사용하기 위해서는 일반적인 선언형 Transaction인 @Transactional 사용에 주의를 해서 사용 하셔야합니다.  

- readOnly = true 사용시 Read를 제외한 Modify 관련 쿼리는 실행 되지 않음.
- AOP 방식의 Annotation이기 때문에 동일한 클래스 호출시 Transactional 분기 일어나지 않음.  
- Propagation 옵션 고려해서 사용 할 것