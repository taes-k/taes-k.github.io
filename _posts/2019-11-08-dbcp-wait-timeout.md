---
layout: post
comments: true
title: Spring dbcp(Hikari pool) wait-timeout connection error
tags: [spring, dbcp, hikari]
---

### 에러내용   

```
com.zaxxer.hikari.pool.ProxyConnection : HikariPool-1 - Connection com.mysql.cj.jdbc.ConnectionImpl@25ec8a5c marked as broken because of SQLSTATE(08S01), ErrorCode(0)
com.mysql.cj.jdbc.exceptions.CommunicationsException: The last packet successfully received from the server was 3,752,361 milliseconds ago.  The last packet sent successfully to the server was 3,752,363 milliseconds ago. is longer than the server configured value of 'wait_timeout'. You should consider either expiring and/or testing connection validity before use in your application, increasing the server configured values for client timeouts, or using the Connector/J connection property 'autoReconnect=true' to avoid this problem.
        at com.mysql.cj.jdbc.exceptions.SQLError.createCommunicationsException(SQLError.java:174)
        at com.mysql.cj.jdbc.exceptions.SQLExceptionsMapping.translateException(SQLExceptionsMapping.java:64)
        at com.mysql.cj.jdbc.ConnectionImpl.setAutoCommit(ConnectionImpl.java:2054)
        at com.zaxxer.hikari.pool.ProxyConnection.setAutoCommit(ProxyConnection.java:388)
        at com.zaxxer.hikari.pool.HikariProxyConnection.setAutoCommit(HikariProxyConnection.java)
        at org.springframework.batch.item.database.AbstractCursorItemReader.doClose(AbstractCursorItemReader.java:402)
        at org.springframework.batch.item.support.AbstractItemCountingItemStreamItemReader.close(AbstractItemCountingItemStreamItemReader.java:138)
        at org.springframework.batch.item.support.CompositeItemStream.close(CompositeItemStream.java:89)
        at org.springframework.batch.core.step.item.ChunkMonitor.close(ChunkMonitor.java:106)
        at org.springframework.batch.item.support.CompositeItemStream.close(CompositeItemStream.java:89)
        at org.springframework.batch.core.step.tasklet.TaskletStep.close(TaskletStep.java:306)
        at org.springframework.batch.core.step.AbstractStep.execute(AbstractStep.java:274)
        at org.springframework.batch.core.job.SimpleStepHandler.handleStep(SimpleStepHandler.java:148)
        at org.springframework.batch.core.job.flow.JobFlowExecutor.executeStep(JobFlowExecutor.java:68)
        at org.springframework.batch.core.job.flow.support.state.StepState.handle(StepState.java:67)
        at org.springframework.batch.core.job.flow.support.SimpleFlow.resume(SimpleFlow.java:169)
        at org.springframework.batch.core.job.flow.support.SimpleFlow.start(SimpleFlow.java:144)
        at org.springframework.batch.core.job.flow.FlowJob.doExecute(FlowJob.java:136)
        at org.springframework.batch.core.job.AbstractJob.execute(AbstractJob.java:313)
        at org.springframework.batch.core.launch.support.SimpleJobLauncher$1.run(SimpleJobLauncher.java:144)
        at org.springframework.core.task.SyncTaskExecutor.execute(SyncTaskExecutor.java:50)
        at org.springframework.batch.core.launch.support.SimpleJobLauncher.run(SimpleJobLauncher.java:137)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        ...
```

---

### 에러이유  

위 에러는 기본적으로 DB 의 커넥션 유지시간과 DBCP의 커넥션 유지시간이 달라 DB에서는 연결이 해제된 커넥션을 DBCP에서는 해당커넥션을 들고있다가 요청이 들어올때 해당 커넥션으로 연결하여 사용해 생기는 오류입니다. 
위 에러가 발생한 어플리케이션은 스프링 배치 커서를 이용한 어플리케이션으로, 데이터를 초기에 한번만 불러온 후 DB 연결 없이 1시간정도 작업 후 마지막에 커넥션을 닫기위해 커넥션에 접근을 해주게 되는데 이때,  wait_time out이 발생하는것으로 확인됩니다.   
  
---

### 해결방법  
에러 로그에서는 친절하게도 해결방법을 알려주고 있습니다.   

- test connection을 사용한다.  
- timeout 설정값을 증가시킨다.  
- Connector/J porperty중 'autoReconnect=true'를 사용한다.  

이중 클라이언트 단에서 가장 간단하게 처리할수 있는 방법은 jdbc 옵션인 'autoReconnect=true'를 사용하는 방법인데, 이옵션을 사용하게되면 쿼리 수행후 DB 세션에 문제가 있을시 SQLException을 리턴후 re-connect를 시도하게됩니다.    

여기서 생기는 문제가 하나 있는데, 트랜잭션 환경을 사용한다면 에러와함께 새로운 커넥션 연결이 되면서 이전 커넥션에서의 트랜잭션은 롤백이되고 다음 쿼리 진행은 새로운 커넥션에서 새로운 트랜잭션으로 시작하게되는 문제점이 있습니다.  

> begin   
> 쿼리1 : 롤백  
> 쿼리2 >> connection error , auto reconnect : 실행  
> 쿼리3 : 실행  
> commit  
  
위와같이 하나의 트랜잭션으로 유지를 보장하지 못한다는 문제가 있기때문에 트랜잭션 환경에서 사용은 비권장하고 있습니다.   

트랜잭션을 사용하는 환경이라면 DB 세션의 재접속 처리는 validationQuery 사용을 권장하고 있습니다. validationQuery 를 사용하게되면  커넥션풀에서 커넥션을 얻어올때/반환할때/주기적으로 테스트를 실행하여 비정상적인 커넥션을 커넥션풀에서 제거 시키거나, 주기적으로 명령을 실행시켜 커넥션을 유지할수 있도록 해줍니다.  
  
> test-on-borrow: 커넥션 풀에서 커넥션을 얻어올때 validation (default: true)  
> test-on-return: 커넥션 풀로 커넥션을 반환할때 validation (default: false)  
> test-while-idle: 주기적으로 커넥션 풀 내의 사용가능 상태의 커넥션 validation(기본값: false)  
  
validationQuery를 사용하게되면 커넥션 사용할때마다 추가적인 비용이 발생하는 것이므로, 적절한 설정 사용이 필수적입니다.  
  
---

### 추가내용 (2019.11.11)  

위의 내용은 Spring boot 2.0 이전에 사용하였던 tomcat dbcp의 설정내용입니다.  
Spring boot 2.0 이후 버전에서 사용하는 Hikari pool 에서는 위와같은 validation은 권장하지 않고 있습니다.  
  
관련 내용 링크 : https://pkgonan.github.io/2018/04/HikariCP-test-while-idle
  
위와같은 validation을 통해 connection을 유지시키는 것은 사실 DB에서 설정한 wait-time-out을 무시하고 강제적으로 idle 한 상태로 만드는 것이기 때문에 Hikari 에서는 위와같은 개념에서 탈피하여 db가 설정한 wait-time-out을 최대한 반영하고, 죽은 커넥션들은 Hikari pool 에서 버리는 방식으로 구현되어있고, 권장하고있습니다.   
  
따라서 Hikari pool을 사용할때에는 DB wait-time-out 시간보다 5초정도 짧게 max-live-time(default: 30min)을 설정해주어 pool 내부에는 언제나 idle한 커넥션들을 유지하여 사용하는 방법이 가장 이상적입니다.  
  
하지만 배치에서 작업에서생기는 문제는 이 방법으로는 해결이 불가능할것같아 spring batch에서 사용하는 jdbcCursorItemReader에 대해서 좀더 알아봐야 할것 같습니다.   
Spring jdbcCursorItemReader doc : https://docs.spring.io/spring-batch/docs/2.0.x/apidocs/org/springframework/batch/item/database/JdbcCursorItemReader.html 