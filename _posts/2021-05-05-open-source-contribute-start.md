---
layout: post
comments: true
title: opensource contribute 시작하기
tags: [opensource]
---

### Opensource contribute

`오픈소스`에 `기여` 하는 일은 거창하고, 어렵고, 해당 소스에 대해서 많은 이해도가 있어야만 `시도 해 볼 수 있다` 라고 생각하고 있었습니다.

사실, 오픈소스를 사용하면서 버그 같은것(?)을 발견한적도 있고 기능이 내가 생각한대로 동작하지 않을때, 오픈소스를 수정하거나 이슈를 생성할 생각을 하기 보다는 단순히 해당 문제들을 회피할 수 있는 코드를 작성해 왔습니다.

그러던 중, 포스팅에서 아래와 같은 내용을 읽게되었습니다.

> `정리하자면 시작하는 팁은 그냥 일단 시작하라는 것이다. 리젝 당하면 어떻고 리뷰에서 까이면 어떤가 맨날 생각만 하고 안 하는 것보다 일단 하는 게 무조건 좋은 것 같다.`
>  
> https://geminikim.medium.com/%EB%88%84%EA%B5%AC%EB%82%98-%ED%95%A0-%EC%88%98-%EC%9E%88%EB%8A%94-%EC%98%A4%ED%94%88%EC%86%8C%EC%8A%A4-%EC%BD%94%EB%93%9C-%EA%B8%B0%EC%97%AC-2ce4b0138b1e

위 내용을 읽고 동기부여가 된 김에 작은것부터 오픈소스 기여를 시작을 해보려고 합니다.

---

### Lombok contribute 시도

Java 개발을 하면서 대부분 사용하고 계실 `Lombok`을 사용하면서 저는 아래와 같은 이슈를 발견했었습니다.

```java
@Getter
@Builder
public class Sample
{
    private String name = "default";
    private Integer age;
}
```

```java
@Builder
public class SampleTest
{
    @Test
    public void test()
    {
        Sample sample = Sample.builder().age(10).build();

        // Test fail
        Assertions.equals("default", sample.getName());
    }
}
```

`Lombok @Builder` 사용시 멤버변수가 설정해둔 값으로 초기화되지 않는것에 대해 '의도한바가 맞는가?' 에 대한 의문이 들었고 직접 수정하기위해 시도해보려 했습니다.

(해당 내용은 포스팅을 통해 작성하기도 했던 내용입니다 - https://taes-k.github.io/2020/12/03/lombok-builder/)

우선 위코드를 `deLombok`하면 아래와 같은 코드가 됩니다.

```java
// DELOMBOK

public class Sample
{
    private String name = "default";
    private Integer age;

    Sample(String name, Integer age)
    {
        this.name = name;
        this.age = age;
    }

    public static SampleBuilder builder()
    {
        return new SampleBuilder();
    }

    public static class SampleBuilder
    {
        private String name;
        private Integer age;

        SampleBuilder()
        {
        }

        public SampleBuilder name(String name)
        {
            this.name = name;
            return this;
        }

        public SampleBuilder age(Integer age)
        {
            this.age = age;
            return this;
        }

        public Sample build()
        {
            return new Sample(name, age);
        }

        public String toString()
        {
            return "Sample.SampleBuilder(name=" + this.name + ", age=" + this.age + ")";
        }
    }
}
```

`SampleBuilder` class를 만들면서 기존에 설정한 초기화 값이 안넘어가는것을 확인 할 수 있는데, 이것은 의도한 바와 다르게 설계되었다는 생각에 오픈소스 기여의 첫 단추인 `issue`를 생성하려고 했습니다.

![1]({{ site.images | relative_url }}/posts/2021-05-05-open-source-contribute-start/1.png)   

`new feature issue` 를 생성하려고 보니, 우선 위키를 참고해 주라는 메인 컨트리뷰터의 간곡한 부탁이 있어서 먼저 확인해보았습니다.

![2]({{ site.images | relative_url }}/posts/2021-05-05-open-source-contribute-start/2.png)

위키 내용을 찾다보니 제가 가지고 있던 의문에 대해 별도로 설정 할 수 있는 기능이 제공 되고 있었습니다.

```java
@Builder
public class Sample
{
    @Builder.Default
    private String name = "default";
    private Integer age;
}
```


```java
// DELOMBOK

public class Sample
{
    @Builder.Default
    private String name = "default";
    private Integer age;

    Sample(String name, Integer age)
    {
        this.name = name;
        this.age = age;
    }

    private static String $default$name()
    {
        return "default";
    }

    public static SampleBuilder builder()
    {
        return new SampleBuilder();
    }

    public static class SampleBuilder
    {
        private String name$value;
        private boolean name$set;
        private Integer age;

        SampleBuilder()
        {
        }

        public SampleBuilder name(String name)
        {
            this.name$value = name;
            this.name$set = true;
            return this;
        }

        public SampleBuilder age(Integer age)
        {
            this.age = age;
            return this;
        }

        public Sample build()
        {
            String name$value = this.name$value;
            if (!this.name$set)
            {
                name$value = Sample.$default$name();
            }
            return new Sample(name$value, age);
        }

        public String toString()
        {
            return "Sample.SampleBuilder(name$value=" + this.name$value + ", age=" + this.age + ")";
        }
    }
}
```

여기까지 `오픈소스에 기여` 실제로 한것은 하나도 없지만, `그냥 일단 시작해` 봄으로서, 벌써 한가지 장점을 발견했다고 생각합니다.  

의문을 가지고 있던 오픈소스에 대해 설계자의 의도와 미리 구현해둔 대응책을 좀더 세밀히 확인 할 수 있다는 점을 깨달았습니다.

---

### Lombok plugin contribute 시도

`Lombok`을 사용하면서 한가지 더 버그라고 생각하던 기능이 있습니다. 

```java
@UtilityClass
public class UtilClazz
{
  public static void method()
  {
     InnerClazz obj = new InnerClazz(); 
  }

  private class InnerClazz
  {

  }
}
```

`Lombok @UtilityClass` 사용시 `Method`와 `Inner class`에 `static`을 붙여주게되는데, 위 코드의 경우 `Java8`에서는 정상작동 하지만 `Java11`이상에서는 정상동작 하지 않습니다. 

(해당 내용 또한 포스팅을 통해 작성했던 내용입니다 - https://taes-k.github.io/2020/11/30/error-note-lombok-utility-class-%EB%B3%B5%EC%82%AC%EB%B3%B8/)

그렇지만, 위 코드를 `Delombok` 해보면 `Java8`, `Java11`에서 동일하게 정상적으로 소스가 변경됩니다.

```java
// DELOMBOK

public final class UtilClazz
{
    private UtilClazz()
    {
        throw new UnsupportedOperationException("This is a utility class and cannot be instantiated");
    }

    public static void method()
    {
        InnerClazz obj = new InnerClazz();
    }

    private static class InnerClazz
    {

    }
}
```

위의 내용으로 보았을때, `Lombok` 코드 자체에는 문제가 없으나 `Intellij plugin`에서 오류가 있을것이라 판단되어 해당내용에 대해 이슈를 작성해두었습니다.

- https://github.com/mplushnikov/lombok-intellij-plugin/issues/1037

포스팅을 썼으면, 해당 이슈를 해결하는것 까지 내용이 정리되었으면 좋았겠지만, 아직 코드에대해 이해도가 부족해 아직은 직접 코드를 수정해 `PR`을 올리지는 못하고 스터디를 진행하고 있는 상태입니다.

이러한 `시작`을 해보면서 또 느낀 장점 한가지는, `issue`를 제기 함으로서 내가 해결은 못하더라도 다른 기여자분들이 `issue`를 확인후 `PR`을 올려 준다면 코드를 이해하는데 큰 도움이 되고 오픈소스에 기여하는 커다란 한 발자국이 될 수 있을것이라는 생각이 들었습니다.

---

### `일단 시작해보자`

사실 너무나도 막막한 `오픈소스 기여` 이지만, `일단 시작해보자`는 생각으로 내가 사용하고 있는 오픈소스에 대해 발견했던 `bug`, `feature` 등을 `issue`로 등록해보면서 큰 한발걸음을 시작해보면 좋을것 같습니다.

