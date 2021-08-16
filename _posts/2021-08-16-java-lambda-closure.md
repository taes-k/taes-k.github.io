---
layout: post
comments: true
title: Java 람다와 클로져
tags: [java, labda, closure]
---

### 클로져 (`Closure`)

아마 자바만 다루셨던 분이라면 `클로져`가 조금은 생소한 개념일 수 있습니다. 주로 자바스크립트와 같이 함수를 `일급객체`로 다루는 언어에서 중요하게 다루는 개념인데 간단하게 짚고 넘어가보도록 하겠습니다.  

```js
// ClosureDemo.js

function add10(num) {
  var innerFunc = function () { return num + 10 }
  return innerFunc
}

var call = add10(10)
console.log(call()) // 20
```

위 코드에서 주목해야할 점은 `var innerFunc = function () { return a + 10; }` 부분입니다. `innerFunc`함수에서 외부함수인 `add10` 함수의 파라미터인 `num`을 사용하고 있습니다. 

같은 함수내에서 사용되기 때문에 문제될 것이 없는것처럼 보이지만, `var call = add10(10)` 를 호출하는 시점에서 `call` 변수가 갖게되는것은 `innerFunc` 함수일 것 입니다.  
그렇다면 해당 함수를 호출하는 `call()`을 수행하게되면 `add10`의 함수는 이미 종료되었기때문에 `num` 파라미터를 어떻게 사용 할 수 있는지에 대해서 혼동이 오실 수 있습니다.

일반적인 프로그래밍 개념에서 보자면, 실행이 종료된 함수(`완료된 스택`)의 변수는 소멸되는것이 맞지만, `자바스크립트`의 함수는 자신이 선언되었을때의 환경(`Lexical environment`)를 기억해 외부에서 함수가 호출 되더라도 처음 함수가 생성됬을때의 환경을 참조하여 불러 올 수 있습니다. 이런 동작을 하는 함수를 `클로져 함수`라고 칭하기도 합니다.

---

### 자바의 클로져

위의 자바스크립트에서 `클로져`에 대해 알아보았는데, 이번에는 자바에서 어떻게 사용되는지 알아보도록 하겠습니다. 우선, 자바 자체는 함수를 `일급객체`로 다루지 않았지만 `Java8`부터 `함수형 인터페이스`와 `람다 표현식`이 나오면서 유사하게 사용이 가능해졌습니다.

위 자바스크립트 예제와 동일하게 사용된 자바 예제를 통해 알아보도록 하겠습니다.

```js
// ClosureDemo.java

public class ClosureDemo {
    public static void main(String[] args) {
        Closure closure = new Closure();
        var call = closure.add10(10);
        System.out.println(call.get()); // 20
    }

    public static class Closure{
        private Supplier<Integer> add10(int num){
            return () -> num+10;
        }
    }
}
```

자바스크립트에서는 `Lexical environment`라는 개념을 통해 이 처리가 가능했지만 자바에서는 이 개념이 존재하지 않습니다. 자바에서는 자바의 argument 전달 방식인 `Call by Value`와 같이 `클로져 함수`에서 사용되는 변수들을 복제해두고 사용하기 때문에 외부에서 함수가 호출 되더라도 초기에 함수가 생성됬을때 넘겨받은 변수값을 사용 할 수 있습니다.

---

### 자바 람다와 클로져

위와같은 코드는 주로 자바에서 `람다함수`를 사용할때 사용되게 되는데 알아보았다시피 일반적이지 않게 동작하기 때문에 내부적으로 어떻게 동작하는지 잘 인지하고 코드를 작성해주셔야 합니다. 

자바에서 내부함수에서 사용되는 외부 변수들은 `복사`해서 사용된다는 특성때문에 몇가지 룰이 존재합니다.  

우선 외부변수가 내부함수에서 변경 될 수 없습니다. 변경을 시도한다면 다음과같은 컴파일 에러가 발생합니다.

```java
// SampleClass.java

 private void doSomething(){
    String st = "origin";
    Supplier<String> supplier = () -> {
        st += "_changed";
        return st;
    };
}
```
> java: local variables referenced from a lambda expression must be final or effectively final

컴파일 오류메세지처럼 내부함수에서 사용되는 변수들은 `final`를 직접 지정해주거나, 키워드가 없더라도 `final`처럼 취급되어야 합니다. (`Java7` 까지는 익명함수에서 반드시 `final` 키워드가 필요했습니다) 만약 꼭 변경이 필요한경우 `atomic` 타입으로 변경하여 사용 될 수 있습니다. 

이와같은 룰이 있는 이유는, 위에서 알아보았다시피 내부함수에서는 외부변수의 `복제본`을 사용하기도 하고 해당 함수가 어떤 `쓰레드`에서 동작할지 알 수 없어 쓰레드간 경쟁에 노출될 수 있기 때문입니다.  

'복제본을 쓰는데 `쓰레드`간 경쟁이 왜 일어나지?' 라고 생각하시는 분들이 있으실텐데 다음 예제를 보여드리도록 하겠습니다.

```java
// SampleClass.java

public static class Sample{
    private void doSomething(){
        SampleObj obj = new SampleObj();

        Supplier<SampleObj> supplier = () -> {
            obj.count++;
            return obj;
        };
    }
}

public static class SampleObj{
    public int count = 0;
    public boolean flag = true;
}
```

자바에서 객체 파라미터가 복제될때에는 참조값을 담은 레퍼런스를 복제하기때문에 람다식에서 사용되는 객체는 `공유`될 수 있습니다.
(참고 링크 : [자바는 Call by Value 로 동작한다](https://taes-k.github.io/2021/01/23/java-call-by-value/)) 

실제로 위 코드는 `obj` 객체자체가 변경되는것은 아니기때문에 람다식에서 `obj.count`값이 변경 될 수 있고 `async`로 동작하게 된다면 다른 쓰레드에의해 변경이 일어나 쓰레드경쟁이 발생 할 수 있습니다.

---

### 자바 람다를 사용할때 

최종으로 정리해보자면 위에서 알아본 이유들로 인해 람다식에서 사용되는 객체의경우 가능하다면 다음과같은 조건하에 사용하는것이 좋습니다. 

- final 선언하여 사용하기
- 불변객체를 사용하기

```java
// SampleClass.java

public static class Sample{
    private void doSomething(){
        final SampleObj obj = new SampleObj(0, true);

        Supplier<SampleObj> supplier = () -> {
            int count = obj.getCount + 1;
            boolean flag = count == 100 ? false, true;
            return new SampleObj(count, flag);
        };
    }
}

public static class SampleObj{ // immutable object
    private final int count = 0;
    private final boolean flag = true;

    public SampleObj(int count, boolean flag){
        this.count = count;
        this.flag = flag;
    }

    public getCount(){
        return count;
    }

    public getFlag(){
        return flag;
    }
}
```

물론 위와같은점들을 지키지 않아도 문제가 생길수 있는 지점을 인지하고 개발한다면 컴파일 에러 없이 사용이 가능하지만, 팀에서 함께 사용되는 코드에서는 다른 팀원혹은 추후에 코드를 보게될 분들을 위해 위같은 룰을 지켜서 코드를 작성하는것이 좋을것 입니다.
