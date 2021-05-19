---
layout: post
comments: true
title: 몰라도 되는 Spring - Spring batch scope
tags: [몰라도 되는 Spring, spring, batch, scope]
---

### Spring scope

아시다시피, `Spring bean`의 life-cycle을 정의하는 Scope는 아래와 같습니다.

> - singleton
> - prototype
> - request
> - session
> - application
> - websocket

갑자기 왠 `Scope` 이야기를 하는지 의문이 드실수도 있는데, `Spring batch`를 사용할때 중요하게 알고 가야 할 내용이기 때문에 이번 포스팅에서 다루고자 합니다.  

`Spring batch`에서는 위 `Spring framework`에서 제공하는 `scope`외에 추가적인 `scope`가 제공됩니다.  
[이전 포스팅](https://taes-k.github.io/2021/03/01/spring-boot-table/)인 메타테이블 구조에서 알아보았듯이, `Spring batch`는 `Job`, `Step` 을 통해 작업이 수행되게 되는데 이 작업단위별로 `scope`를 지정해 수행이 가능합니다.

즉, `Bean`의 생명주기가 `Job` 실행할때마다 생성 되거나, `Step`이 실행될때마다 생성되도록 설정 해 줄수 있는 내용입니다.

> - @JobScope
> - @StepScope

---

### Spring batch scope

`Spring batch scope`가 어떻게 사용되는지 알아보기 위해 우선, 예제코드를 준비했습니다.

```java

    @Bean
    public Job sampleJob()
    {
        return jobBuilderFactory.get("sampleJob")
            .start(sampleStep())
            .build();
    }

    @Bean
    public Step sampleStep()
    {
        return stepBuilderFactory.get("sampleStep")
            .<String, String>chunk(1_000)
            .reader(sampleItemReader())
            .writer(sampleItemWriter())
            .build();
    }

    @Bean
    public FlatFileItemReader<String> sampleItemReader()
    {
        FlatFileItemReader<String> flatFileItemReader = new FlatFileItemReader<>();
        flatFileItemReader.setResource(new FileSystemResource("/resource/sample.csv"));
        flatFileItemReader.setLineMapper((line, lineNumber) -> line);
        flatFileItemReader.setLinesToSkip(1);
        return flatFileItemReader;
    }

    @Bean
    public ItemWriter<String> sampleItemWriter()
    {
        return csvOtherCompanyProds ->
        {
            log.info("ITEM write ...");
            ...
        }
    }
        
```

위와같이 Job, Step들은 `Bean`으로 등록되어 사용되어지고 있습니다.  
`Bean`의 Default scope는 `Singleton` 으로 구성되어있는데, 그렇다면 호출할때마다 파라미터가 필요하면 어떻게 처리해야 할까요?  

위 예제에서 `File path`를 넘겨서 처리하기 위해 파라미터를 추가해 보겠습니다.

```java

    @Bean
    public Job sampleJob(String filePath)
    {
        return jobBuilderFactory.get("sampleJob")
            .start(sampleStep(filePath))
            .build();
    }

    @Bean
    public Step sampleStep(String filePath)
    {
        return stepBuilderFactory.get("sampleStep")
            .<String, String>chunk(1_000)
            .reader(sampleItemReader(filePath))
            .writer(sampleItemWriter())
            .build();
    }

    @Bean
    public FlatFileItemReader<String> sampleItemReader(String filePath)
    {
        FlatFileItemReader<String> flatFileItemReader = new FlatFileItemReader<>();
        flatFileItemReader.setResource(new FileSystemResource(filePath));
        flatFileItemReader.setLineMapper((line, lineNumber) -> line);
        flatFileItemReader.setLinesToSkip(1);
        return flatFileItemReader;
    }

    @Bean
    public ItemWriter<String> sampleItemWriter()
    {
        return csvOtherCompanyProds ->
        {
            log.info("ITEM write ...");
            ...
        }
    }
        
```

위처럼 설정시, 처음에 초기화된 `Singleton Bean`이 더이상 재정의 되지 않기때문에 Job 실행시마다 `filePath` 를 넘겨서 동적인 처리를 하려했던 의도를 수행하지 못하게 됩니다. 

그렇기 때문에, 위 `Bean`을 사용할 때 `Singleton`이 아닌 `Job, Step scope` Bean으로 등록해주어야 하는 이유입니다.

`@JobScope`, `@StepScope`를 사용하게 되면, 지난 포스팅에서 확인했던 메타테이블의 `JobParameter`를 Bean 등록 시점에서 사용 할 수 있습니다.

`JobParameter`를 아래와같이 설정후 Job을 수행시키는 예제를 보도록 하겠습니다.

```java
    public void runJob(String filePath)
    {
        JobParameters params = new JobParametersBuilder()
            .addString("JobID", String.valueOf(System.currentTimeMillis()))
            .addString("filePath", filePath)
            .toJobParameters();

        jobLauncher.run(targetJob, params);
        runJob(sampleJob(), params);
    }

    @Bean
    public Job sampleJob()
    {
        return jobBuilderFactory.get("sampleJob")
            .start(sampleStep())
            .build();
    }

    @Bean
    public Step sampleStep()
    {
        return stepBuilderFactory.get("sampleStep")
            .<String, String>chunk(1_000)
            .reader(sampleItemReader(null))
            .writer(sampleItemWriter())
            .build();
    }

    @StepScope
    @Bean
    public FlatFileItemReader<String> sampleItemReader(@Value("#{jobParameters[filePath]}") String filePath)
    {
        FlatFileItemReader<String> flatFileItemReader = new FlatFileItemReader<>();
        flatFileItemReader.setResource(new FileSystemResource(filePath));
        flatFileItemReader.setLineMapper((line, lineNumber) -> line);
        flatFileItemReader.setLinesToSkip(1);
        return flatFileItemReader;
    }

    @Bean
    public ItemWriter<String> sampleItemWriter()
    {
        return csvOtherCompanyProds ->
        {
            log.info("ITEM write ...");
            ...
        }
    }
        
```

위 코드에서 자세히 보셔야 할 내용은 바로 아래의 코드 부분입니다.
```java
    @StepScope
    @Bean
    public FlatFileItemReader<String> sampleItemReader(@Value("#{jobParameters[filePath]}") String filePath)
```

`@StepScope`로 지정된 `sampleItemReader`는 매번 Step이 수행될때마다 `jobParameter`로 넘겨진 'filePath' 값을 조회해 Bean을 등록해 사용하기 때문에 Job이 수행될 때마다 넘어온 `filePath`에서 itemRead를 할 수 있게 되었습니다.  

정적으로 동일한 로직을 수행하는 Job 이라면 `Singleton`으로 구현하는게 좋지만, 위 예제와같이 수행로직이 변경되어야 하는 경우에는 scope를 지정해 간단하게 동적 로직 수행이 가능합니다.
