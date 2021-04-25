---
layout: post
comments: true
title: Java 바이트 코드조작하기
tags: [java, bci]
---

### Java class file

Java 바이트 조작에대한 글을 쓰기 앞서서, 간략하게 Java 프로그램이 실행되는 과정에대해 짚고 가겠습니다.  

아시다시피 Java는 `JVM`이라는 독립적인 환경에서 프로그램을 수행시기 때문에 OS에 대해 비교적 덜 의존적이라는 특성을 갖습니다. 이러한 이유로 인해 우리가 작성한 Java 프로그램은 일련의 과정을 거쳐 `JVM` 로딩 후 사용하게 됩니다.


![1]({{ site.images | relative_url }}/posts/2021-04-25-byte-code-instrumentation/1.png)   

위의 그림과 같이 개발자들이 작성한 java 코드는 `컴파일` 과정을 통해서 `JVM`에서 사용가능한 class 파일(byte-code)로 변환되고 `클래스 로딩` 과정을 거쳐 `JVM  ` 메모리에 적재되어 실행가능한 프로그램이 됩니다.

여기서 주목해야 할 것은, 최종적으로 `JVM`에 적재되는 코드는 개발자가 작성한 `java 파일`가 아닌 `class 파일` 즉, 바이트 코드인것을 기억해두셔야 합니다.

---

### Java 바이트 코드 조작

위에서 알아본 내용을 통해 이번 포스팅에서 다루고자 하는 내용을 어느정도 예상하셨을거라 생각합니다.  

우리가 일반적으로 프로그램 동작을 변경시키고자 할때는 `java 코드`를 수정해 컴파일 레벨에서 코드를 변경시키지만, `Byte 코드`를 수정해서 원하는 기능을 반영 할 수도 있습니다. 

즉, 개발자의 소스코드를 직정 수정없이 원하는 기능을 구현 할 수 있다는 장점을 갖습니다.

생소하실수도 있겠지만 이 방식은 프로파일링, 모니터링 툴에서 사용되는 경우가 많고 `Spring boot AOP`에서 사용하는 `CGLIB` 바이트코드 조작을 통한 Proxy를 제공하고 있습니다.


![2]({{ site.images | relative_url }}/posts/2021-04-25-byte-code-instrumentation/2.png)   

Java에서 대표적으로 Byte 코드 조작을 위해 사용할수 있는 대표적인 라이프러리들은 아래와 같습니다.

- ASM
- ByteBuddy
- JavaAssist
- CGLIB


---

### 바이트 코드 조작 예제

`ByteBuddy`를 활용해 바이트 코드 조작 예제를 보도록 하겠습니다.

`Target`클래스의 `getTarget()`메서드의 반환값을 `Target.java` 소스코드 변경 없이 변경해보는 예제입니다.

```java
// Target.java

public class Target
{
    public String getTarget()
    {
        return "target";
    }
}
```

```java
public class Main
{
    public static void main(String[] args)
    {
        ByteBuddyAgent.install();
        new ByteBuddy()
            .redefine(Target.class)
            .method(ElementMatchers.named("getTarget"))
            .intercept(FixedValue.value("changedTarget"))
            .make()
            .load(Target.class.getClassLoader(), ClassReloadingStrategy.fromInstalledAgent());

        System.out.println("TARGET : " + new Target().getTarget());
    }

}
```


```java
System.out.println("TARGET : " + new Target().getTarget());
``` 
일반적인 상황이라면, 위 코드 호출시 아래와 같은 결과가 나오겠지만, 
> Target : target  

의 결과가 나와야 겠지만, ByteBudyy를 사용해 `getTarget` 메서드의 결과를 `changedTarget`으로 변경하도록 했기 때문에 다음과 같은 결과를 확인할수 있습니다.


![3]({{ site.images | relative_url }}/posts/2021-04-25-byte-code-instrumentation/3.png)   

그런데 실제로 컴파일된 `Target.class` 파일을 보면, 조금 의아해 하실수도 있습니다.

![4]({{ site.images | relative_url }}/posts/2021-04-25-byte-code-instrumentation/4.png)  

결과값을 보면 분명히 바이트조작을 통해서 `Target.class` 파일이 변경되었을거라 예상하셨을수도 있으실텐데 class 파일을 변경하는것이 아닌 `클래스로딩`시점에 변경된 바이트코드를 로딩하기때문입니다. 

이와같이 Spring에서 사용하는 `CGLIB`또한, 바이트코드를 조작해 `Proxy`를 만들어 `AOP`를 구현하고 있습니다.