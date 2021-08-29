---
layout: post
comments: true
title: Java biased lock
tags: [java, labdbiased lock]
---

### java lock

이번 포스팅의 주제인 `biased-lock`에 대해 이야기하기 앞서, `java`에서 `lock`이 어떻게 구현되어지고 사용되는지 먼저 알아보도록 하겠습니다.

```java
public class Sample{

    public syncronized void runSample(){
        
        // do something
        ...
        
    }
}
```

일반적으로 `Java`객체에서 `lock`을 사용한다고 하면 위 코드와 같이 `synchronized` keyword를 사용해 간단하게 구현이 가능한데요, 이를 `Intrinsic lock` 이라고 합니다.

일반적인 상황에서 이 `lock`은 객체의 `monitor`에 의해 제어되는데, 객체의 `monitor`를 획득에 성공했을때에만 실행 가능하게 하여 경쟁상황에서 하나의 실행만을 보장하게 됩니다.  
이때 `lock`은 `OS`기반으로 관리가되어지는데 `lock`의 획득/해제시에 `System call`을 사용하게되어 `user mode` <-> `kerner mode` `context switching`이 발생하게 됩니다.

위와같은 `lock` 사용방식을  `Heavy-weight lock`이라고 말합니다.

---

### Light-weight lock

여러 쓰레드에의해 `lock` 경쟁이 발생하더라도 `lock`은 짧은시간동안 보유하고 해제된다는 경험적인 판단하에, `lock`의 관리를 `OS`에 맡기지 않고 `Spin-lock`을 통해 `CAS(Compare And Swap)` 방식으로 처리하는 방식입니다.

위에서 알아보았던 `Heavy-weight lock`에서 발생 하는 `context switching` 오버헤드를 피할수 있지만, `Spin-lock`에서 사용되는 리소스의 낭비가 있을 수 있습니다.

---

### Biased lock

이번 포스팅의 주제인 `Biased lock`은 `jdk 1.6`부터 제공된 `lock` 최적화 기법중 하나입니다.

개발을 하면서 일반적으로 쓰레드간의 경쟁을 피하기 위해 `Synchronized` 키워드를 통해 쉽게 `lock`을 설정하지만, 그 중 일부는 여러 쓰레드간의 경쟁이 발생하기 보다는 동일한 스레드에서만 접근이 있는경우가 자주 발생 합니다. 

위와 같은 경우 모니터를 할당받고 해제하는 과정이 성능상 비효율적일 수 있는데, `thread id`를 체크하여 lock을 획득하고자 하는 쓰레드가 반복해서 동일한 쓰레드라고 판단되면 `OS lock`을 사용하지 않도록합니다.

---

### lock optimization

객체는 `JVM`에서 `OOP(Ordinary object pointer)` 라는 구조체로서 나타나게되는데 객체 Header의 일부인 `Mark word`에서 `lock` 정보가 관리됩니다.

![1]({{ site.images | relative_url }}/posts/2021-08-29-java-biased-lock/1.png)   

객체의 상태에따라 위와같이 5가지 케이스로 저장됩니다.  
위에서 알아보았던 객체의 `lock` 상태에 따라 다른 케이스로 관리되는것을 확인하실 수 있습니다.

그렇다면 객체는 어떤 절차를 통해서 위 `lock`의 상태를 결정하게 될까요?  
아래 사진을 통해 조금더 자세히 알아보도록 하겠습니다.

![2]({{ site.images | relative_url }}/posts/2021-08-29-java-biased-lock/2.png)   

(출처: [openJdk wiki](https://wiki.openjdk.java.net/display/HotSpot/Synchronization))

위 다이어그램을 통해 파악 할 수 있다시피 `lock` 상태는 객체가 생성될때 바로 특정지어지지 않습니다. 객체가 사용되어지면서 경험적으로 `lock`상태가 결정지어지게 되면 순차적으로 업데이트가 이루어지게됩니다.

- biased lock
- light-weight lock
- heavy-weight lock

이는 위에서 계속 알아보았다시피 `lock`의 오버헤드를 최대한 줄이고자하는 최적화를 위한 `step`이라고 생각하면 이해하는데 도움이될것이라 생각합니다.

---

### Biased lock in recent Java

위에서 열심히 `Biased lock`에 대해서 설명드렸지만, 최근 `Java`에서는 `Biased lock`이 `Deprecated` 되었습니다. 

- https://openjdk.java.net/jeps/374

Deprecate 이유를 패치노트에서 확인해보면 아래와 같습니다.
-  과거 java API에서의 `Biased lock`의 성능향상은 분명했지만 최근 java API 에서는 자체적으로 개선 및 최적화가 많이 되었기때문에 `Biased lock`이 주는 효과가 크지 않습니다.
- `Biased lock`이 비활성화된 상태에서 `Thread-pool`, `Worker-thread` 로 구성된 프로그램이 더 잘 동작합니다.
- `Biased lock`의 해지작업에서 발생하는 Overhead를 제거합니다.
- ...

즉, 과거 `java`비해 `Biased lock`을 통해 얻을수 있는 이점이 많이 줄어든것으로 납득할만한 이유를 확인하실수 있습니다.

만약 `Biased lock`을 믿고 `Synchronized`를 남용하여 개발하셨던 경우 코드의 수정이 필요 할 수도 있을것 같습니다. 

아직 많은분들이 `java8`, `java11`을 사용하고 계셔서 영향은 없으시겠지만 올해(`2021`)말 `LTS` 버전인 `java17`이 오픈 예정인만큼 미리 알아두시고 대비해두시면 좋을것 같습니다.

---

### Reference

- https://www.codetd.com/en/article/12978167
- https://wiki.openjdk.java.net/display/HotSpot/Synchronization
- https://blogs.oracle.com/dave/biased-locking-in-hotspot
- https://openjdk.java.net/jeps/374