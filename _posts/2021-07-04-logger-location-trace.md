---
layout: post
comments: true
title: Logger의 줄번호 출력 성능 이슈
tags: [log4j, slf4j, logback, logger, stacktrace]
---

### Logger에서 줄번호를 찾는 방법   

`log4j`, `logback`, `log4j2`등 `logger`를 사용하실때 일반적으로 아래와 같은 패턴을 사용하여 로깅 패턴이 설정되어있는 경우가 많이 있을것 입니다.

```
%F : 파일명
%C : 클래스명
%L : 줄번호
%M : 메서드명
```

`Logger`에서는 로그를 생성하기위해 현재 수행중인 위치에서 `StackTrace`를 발생 시켜 위 정보들을 조회하고 있습니다.

Java에서 StackTrace를 조회하는 방법은 아래 방법들이 존재합니다.


- Throwable을 통한 조회
```
Throwable().getStackTrace()
```

- Thread를 통한 조회
```
 Thread.currentThread().getStackTrace()
```

- ThreadWalker를 통한 조회 (Upper Java 9)

```
StackWalker.getInstance().walk((s) -> s.collect(Collectors.toList()))
```

StackTrace 를 호출하면 List<StackTraceElement>를 반환하는데,

`StackTraceElement 객체`에는 Class name, Method name, Line number 등의 정보를 가지고 있는것을 확인 할 수 있습니다.

![1]({{ site.images | relative_url }}/posts/2021-07-04-logger-location-trace/1.png)   

위 코드들을 보시면 `StackTraceElement` 객체에는  `lineNumber` 뿐만아니라 `className`, `MethodName` 등의 정보도 포함하고 있음을 알 수 있습니다.

실제 Logger에서 어떻게 사용되고 있는지 코드를 확인해보겠습니다.

**Logback**
- LoggingEvent.java

![2]({{ site.images | relative_url }}/posts/2021-07-04-logger-location-trace/2.png)   

- CallerData.java

![3]({{ site.images | relative_url }}/posts/2021-07-04-logger-location-trace/3.png)   


**Log4j**
- LogAdaptor.java

![4]({{ site.images | relative_url }}/posts/2021-07-04-logger-location-trace/4.png)   


로거들에서 StackTrace 정보를 사용하기위해  `Throwable().getStackTrace()`를 호출하는것을 확인하실 수 있습니다.

---

### Stacktrace overhead


로그에 Location을 찍게되면 로깅을 할때마다 `Throwable().getStackTrace()`를 호출한다는 것인데 

overhead가 얼마나 발생하는지 알아보기위해 직접 테스트를 수행해보도록 하겠습니다.

테스트 코드)

```java
public class SampleTest {
 
    private StopWatch stopWatch = new StopWatch();
 
    @BeforeEach
    void setStartWatch() {
        stopWatch.start();
    }
 
    @AfterEach
    void setStopWatch() {
        stopWatch.stop();
        System.out.println(stopWatch.getTotalTimeMillis()+" ms");
    }
 
    @DisplayName("Stacktrace테스트_비교군_10,000")
    @Test
    void StackTraceTest_base_10000(){
        for(int i =0; i< 10_000; i++) {
            new Throwable();
        }
    }
 
    @DisplayName("Stacktrace테스트_getSTackTrace_10,000 (Throwable)")
    @Test
    void StackTraceTest_getStackTrace_10000(){
        for(int i =0; i< 10_000; i++) {
            new Throwable().getStackTrace();
        }
    }
 
    @DisplayName("Stacktrace테스트_비교군_100,000")
    @Test
    void StackTraceTest_base_100000(){
        for(int i =0; i< 100_000; i++) {
            new Throwable();
        }
    }
 
 
    @DisplayName("Stacktrace테스트_getSTackTrace_100,000 (Throwable)")
    @Test
    void StackTraceTest_getStackTrace_100000(){
        for(int i =0; i< 100_000; i++) {
            new Throwable().getStackTrace();
        }
    }
}
```

**결과)**

![5]({{ site.images | relative_url }}/posts/2021-07-04-logger-location-trace/5.png)   

- stackTrace 10,000회 : 55ms → 315ms (572%)
- stackTrace 100,000회 : 616ms → 3157ms (512%)

StackTrace를 호출하는 다른 방법들도 함께 비교해보면 아래와 같습니다


![6]({{ site.images | relative_url }}/posts/2021-07-04-logger-location-trace/6.png)   


- stackTrace 10,000회 (Throwable): 59ms → 306ms (518%)
- stackTrace 10,000회 (Thread): 59ms → 336ms (569%)
- stackTrace 10,000회 (ThreadWalker): 59ms → 329ms (557%)
- stackTrace 100,000회 (Throwable): 563ms → 3086ms (548%)
- stackTrace 100,000회 (Thread): 563ms → 3162ms (561%)
- stackTrace 100,000회 (ThreadWalker): 563ms → 2484ms (441%)

---

### 결과도출

테스트 결과를 보면 StackTrace의 사용이 결코 싸지 않다는것을 확인하실수 있습니다.  

Java9 부터 개선된 ThreadWalker를 제공하기는 하지만, 이또한 비용이 싸지않습니다.

Local Debug 용도라면 괜찮지만, Log가 자주 발생하는 운영환경이라면 Location logging 제거만으로 로깅성능개선 효과를 보실 수 있을거라 기대합니다.




