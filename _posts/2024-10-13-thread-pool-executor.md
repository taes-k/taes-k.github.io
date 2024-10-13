---
layout: post
comments: true
title: threadPoolExecutor의 쓰레드 할당 방식
tags: [java, spring, springboot, threadPoolExecutor]
---

### threadPoolExecutor

일반적으로 멀티쓰레딩 작업이 필요할때 우리는 별도의 `ThreadPoolTaskExecutor`를 등록하여 쓰레드풀을 통해서 멀티쓰레드가 관리 될 수 있도록 합니다. 이때 작업의 성격에 따라 쓰레드풀을 공용으로도 상요하기도 하고 작업마다 나누어서 사용하기도 하는데, 작업의 트래픽이나 중요도 등을 고려하여 알맞은 쓰레드풀 설정값을 정하게 됩니다.

```java
@Configuration
public class AsyncConfig {
    @Bean(name = "threadPoolTaskExecutor")
    public Executor threadPoolTaskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(50);
        executor.setMaxPoolSize(100);
        executor.setQueueCapacity(1000);
        executor.setKeepAliveSeconds(60);
        executor.setThreadNamePrefix("async-Thread-");
        executor.initialize();
        return executor;
    }
}
```

쓰레드 풀 자체는 위와같이 간단한 코드로 등록해주고 사용 할 수 있고 주요 설정값에 대해서 알아보도록 하겠습니다.

- `CorePoolSize`
    - 최소한으로 유지되는 쓰레드 갯수
    - 해당 설정값만큼의 쓰레드는 초기에 생성되어 작업이 있을 때까지 대기 상태로 유지됨
- `MaximumPoolSize`
    - 풀에서 생성할 수 있는 최대 쓰레드 갯수
    - `corePoolSize` 이상의 작업이 들어오면 이 값만큼까지 추가로 쓰레드를 생성할 수 있음
- `KeepAliveTime`
    - corePoolSize 이상의 쓰레드가 생성되었을때, 새로운 작업이 없으면 해당 쓰레드를 유지하는 시간
    - 설정된 시간이 지나면 해당 쓰레드는 제거됨
- `QueueCapacity`
    - 쓰레드가 수행할 작업을 대기시키기 위한 큐에 대기시킬수 있는 작업의 최대 갯수
    - 대기 작업갯수가 설정된 크기를 넘어서게되면 오류 발생

### executor 에서 쓰레드가 어떻게 수행될까

위의 설정 코드도 직관적이고, 설정값들에 대한 설명을 들었을때도 그다지 어려운것이 없어서 대충 어떤식으로 수행되는지 상상이 되실겁니다. 그럼 다음과 같은 코드를 수행시킨다고 했을 때 쓰레드가 executor 에서 어떻게 수행될지 상상해 보시길 바랍니다.

```java
    @GetMapping("/run/cpu")
    public void runCpuExample() throws ExecutionException, InterruptedException {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        // 1000개의 비동기 작업 실행
        List<CompletableFuture<String>> futures = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            futures.add(asyncExampleService.executeCpuTask(i));
        }

        // 모든 작업이 완료될 때까지 대기
        CompletableFuture.allOf(futures.toArray(new CompletableFuture[0])).join();
        stopWatch.stop();
        log.info("All tasks finished in {} ms",  stopWatch.getTotalTimeMillis());
    }
```

1,000개의 쓰레드를 동시에 실행시키는 작업이니, 작업대기 큐가 넘칠일은 없어 안전할테고 처리 할 갯수가 쓰레드 최소갯수인 `cordPoolSize` 50개보다 많으니 `maxPoolSize`인 100개씩 처리하지 않을까 라고 생각할 수 있는데, 그러나 실제로 수행되는 결과를보면 조금 의아한 점이 있습니다.

![1]({{ site.images | relative_url }}/posts/2024-10-13-thread-pool-executor/1.png)  

로그이름을 살펴보면 그 어느로그도 `async-thread-50` 이상의 숫자가 보이지 않는것을 확인 할 수 있습니다. 

어쩌면 작업이 빠르게 끝나서 쓰레드가 50개 이상 필요없는것이 아닐까라고 생각할수도 있겠지만, 처리 작업량을 더 늘려서 실행시켜보아도 동일한 결과를 얻게 된다는것을 아실 수 있을겁니다. 

그러면, 위 결과를 통해 우리가 설정한 `maxPoolSize`가 우리의 생각처럼 동작하지 않는다는건데, 애초에 이런 혼동이 어디서 왔는지 생각해보면 우리는 DB 연결시 `threadPool` 과 비슷한 역할을 하는 `connectionPool` 을 사용하면서 `minPoolSize`와 `maxPoolSize` 설정해본 경험이 있었을텐데 이 커넥션 풀에서는 우리가 생각했던 것 처럼 `minPoolSize` 이상의 커넥션 요청이 들어오면 최대 `maxPoolSize` 갯수까지 커넥션을 연결 시켜 주는 풀의 역할을 해주지만, 쓰레드풀에서는 조금 다른 수행방식을 가지는것 입니다.

### executor 에서 쓰레드 수행 순서

1. Thread 요청 (~50개)
    - (쓰레드풀의 쓰레드가 `corePoolSize` 만큼 사용중이지 않은상태)
    - 즉시 작업이 쓰레드를 할당받아 수행
2. Thread 요청 (50~1050개)
    - (쓰레드풀의 쓰레드가 `corePoolSize` 만큼 사용중인 상태
    - 더이상 쓰레드를 할당 받을 수 없는 상태
    - (작업대기 큐에 `QueueCapacity` 만큼 대기중이 아닌 상태)
    - 작업대기 큐에 대기
3. Thread 요청 (1050~1100개)
    - (쓰레드풀의 쓰레드가 `corePoolSize` 만큼 사용중인 상태)
    - 더이상 쓰레드를 할당 받을 수 없는 상태
    - (작업대기 큐에 `QueueCapacity` 만큼 대기중인 상태)
    - (쓰레드풀의 쓰레드가 `maximumPoolSize` 만큼 사용중이지 않은상태)
    - 큐의 작업이 신규 쓰레드를 할당받아 수행
    - 신규 작업은 작업대기 큐에 대기
4. Thread 요청 (1100개 초과)
    - (쓰레드풀의 쓰레드가 `corePoolSize` 만큼 사용중인 상태)
    - 더이상 쓰레드를 할당 받을 수 없는 상태
    - (작업대기 큐에 `QueueCapacity` 만큼 대기중인 상태)
    - (쓰레드풀의 쓰레드가 `maximumPoolSize` 만큼 사용중인 상태)
    - `RejectedExecutionException` 에러발생
        ```
        java.util.concurrent.RejectedExecutionException: Task java.util.concurrent.CompletableFuture$AsyncSupply@1843d3c8 rejected from org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor$1@75fc1992[Running, pool size = 100, active threads = 100, queued tasks = 1000, completed tasks = 0]
        ```

![2]({{ site.images | relative_url }}/posts/2024-10-13-thread-pool-executor/2.png)  

위 수행되는 로직을 보면, `maximumPoolSize` 는 단순히 `corePoolSize` 이상의 쓰레드 작업 요청이 있을때 쓰레드 사용을 확장 시킬수 있는것이 아닌, `QueueCapacity` 까지 다 채워져서 더이상 작업대기를 할 수 없는 상황까지 되었을때 작업대기큐가 터지기 전에 처리량을 늘리기위한 설정이라고 볼 수 있습니다. 

쓰레드풀을 설정할때 이렇게 평상시에도 `maximumPoolSize` 만큼 수행될것을 가정하고 설정하면 안되고, 일반적인 상황에도 `maximumPoolSize` 만큼의 쓰레드 수행을 기대하고 있다면, `corePoolSize`와 `maximumPoolSize`를 둘다 100개씩으로 설정하는 방식으로 세팅이 필요하는것을 잊으면 안됩니다.