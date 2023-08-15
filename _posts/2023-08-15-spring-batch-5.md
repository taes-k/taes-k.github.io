---
layout: post
comments: true
title: Spring batch 5
tags: [spring, jdbc, springdatajdbc]
---

### Spring batch 5 메이저 업데이트

작년 말, Spring 6, Spring boot 3.0 업데이트와 함께 Spring batch 또한 5.0 으로 메이저 업데이트가 되었습니다. 업데이트 된지는 반년이 지났지만 최근 배치 프로젝트 작업을 진행하면서 확인했던 변경점들을 정리해보려고 합니다.

---

### MetaTable 변경 - 다양한 파라미터 타입 지원

[이전 포스팅](https://taes-k.github.io/2021/03/01/spring-batch-table/)에서 스프링 배치 메타테이블들의 역할에 대해 살펴본적이 있습니다. 스프링 배치를 수행하기 위해 필수로 필요한 테이블들인데 이번 업데이트에서 이 테이블 구조가 변경되어 기존에 이전버전의 스프링 배치를 사용하고 있던경우 메타테이블을 그대로 사용하실 수 없습니다.

Spring Batch5 이전

![1]({{ site.images | relative_url }}/posts/2023-08-15-spring-batch-5/1.png)  

Spring Batch5

![12]({{ site.images | relative_url }}/posts/2023-08-15-spring-batch-5/2.png) 

https://docs.spring.io/spring-batch/reference/schema-appendix.html


변경된점을 살펴보면 `BATCH_JOB_EXECUTION_PARAMS` 테이블에 변경이 크게 일어난것을 확인하실수 있습니다. 기존에는 배치 파라미터로 4개의 Type(`Long`, `Double`, `String`, `Date`)만 지원하여 각각의 타입별로 칼럼을 제공하던 방식과는 다르게 파라미터 Type 을 저장하여 다양한 타입의 파라미터를 제공할 수 있는 방향으로 변경되었다고 볼 수 있겠습니다.

---

### Job,Step BuilderFactory Deprecated

기존 배치에서는 `Job`, `Step` 생성을 위해 아래와 같은 코드를 통해 생성해주었었습니다.

```java
@RequiredArgsConstructor
@Configuration
public class BatchConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job sampleJob() {
        return jobBuilderFactory.get("sampleJob")
            .start(myStep())
            .build();
    }

    @Bean
    public Step smapleStep() {
        return stepBuilderFactory.get("smapleStep")
            .<String, String>chunk(10)
            .reader(reader())
            .processor(processor())
            .writer(writer())
            .build();
    }
}
```

SpringBatch5 에서는 `JobBuilderFactory`, `StepBuilderFactory`가 Deprecated 되었습니다. 아래와 같은 코드로 대체할 수 있습니다.

```java
@RequiredArgsConstructor
@Configuration
public class BatchConfiguration {

    private final JobRepository jobRepository;
    private final PlatformTransactionManager transactionManager;

    @Bean
    public Job sampleJob() {
        return new JobBuilder("sampleJob", jobRepository)
            .start(myStep())
            .build();
    }


    @Bean
    public Step myStep() {
        return new StepBuilder("myStep", jobRepository)
            .<String, String>chunk(10, transactionManager)
            .reader(reader())
            .processor(processor())
            .writer(writer())
            .build();
    }
}
```

사용법에 크게 달라진것도 없고 사실 기존 `JobBuilderFactory`, `StepBuilderFactory` 코드를 보면 Factory 에서 수행되고 있던 코드를 끄집어내서 사용하는것을 확인하실수 있습니다.

```java
@Deprecated(since = "5.0.0", forRemoval = true)
public class JobBuilderFactory {
	private final JobRepository jobRepository;

	public JobBuilderFactory(JobRepository jobRepository) {
		Assert.notNull(jobRepository, "JobRepository must not be null");
		this.jobRepository = jobRepository;
	}
	public JobBuilder get(String name) {
		return new JobBuilder(name, this.jobRepository);
	}
}

@Deprecated(since = "5.0.0", forRemoval = true)
public class StepBuilderFactory {
	private final JobRepository jobRepository;
	public StepBuilderFactory(JobRepository jobRepository) {
		Assert.notNull(jobRepository, "JobRepository must not be null");
		this.jobRepository = jobRepository;
	}
	public StepBuilder get(String name) {
		return new StepBuilder(name, this.jobRepository);
	}
}
```

https://github.com/spring-projects/spring-batch/issues/4188

어쩌면 일반적으로 잘 쓰고 있던 Facotry가 deprecated 되는데 의아함을 가지실수도 있는데 이유를 다음과같이 설명하고 있습니다.

- 암시적인 구성을 통해 builder 내에 JobRepository 가 설정되는것을 숨김
- 여러 저장소를 갖는 프로젝트의 경우 JobBuilderFactory에 구성되는 JobRepository 설정을 위해 custom `@EnableBatchProcessing`을 구성해함

위와같은 이유로인해 `JobRepository`, `TransactionTemplate`을 배치 의도에 맞게끔 명시적인 설정을 지향하기 위해 depreacted 했다고 볼 수 있겠습니다.

---

### `@EnableBatchProcessing` 기능 추가

기존에 Spring Batch 기본 설정을 위해 `@EnableBatchProcessing`을 사용합니다. 

```java
@Configuration
@EnableBatchProcessing(dataSourceRef = "batchDataSource", transactionManagerRef = "batchTransactionManager")
public class MyJobConfiguration {

	@Bean
	public Job job(SampleJob sampleJob) {
		return new JobBuilder("sampleJob", jobRepository)
				.build();
	}

}
```

위에서 살펴본 내용 연관으로 여기에 dataSourceRef, transactionRef을 정의하여 사용할수 있는 기능이 추가되었습니다. 이 방식 외에도 프로그래밍 방식으로 설정 가능한 `DefaultBatchConfiguration` 기능도 추가되었습니다.

```java
 @Configuration
 public class MyJobConfiguration extends DefaultBatchConfiguration {

     @Bean
     public Job sampleJob(JobRepository jobRepository) {
         return new JobBuilder("sampleJob", jobRepository)
                 // define job flow as needed
                 .build();
     }

 }
 ```

---

### 그리고 여러가지...

- java17 : SpringBatch5 는 SpringFramework6 의존성을 따르므로 java 최소 요구조건으로 java17 을 요구합니다.
Micrometer
- MariaDB : 기존에는 MySQL 으로 간주하여 처리했지면 MariaDB 완벽지원
- MavenBom : SpringBatch 모듈을 위한 일괄적인 버전관리 가능
- GraalVM : GraalVM 기본 지원
- 등 ...

---

### reference
- [스프링 Document](https://docs.spring.io/spring-batch/docs/5.0.2/reference/html/whatsnew.html)
- [Migration Guide](https://github.com/spring-projects/spring-batch/wiki/Spring-Batch-5.0-Migration-Guide)
- [What's New](https://docs.spring.io/spring-batch/docs/5.0.0/reference/html/whatsnew.html#whatsNew)