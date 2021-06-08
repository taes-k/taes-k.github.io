---
layout: post
comments: true
title: 몰라도 되는 Spring - Spring batch chunk item reader
tags: [spring, batch, chunk, itemreader]
---

### Item Reader

지난 포스팅에서 잠깐 나왔듯이 `청크지향 프로세싱`을 하게 된다면 `Iteam Reader`, `Item Processor`, `Item Writer` 구성이 가능합니다.  

이 중 에서도 이번 포스팅에서 다룰 `Item Reader`는 프로젝트에서 사용하는 다양한 `Persistance framework`를 지원하기 위해 다양한 구현체들을 제공하고 있습니다.

![1]({{ site.images | relative_url }}/posts/2021-06-16-spring-batch-chunk-reader/1.png) 

![2]({{ site.images | relative_url }}/posts/2021-06-16-spring-batch-chunk-reader/2.png) 

위 `Abstract class`이미지를 보면 `ItemReader`는 `Cursor`와 `Paging` 두가지 방식으로 제공되고 있음을 알 수 있습니다.

---

### Curosr ItemReader

먼저 `Cursor`를 활용한 `ItemReader`방식에 대해 알아보도록 하겠습니다.  
`Cursor`를 이용한 방식은 RDB에서 일반적으로 `Stream`을 받을수 있는 솔루션이기 때문에 배치 개발자들에게 일반적으로 많이 사용되는 방법입니다.  

적절한 `fetchsize`를 지정하여 배치 성능을 향상 시킬 수 있습니다.

```java
@Bean
public JdbcCursorItemReader<CustomerCredit> itemReader() {
	return new JdbcCursorItemReaderBuilder<CustomerCredit>()
			.dataSource(this.dataSource)
			.name("creditReader")
			.sql("select ID, NAME, CREDIT from CUSTOMER")
			.fetchSize(1000)
			.rowMapper(new CustomerCreditRowMapper())
			.build();

}
```

`Cursor ItemReader`는 `Cursor`를 사용하는 특성상 다음과 같은 특징을 가집니다.

- 하나의 커넥션에서 `Cursor`를 통해 관리
- `Result Set`을 통한 `Cursor` 관리

위같은 특징들로 인해 아래와 같은 장/단점이 있습니다.

- 일반적으로 빈번한 조회가 일어나는 배치프로세스의 특성상 `Cursor`를 이용해 빠른 조회성능을 기대 할 수 있습니다.
- `Cursor`를 관리하는 `ResultSet`이 쓰레드세이프 하지 않기때문에 멀티쓰레드 환경에서 사용이 불가능합니다.
- 대용량 처리를 하는 배치프로세스의 특성상 하나의 커넥션을 오래동안 열고 있게되면 `timeout`이 발생 할 수 있습니다. 일반적인 api 설정보다도 긴 `connection timeout`, `network timeout`등의 설정이 필요합니다.

---

### Paging ItemReader

다음은 `Paging` 쿼리를 이용한 `ItemReader` 입니다.  
DB에 요청하는 쿼리로써 페이징을 하기 때문에 위에서 알아본 `Cursor`방식의 단점을 피하기 위해 주로 사용되어집니다. 

```java
@Bean
public JdbcPagingItemReader itemReader(DataSource dataSource, PagingQueryProvider queryProvider) {
	Map<String, Object> parameterValues = new HashMap<>();
	parameterValues.put("status", "NEW");

	return new JdbcPagingItemReaderBuilder<CustomerCredit>()
           				.name("creditReader")
           				.dataSource(dataSource)
           				.queryProvider(queryProvider)
           				.parameterValues(parameterValues)
           				.rowMapper(customerCreditMapper())
           				.pageSize(1000)
           				.build();
}

@Bean
public SqlPagingQueryProviderFactoryBean queryProvider() {
	SqlPagingQueryProviderFactoryBean provider = new SqlPagingQueryProviderFactoryBean();

	provider.setSelectClause("select id, name, credit");
	provider.setFromClause("from customer");
	provider.setWhereClause("where status=:status");
	provider.setSortKey("id");

	return provider;
}
```


`Paging ItemReader`는 `Paging` 쿼리를 사용하는 특성상 다음과 같은 특징을 가집니다.

- 호출 할때마다 매번 다른 `Connection` 연결
- `Limit`, `Offset` 쿼리를 통한 페이징처리

위같은 특징들로 인해 아래와 같은 장/단점이 있습니다.

- `Cursor`에 비해 조회성능이 느릴 수 있습니다.
- 멀티쓰레드 환경에서 병렬처리가 가능합니다.
- `Limit`을 통해 페이징 사이즈를 조절 하기 때문에 `Order`지정이 반드시 필요합니다.
- `Limit`, `Offset`을 사용하는 특성상 대용량처리를 수행할 경우 작업 후반부로 갈수록 성능이 점점 떨어 질 수 있습니다.

---

### Step partitioning

배치 프로세스를 수행하면서 `N천만건`의 대용량 데이터를 처리해야한다고 가정했을때, 어떤 `ItemReader`를 사용하는것이 좋을까요? 

위에서 알아본 내용으로 고민해보자면 `Cursor`방식을 사용하면 `Connection time`이 길어질 수 있고 병렬처리가 불가능하다는점이 단점이 될 수 있습니다. 반면에 `Paging`방식을 사용하면 병렬처리를 할 수 있지만 `LIMIT` 쿼리의 특성상 후반부로 갈수록 성능이 점점 나빠지게 될 것 입니다. 

```sql
-- OFFSET 후반부 예제 쿼리

SELECT *
FROM sample_log
ORDER BY log_seq
LIMIT 1000 OFFSET 20005000
```

`Paging`방식으로 처리시, 위와같이 `Offset`이 굉장히 커지게되면서 단순 조회 쿼리가 매우 느려지는 현상이 발생하게 될 수 있습니다.  

이와같은 현상을 회피하기위한 방법으로 `Step Partitioning`을 사용 할 수 있습니다. 사용법은 아래와 같습니다.  

```java
// SamplePartitioner.java


public class SamplePartitioner implements Partitioner {

    private final JdbcOperations jdbcTemplate;
    private final String table;
    private final String column;

    public SamplePartitioner(JdbcOperations jdbcTemplate, String table, String column) {
        this.jdbcTemplate = jdbcTemplate;
        this.table = table;
        this.column = column;
    }

    @Override
    public Map<String, ExecutionContext> partition(int gridSize) {
        int min = jdbcTemplate.queryForObject("SELECT MIN(" + column + ") from " + table, Integer.class);
        int max = jdbcTemplate.queryForObject("SELECT MAX(" + column + ") from " + table, Integer.class);
        int targetSize = (max - min) / gridSize + 1;

        Map<String, ExecutionContext> result = new HashMap<>();
        int number = 0;
        int start = min;
        int end = start + targetSize - 1;

        while (start <= max) {
            ExecutionContext value = new ExecutionContext();
            result.put("partition" + number, value);

            if (end >= max) {
                end = max;
            }
            value.putInt("minValue", start);
            value.putInt("maxValue", end);
            start += targetSize;
            end += targetSize;
            number++;
        }

        return result;
    }
}
```

위 코드는 Spring Batch Sample 코드에서 확인 가능합니다. [ColumnRangePartitioner.java](https://github.com/spring-projects/spring-batch/blob/d8fc58338d3b059b67b5f777adc132d2564d7402/spring-batch-samples/src/main/java/org/springframework/batch/sample/common/ColumnRangePartitioner.java)

```java
// SampleStepConfig.java

@Bean(name = "sampleStep")
public Step sampleStep() {
	return stepBuilderFactory.get("sampleStep")
			.partitioner("sampleSteps", partitioner()
			.step(step1())
			.partitionHandler(partitionHandler())
			.build();
}

@Bean(name = "samplePartitioner")
@StepScope
public ProductIdRangePartitioner partitioner() {
	String table = "sample_log";
	String column = "log_seq";

	return new ProductIdRangePartitioner(jdbcTemplate, table, column);
}


@Bean(name = "samplePartitionHandler")
public TaskExecutorPartitionHandler partitionHandler() {
	TaskExecutorPartitionHandler partitionHandler = new TaskExecutorPartitionHandler();
	partitionHandler.setStep(step1());
	partitionHandler.setTaskExecutor(executor());
	partitionHandler.setGridSize(10);

	return partitionHandler;
}

...


@Bean
public JdbcPagingItemReader itemReader(
	DataSource dataSource, 
	PagingQueryProvider queryProvider,
	@Value("#{stepExecutionContext[minValue]}") Long minValue,
	@Value("#{stepExecutionContext[maxValue]}") Long maxValue) {
	Map<String, Object> parameterValues = new HashMap<>();
	parameterValues.put("status", "NEW");

	return new JdbcPagingItemReaderBuilder<CustomerCredit>()
		.name("creditReader")
		.dataSource(dataSource)
		.queryProvider(queryProvider(minValue, maxVallue))
		.parameterValues(parameterValues)
		.rowMapper(customerCreditMapper())
		.pageSize(1000)
		.build();
}

@Bean
public SqlPagingQueryProviderFactoryBean queryProvider(int minValue, int maxValue) {
	SqlPagingQueryProviderFactoryBean provider = new SqlPagingQueryProviderFactoryBean();

	provider.setSelectClause("select *");
	provider.setFromClause("from sample_log");
	provider.setWhereClause("where log_seq >= "+minValue +" and log_seq <= " + maxValue);
	provider.setSortKey("log_seq");

	return provider;
}

```

위와같이 `Partitioning`을 통해 `itemReader`을 수행하기 전에 미리 키값을 가지고오는 방식으로, 대상 데이터를 파티셔닝해 구간을 나누어 쿼리를 수행함으로서 `offset` 으로 인한 후반부 read 성능 저하를 어느정도는 해결 할 수 있습니다.

---

### Custom ItemReader

Spring에서 제공하지 않는 `ItemReader`를 직접 구현 할 수도 있습니다.  
`PagingItemReader` 에서 `'Limit Offset'`으로 인해 후반부로 갈 수록 조회성능이 저하되는 현상이 발생했기때문에, 키값을 이용한 `CustomPagingItemReader`를 구현해보도록 하겠습니다.

쿼리는 아래와 같이 수행 되게 될 것 입니다.

```sql
-- OFFSET 후반부 예제 쿼리
-- PK => log_seq

SELECT *
FROM LOG
WHERE log_seq >= 20005000
ORDER BY log_seq
LIMIT 1000

...

SELECT *
FROM LOG
WHERE log_seq >= 20006000
ORDER BY log_seq
LIMIT 1000
```

위내용을 수행하기 위해서는 `ItemReader`를 상속받아 구현해야합니다. 아래 코드는 `AbstractPagingItemReader.java`를 참고하여 구현해본 코드입니다.

```java

public class CustomItemReader implements ItemReader<Map<String, Object>> {
    private JdbcTemplate jdbcTemplate;
    private String targetTable;
    private String targetColumn;
    private Long lastValue;
    private int pageSize;
    private List<Map<String, Object>> results;
    private int resultSize;
    private int readOffset;
    private Object lock = new Object();

    @Override
    public Map<String, Object> read() {

        synchronized (lock) {
            if (CollectionUtils.isEmpty(results) || readOffset >= resultSize) {
                readPage();
            }

            if (CollectionUtils.isEmpty(results))
                return null;

            return results.get(readOffset++);
        }
    }

    public void readPage() {
        String query = "SELECT log_seq, log_title, log_description FROM " + targetTable + "WHERE " + targetColumn + ">=" + lastValue + " ORDER BY " + targetColumn + " LIMIT " + pageSize;
        List<Map<String, Object>> result = jdbcTemplate.query(query, new CustomRowMapper());

        if (!CollectionUtils.isEmpty(result)) {
            resultSize = result.size();
            lastValue = (Long) result.get(resultSize - 1).get(targetColumn);
        }
        
        results = result;
        readOffset = 0;
    }

    public class CustomRowMapper implements RowMapper<Map<String, Object>> {
        @Override
        public Map<String, Object> mapRow(ResultSet rs, int rowNum) throws SQLException {
            Map map = new HashMap<>();
            map.put("seq", rs.getInt("log_seq"));
            map.put("title", rs.getString("log_title"));
            map.put("description", rs.getString("log_description"));

            return map;
        }
    }
}
```

위와같이 원하는 로직으로 DB를 조회하여 데이터를 가지고 올 수 있게끔 `ItemReader`를 직접 구현 해 줄 수 있습니다.

---

### Reference

- [Spring-batch doc](https://docs.spring.io/spring-batch/docs/current/reference/html/readersAndWriters.html)
