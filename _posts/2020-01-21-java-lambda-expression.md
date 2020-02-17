---
layout: post
comments: true
title: 자바 람다식
tags: [java]
---

(해당 포스팅은 자바 프로그래밍에는 익숙하지만, java8부터 지원하는 `함수형 프로그래밍`에 대해서는 미숙한 독자분들을 대상으로 합니다.)

---

### Lambda Expression
java8 부터는 `람다표현식(람다식)`을 지원합니다.  
아마도 java8 이전버전을 사용하시던 분이라면 `람다식`이 익숙하지 않으실 수 있는데 사실 람다식, 또는 `람다함수`는 java8에서 처음 나온 개념이 아닙니다.  
  
람다식은 수학 `람다 계산법`에서 사용된 식으로 프로그래밍 언어로서는 `LISP`에 첫 도입이 되어 이후 C#, Scala, Python 등 현재는 대부분의 언어에서 지원하고 있습니다.    

람다식은 `익명함수`를 지칭하는 용어로서 말 그대로 선언되어있지 않은 함수를 자체적으로 함수화한 표현식을 뜻합니다. 설명으로만 풀어 쓰자니 이해하기 힘드실 것 같아 예제를 통해 알아보도록 하겠습니다.  

```java
public String concat(String message1, String message2){
    return message1+message2;
}
```

위에서 선언한 `concat` 메서드는 2개의 String parameter를 받아 붙여주는 메서드 입니다. 위의 메서드를 람다식으로 표현하자면 다음과 같습니다.  
  
```java
(a,b) -> a+b 

// (a,b) : parameter 선언
// -> : parameter와 body를 구분하는 구분자
// a+b : return 값
```

혹은 다음과 같이 표현이 가능합니다.  
  
```java
(a,b) -> {return a+b;}

// (a,b) : parameter 선언
// -> : parameter와 body를 구분하는 구분자
// {return a+b;} : 함수 body
```

위의 예제로 알 수 있듯이 람다식의 가장 큰 장점은 코드가 매-우 간결해질 수 있다는 장점이 있습니다. 하지만 모든 경우에서 이러한 람다식을 사용 할 수 있는 것은 아닙니다.  

---

### Functional Interface

자바에서 람다식을 사용 가능한경우는, `Functional Interface (함수현 인터페이스)`로 선언되어있는 경우에서만 람다식으로 표현이 가능합니다.    

따라서 어떤 경우에 람다식 표현을 사용하는지 알아보기 위해 먼저, `Functional Interface (함수형 인터페이스)`에 대해서 알아보도록 하겠습니다.  
  
함수형 인터페이스는 하나의 메서드만을 갖는 Interface 입니다. 다음은 함수형 인터페이스 예제 코드입니다.  

```java
@FunctionalInterface
interface Concat {
    public String process(String message1, String message2);
}
```

위의 예제를 보시면 `concat` 인터페이스는 `process` 하나의 메서드만을 갖습니다.  
위와 같은 조건을 만족할 때, 해당 인터페이스는 함수형인터페이스의 조건에 충족한다고 할 수 있겠습니다.  

위와같은 인터페이스가 있을때, 일반적으로는 인터페이스를 구현한 구현체를 직접 작성하여 해당 메서드를 호출할수도 있겠지만, 다음과 같은 람다식을 통해서 표현이 가능해집니다.  
  
```java
public static void main(String[] args){
   Concat concatString = (m1, m2) -> m1 + m2;
   System.out.println(concatString.process("하나","둘"));
}
```

해당 익명함수를 사용하게 되면 원하는 상황적 옵션에 따라서 결과값을 다르게 도출이 가능해집니다.  

```java
public static void main(String[] args){
   Concat concatString = (m1, m2) -> m1 + m2;
   System.out.println(concatString.process("하나","둘"));

   Concat concatSplitString = (m1, m2) -> m1 + "," + m2;
   System.out.println(concatSplitString.process("하나","둘"));
}
```

위의 함수형 인터페이스를 메서드화시키면 다음과 같이 `함수형 프로그래밍`답게 코드를 구현 할 수 있습니다.

```java
public static void main(String[] args){
   concat("하나", "둘", (m1, m2) -> m1 + m2);
   concat("하나", "둘", (m1, m2) -> m1 + "," + m2);
}

public static void concat(String m1, string m2, Concat func){
   System.out.println(concatString.process(m1,m2));
}

```

---

#### java.util.function

java에서는 람다식을 더 편하게 사용 할 수 있도록 위와같이 별도의 Functional Interface를 선언하지 않고 익명함수를 사용할수 있도록 `java.util.function` 내장 함수형 인터페이스를 제공하고 있습니다.  

![1]({{ site.images | relative_url }}/posts/2020-01-21-java-lambda-expression/1.png)  

위처럼 굉장히 많은 interface들을 제공하고 있는데, 그중에서 많이 사용되는 interface들 몇 개만 추려서 알아보도록 하겠습니다.  

***-Functions\<T,R\>***  

```java
@FunctionalInterface
public interface Function<T, R> {
  
  R apply(T t);

  default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
    Objects.requireNonNull(before);
    return (V v) -> apply(before.apply(v));
  }

  default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
    Objects.requireNonNull(after);
    return (T t) -> after.apply(apply(t));
  }

  static <T> Function<T, T> identity() {
    return t -> t;
  }
}
```

위의 `Function<T,R>` 인터페이스를 보시면 위에서 말한 것과 다르게 여러개의 메서드를 가지고 있어 의아해하실 수도 있는데, 위의 인터페이스를 통해 `Functional Interface`는 하나의 `추상 메서드`를 가진다라고 재정의할수 있겠습니다.  

위의 `Function<T,R>` 인터페이스는 하나의 제네릭(T) 타입의 파라미터를 가지며, 제니릭(R) 타입의 결과값을 리턴하는 함수형 인터페이스입니다.  

위와같이 1 pram을 받아 return값이 있을때 사용 할 수 있어 일반적으로 가장 많이 사용되는 함수형 인터페이스이며 compose, andThen의 미리 정의된 메서드를 통하여 함수의 연계가 가능함을 알 수 있습니다.  

***-Consumer\<T\>***  

```java
@FunctionalInterface
public interface Consumer<T> {

    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

다음 `Consume<T>` 인터페이스는 하나의 제네릭(T) 타입의 파라미터를 가지며 return 하지 않는 함수형 인터페이스입니다.  
  
이름에서도 알 수 있듯이 파라미터를 '소비'하고 return 값이 없는 void 상태일 때 사용 할 수 있습니다.

***-BiConsumer\<T,U\>***  

```java
@FunctionalInterface
public interface BiConsumer<T, U> {

    void accept(T t, U u);

    default BiConsumer<T, U> andThen(BiConsumer<? super T, ? super U> after) {
        Objects.requireNonNull(after);

        return (l, r) -> {
            accept(l, r);
            after.accept(l, r);
        };
    }
}
```

다음 `BiConsume<T>` 인터페이스는 두 개의 제네릭(T,U) 타입의 파라미터를 가지며 return 하지 않는 함수형 인터페이스입니다.  
  
이름에서도 알 수 있듯이 두 개의 파라미터를 '소비'하고 return 값이 없는 void 상태일 때 사용 할 수 있습니다.

***-IntConsumer***  

```java
@FunctionalInterface
public interface IntConsumer {

    void accept(int value);

    default IntConsumer andThen(IntConsumer after) {
        Objects.requireNonNull(after);
        return (int t) -> { accept(t); after.accept(t); };
    }
}
```

다음 `IntConsume<T>` 인터페이스는 하나의 int 타입의 파라미터를 가지며 return 하지 않는 함수형 인터페이스입니다.  
  
위의 다른 함수형 인터페이스들과는 다르게 제네릭 타입을 사용하지 않고 정해진 type의 parameter를 사용하는 interface도 지원함을 알 수 있습니다.


---

#### Runnable

사실 이미 자바 프로그래밍을 하시면서 위에서 언급한 Functional Interface를 사용할 때보다도, 쓰레드 프로그래밍을 하시면서 `Runnable`을 사용할 때 람다식을 직/간접적으로 많이 사용하셨을것이라 생각됩니다.

다음은 Runnable을 사용한 예제코드입니다.  

```java
public static void main(String[] args){
    Runnable r = new Runnable()
    {
        @Override
        public void run()
        {
            System.out.println("Runnable")
        }
    };

    for(int i=0; i<10; i++){
        Thread t = new Thread(r); 
        t.start
    }
}
```

위의 run 구현체 메서드를 직접 작성 해 준 코드는 람다식을 사용하여 다음과 같이 표현 할 수 있습니다.  

```java
public static void main(String[] args){
    Runnable r = ()->{System.out.println("Runnable")};

    for(int i=0; i<10; i++){
        Thread t = new Thread(r); 
        t.start
    }
}
```

위 `Runnable`을 구현할 때 람다식으로 표현이 가능한 것으로 보아 아마도 `Runnable`은 Functional Interface임을 눈치채셨을 것 같습니다.  

아래는 `java.lang` -> `Runnable interface` 코드입니다.  

```java
@FunctionalInterface
public interface Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see     java.lang.Thread#run()
     */
    public abstract void run();
}
```

위 코드를 보시면 알수 있다시피, `Runnable` 역시 Functional Interface로 구현된 객체임을 알 수 있습니다.  

---
#### Stream

자바8에서는 람다식과 함께 `Stream`이라는 새로운 기능을 제공합니다.   
'람다식에 대해서 알아보다가 갑자기 왜 Stream?' 이라고 생각하시는 분도 계시겠지만, 함께 나온 형제답게 람다식의 활용도가 높을 때 중 하나는 바로 Stream을 사용할 때입니다.  

`Stream`에 대한 설명은 이곳에서 하지 않겠습니다. 다만 Stream에서 람다식에 어떻게 활용되는지 알아보도록 하겠습니다.  

다음은 Stream 예제입니다.

```java
List<String> numbers = Arrays.asList("one", "two", "three", "four", "five", "six", "seven");

numbers.stream()
    .map(s -> s.toUpperCase())
    .filter(s -> s.contains("O"))
    .forEach(s -> System.out.println(s));
```
위 예제코드는 String 리스트를 대문자로 치환하여 "O"가 포함된 String만을 출력하는 예제입니다.  
위의 구현에서 보시면 Stream 메서드에는 람다식 파라미터로써 코드가 작성되어 있는데요, 그렇다면 실제 Stream 내부 메서드가 어떻게 구현되어있는지 살펴보도록 하겠습니다.

```java
// jav.util.stream

public interface Stream<T> extends BaseStream<T, Stream<T>> {

    Stream<T> filter(Predicate<? super T> predicate);

    <R> Stream<R> map(Function<? super T, ? extends R> mapper);

    IntStream mapToInt(ToIntFunction<? super T> mapper);

    LongStream mapToLong(ToLongFunction<? super T> mapper);
```
위 `java.util.stream`에서 확인해보면 이번에도 역시나 직감 하셨듯이 Stream 메서드들 역시 Functional interface 들을 파라미터로 받는 메서드로 구현되어 있어 람다식을 사용함을 알 수 있습니다.  

---

### Method Reference

자, 이번에는 람다식을 한번더 간결하게 사용하는 방법에대해서 알아보도록 하겠습니다.  
다음 예제코드는 위의 Stream 예제코드와 똑같은 코드입니다.

```java
List<String> numbers = Arrays.asList("one", "two", "three", "four", "five", "six", "seven");

numbers.stream()
    .map(String::toUpperCase)
    .filter(s -> s.contains("O"))
    .forEach(System.out::println);
````

위의 예제코드를 보시면 이제는 느낌상으로 어떤 역할을 하는 코드인지 감은 잡히실거라 예상됩니다. 

> `"s -> s.toUpperCase()"` -> `"String::toUpperCase"`  
> `"s -> System.out.println(s)"` -> `"System.out::println"`

예제코드를 살펴보시면 좌측의 람다식이 우측의 `메서드 참조` 형태로 변경 되었습니다.  
`메서드 참조`는 람다식이 하나의 메서드만을 호출 할때 불필요한 변수들을 제거하여 간결하게 표현할수 있는 방법으로 다음과같이 표현 할 수 있습니다.  

> `"s -> ClassName.MethodName(s)"`  ->  `"ClassName::MethodName"`   
> `"s -> s.instanceMethodName()"`  -> `"ClassName::MethodName"`  
> `"(s1, s2) -> s1.instanceMethodName(s2)"` -> `"ClassName::MethodName"`

위와같은 형태로 `메서드참조`를 표현 할 수 있습니다.

---
### 마무리 

java8 부터 지원하는 람다식을 사용하면 코드가 간결해지고 로직을 한눈에 이해하기 쉬울 수 있으나 디버깅시 추적이 어렵다는 단점이 있을 수 있습니다. 또한 처음부터 함수형 프로그래밍으로써 설계를 하여 개발을 하지 않는 이상은 모든 코드에서 적용 할 수도 없기에 람다식으로 표현하기 좋은 상황을 잘 캐치하여, 잘 사용 하는것이 좋겠습니다.  