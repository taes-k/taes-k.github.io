---
layout: post
comments: true
title:  몰라도 되는 Spring - 리플렉션으로 만드는 Dynamic proxy
tags: [spring, aop, proxy, reflection]
---

### Dynamic proxy

이전 [AOP Proxy](https://taes-k.github.io/2021/02/07/spring-aop-proxy/) 포스팅에서 Spring 에서 사용하는 `Dynamic proxy`와 `CGLib`를 이용한 proxy에 대해 말씀드리면서 다음과 같은 설명이 있었습니다. 

> JDK dynamic proxy  
> 
> - Reflection을 통해 동적으로 proxy 객체 생성  
> - interface를 기준으로 proxy 생성  

`JDK Dynamic proxy`는 'Reflection 을 통해 동적으로 proxy 객체를 생성한다.' 라고 했는데 좀더 정확하게 전달드리기 위해 내용을 보충해보도록 하겠습니다.

---

### Reflection

우선, 리플렉션에 대해 간단하게 짚고 넘어가보겠습니다.  

리플렉션은 자바 코드 자체를 추상화하여 구체적인 객체정보를 알지 못하더라도 클래스정보들을 접근 할 수 있도록 하는 자바 API 입니다. 이를 통해 `동적 객체 선언`, `동적 메서드 호출` 기능을 사용 할 수 있는데, Spring에서는 `DI`, `Proxy`등에서 리플렉션이 사용됩니다. 

```java
    // Generic
    Sample sample = new Sample();
    sample.doSomething();

    // Reflection
    Class sampleClazz = Class.forName("Sample");
    Method sampleMethod = sampleClazz.getMethod("doSomething");
    sampleMethod.invoke();
```

물론 `Effective java` 책에서는 리플렉션 사용을 권장하고있지는 않습니다.  

> `아이템 65) 리플렉션보다는 인터페이스를 사용하라`  
>   
> - 컴파일타임 타입 검사가 주는 이점을 하나도 누릴 수 없다.  
> - 리플렉션을 이용하면 코드가 지저분하고 장황해진다.
> - 성능이 떨어진다.
> - 리플렉션은 아주 제한된 형태로만 사용해야 그 단점을 피하고 이점만 취할 수 있다.
> - 리플렉션은 인스턴스 생성에만 쓰고, 이렇게 만든 인스턴스는 인터페이스나 상위 클래스로 참조해 사용하자.

특수한 시스템을 만들때 사용할 수 있는 강력한 기능이지만, 아무래도 컴파일 타임에서 명확한 `객체`를 사용하지 못하다보니 잘못쓰면 독이되기 쉬운 API 입니다.  

이러한점을 미리 알아두고 `Dynamic proxy`에서는 리플렉션을 어떻게 사용하는지 알아보도록 하겠습니다.

---

### Reflection in Dynamic proxy

예제 코드를 통해 알아보도록 하겠습니다.

```java
interface Sample {
    void doSomething();
    String saySomething();
    String saySomething2();
}
```

```java
public class SampleImpl implements Sample {
    public void doSomething(){
        System.out.println("do-something");
    }

    public String saySomething(){
        return "something";
    }

    public String saySomething2(){
        return "something2";
    }
}
```

위와같이 구현된 코드에서 `SampleImpl` class에 일반적인 `Proxy`객체를 만들어 보도록 하겠습니다.

```java
public class SampleProxy implements Sample {
    Sample sample;

    public SampleProxy(Sample sample){
        this.sample = sample;
    }

    public void doSomething(){
        System.out.println("do-something in proxy");
        sample.doSomething();
    }

    public void saySomething(){
        sample.doSomething().toUpperCase();
    }

    public void saySomething2(){
        sample.doSomething2().toUpperCase();
    }
}
```

위와같이 일반적인 방법으로 Proxy 객체를 만들수 있지만, 인터페이스에 대해서 모든 메서드를 직접 구현해 줘야하고 동일한 액션이 있을경우 중복해서 만들어 줘야한다는 특징이 있습니다.

그렇다면 위 Proxy를 리플렉션을 이용한 `Dynamic proxy`를 통해 다시 만들어 보겠습니다.   

```java
public class SampleProxy implements InvocationHandler {
    Sample sample;

    public SampleProxy(Sample sample) {
        this.sample = sample;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        switch (method.getName()) {
            case "doSomething":
                System.out.println("do-something in proxy");
                return method.invoke(sample, args);
            case "saySomething":
            case "saySomething2":
                return ((String) method.invoke(sample, args)).toUpperCase();
            default:
                return method.invoke(sample, args);
        }
    }
}
```

`Dynamic proxy`는 `InvocationHandler`를 구현하여 사용 할 수 있습니다. Reflection api를 사용하는 `invoke` 메서드를 구현 함으로서 proxy 호출 역할을 하는것을 확인 하실 수 있습니다.

실제 프록시객체를 어떻게 수행시킬 수 있는지 보도록 하겠습니다.

```java
public class Main {
    public static void main(String[] args) {
        Sample sample = (Sample) Proxy.newProxyInstance(
                Main.class.getClassLoader(),
                new Class[]{Sample.class},
                new SampleProxy(new SampleImpl())
        );

        sample.doSomething();
        System.out.println("result : " + sample.saySomething());
        System.out.println("result : " + sample.saySomething2());
    }
}
```

```java
// 결과

do-something in proxy
do-something
result : SOMETHING
result : SOMETHING2

```
결과를 보시면 원하던대로 Proxy를 통해 잘 수행된것을 보실수 있습니다.

`Dynamic proxy`로 만들어진 객체는 (Proxy.newProxyInstance)는 최종적으로 `Sample` 인터페이스를 구현한 객체를 반환하게 됩니다.  

클라이언트는 Proxy가 없을때와 동일하게, `Sample` 인터페이스를 참조해 객체를 사용 할 수 있습니다. 이 점은 위 리플렉션에대해 알아보면서 주의사항을 잘 따르고 있음을 확인 하실 수 있습니다.

> - 리플렉션은 인스턴스 생성에만 쓰고, 이렇게 만든 인스턴스는 인터페이스나 상위 클래스로 참조해 사용하자.

---
### 참고문서

- 토비의 스프링 3.1
- Effective java 3판