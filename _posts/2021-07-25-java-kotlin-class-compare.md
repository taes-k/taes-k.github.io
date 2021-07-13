---
layout: post
comments: true
title: Java vs Kotlin 클래스, 인터페이스 비교
tags: [kotlin]
---

### 자바와 코틀린의 클래스, 인터페이스 비교

코틀린에서는 자바와 같이 클래스, 인터페이스를 사용 할 수 있지만, 그 사용방법에는 차이가 존재합니다. 이번 포스팅을 통해 코틀린에서 클래스와 인터페이스가 어떻게 정의되고 사용되는지 자바와의 비교를 통해 알아보도록 하겠습니다.

---

### 클래스

가장먼저 클래스를 만들고 사용할때의 차이를 보도록 하겠습니다.

```java
// Sample.java

public final class Sample {
    private String name;
    private String nickName;

    public Sample(String name, String nickName) {
        this.name = name;
        this.nickName = nickName;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getNickName() {
        return nickName;
    }

    public void setNickName(String nickName) {
        this.nickName = nickName;
    }
}
```

```java
// SampleKt.kt

class SampleKt(var name:String, var nickName:String?) {
}
```

- 코틀린 클래스의 default modifier는  `public final` 입니다.
  - 기본적으로 클래스 상속이 닫혀있습니다.  

- 위 예시와같이 매우 간단하게 클래스, 프로퍼티, 생성자를 함께 만들어 줄 수도 있으나 원하는대로 생성자를 추가 해 줄 수도 있습니다.  
  다만, 코틀린에서는 `객체 생성시에 파라미터 시그니쳐에 default값을 직접 명시 가능` 하기 때문에 설계의도상 생성자를 여러개 만들지 않는것을 권장하고 있습니다.

  ```java
  class SampleKt(var name:String, var nickName:String? = null) {
  }
  ```
  ```java
  var sample1 = SampleKt(name = "테스터1`", nickName = "별명1")
  var sample2 = SampleKt(name = "테스터2`", nickName = null)
  var sample3 = SampleKt(name = "테스터3")
  ```
  

---

### 인터페이스

인터페이스 역시 자바와 비슷하게 사용할수있지만 약간의 다른점이 존재합니다.

```java
// SampleInterface.java

public interface SampleInterface {
    // Interface에서 프로퍼티 생성 불가능
    // String name;

    void func0();
    default void func1(){
        System.out.println("Click SampleInterface");
    }
}
```

```java
// SampleInterfaceKt.java

interface SampleInterfaceKt {
    var name: String
    fun func0()
    fun func1() = println("Click SampleInterfaceKt")
}
```

- `default` 키워드 없이 default 메서드를 구현해야 합니다.
- 코틀린 인터페이스는 프로퍼티를 가질 수 있습니다.
  - Interface에서 구현된 프로퍼티 선언에는 실제 필드나 게터/세터 선언이 포함되어 있지 않기때문에, 상속받는 클래스에서 추상프로퍼티를 구현해줘야 합니다.
    ```java
    // SampleImplementsKt.kt

    class SampleImplementsKt(override var name: String) :SampleInterfaceKt {
        ...
    }

    class SampleImplementsKt2() :SampleInterfaceKt {
        override var name: String 
            get() = name
            set(_name){name = _name} 

        ...
    }
    ```

---

### 상속

이번에는 코틀린에서 `상속`을 처리하는 코드들을 알아보도록 하겠습니다.

```java
// SampleImplements.java

public class SampleImplements implements SampleInterface {
    // @Override
    public void func0() {
        System.out.println("This is fun0 in SampleImplements");
    }

    // @Override
    public void func1() {
        System.out.println("This is fun1 in SampleImplements");
    }
}
```

```java
// SampleImplementsKt.kt

class SampleImplementsKt(override var name: String) :SampleInterfaceKt {
    override fun func0() = println("This is fun0 in SampleImplementsKt")
    override fun func1() = println("This is fun1 in SampleImplementsKt")
}
```

- Java 에서는 `@Override`를 명시하는것이 옵션인 반면, `Kotlin`에서는 `override` 상속제어자를 필수로 명시해 주어야 합니다.

인터페이스 기본적으로 열린상태 (`open`) 이기 때문에 별도의 설정없이 상속하여 사용이 가능합니다. 하지만, 위 클래스에서 알아보았듯이 `Kotlin`에서의 default modifier는 `public final`이기때문에 상속이 불가능합니다. 

그렇기 때문에 상속의 대상이 되는 크래스들은 `열린 클래스`와 `열린 메서드`를 만들어 줘야 합니다.

```java
// SampleImplementsKt.kt

open class SampleImplementsKt(override var name: String) :SampleInterfaceKt {
    override fun func0() = println("This is fun0 in SampleImplementsKt")
    final override fun func1() = println("This is fun1 in SampleImplementsKt")
    open fun func2() = println("This is fub2 in SampleImplementsKt")
}
```

```java
// SampleKt.kt

class SampleExtendsKt(name: String) : SampleImplementsKt(name){
    override fun func0() = println("This is fun0 in SampleExtendsKt")
    
    // 닫힌 메서드라서 메서드 상속 불가
    // override fun func1() = println("This is fun1 in SampleImplementsKt")
    override final fun func2() = println("This is fub2 in SampleExtendsKt")
}
```

- 클래스 상속을 위해서는 `open class`를 명시하여 열린 클래스로 생성해줘야 합니다.
- 클래스와 마찬가지로 function 또한 기본적으로 닫힌 메서드로 생성되기때문에 메서드 상속을 위해서는 `open fun`을 명시해야 합니다.
- `override` 가 명시된 메서드는 기본적으로 `open`이 되어 재상속이 가능한 상태입니다. 재상속이 불가능하게 하려면 `final override`를 직접적으로 명시해 줘야 합니다.
- 추상클래스를 만들때에도, 추상메서드로 선언하지 않는다면 `닫힌 메서드`가 기본 설정입니다.

  ```java 
  abstract class SampleAbstract {
  abstract fun fun0()
  fun fun1(){} // public final
  }
  ```
- 인터페이스와 다르게, 클래스를 상속시 추가로 다른점이라고하면 부모 클래스를 선언시 뒤에 괄호 `()`가 들어간다는 점 인데, 디폴트 생성자를 만들어주는 `Kotlin`의 특성때문이라고 이해 할 수 있습니다.

---

### 접근제어

자바에서는 알다시피 4개의 접근제어자를 제공합니다.
- public
- default
- protected
- private

Kotlin도 동일하게 4개의 접근제어자를 제공하지만, 자바의 기본 접근제어자인 `default` 대신 `internal` 제어자를 제공하며 `public`이 기본 접근제어자입니다.

또한 `패키지`단위가 아닌 `모듈`단위 접근제어를 한다는점에서 차이가 있습니다.

---

### 내부클래스, 중첩클래스, 봉인클래스

자바에서는 일반적으로 내부 클래스로 사용될때 중첩클래스를 선언합니다.  
`Kotlin`에서도 마찬가지로 중첩클래스를 생성 할 수 있으나 `바깥클래스 인스턴스 접근권한이 없다` 는 차이가 있습니다.

```java
// Sample.java

public final class Sample {
    private String name;

    public class InnerSample{
        private String outerName = Sample.this.name;
    }

    static public class NestedSample{
        // 접근 불가
        // private String outerName = Sample.this.name;
    }
}
```


```java
// SampleKt.java

class SampleKt(var name:String) {
    inner class InnerSample(){
        var outerName = this@SampleKt.name
    }

    class NestedSample(var age:Int){
        // 접근 불가
        // var outerName = this@SampleKt.name
    }
}
```

- `Kotlin`에서 선언한 `중첩 클래스`는 기본적으로 `NestedClass` 로서, `static` modfier가 붙여진다고 볼 수 있습니다.
- `내부 클래스`로 사용하기 위해서는 `inner` modifer를 명시해 주어야 합니다.

클래스들을 묶어주기위해서라면, `Java15`부터 제공하는 기능인 봉인클래스 (`Sealed`) 또한 `Kotlin` 에서 동일한 기능을 제공하고 있습니다.

```java
// SamplesKt.kt

sealed class SamplesKt {
    class SampleA{}
    class SampleB{}
}
```

---

### 비공개 생성자

Java에서는 `UtliityClass` 를 만들때 비공개 생성자로 클래스를 선언하는 경우가 일반적입니다. 하지만, `Kotlin`에서는 비공개 생성자를 만드는것은 일반적이지 않은 상황인데 그 이유를 알아보도록 하겠습니다.

```java
// SampleUtils.java

public class SampleUtils {
    private SampleUtils() {
        throw new IllegalStateException("Utility class");
    }

    static public String getSampleA(){
        return "SampleA";
    }
    
    static public String getSampleB(){
        return "SampleB";
    }
}
```


```java
// SampleUtilsKt.kt


fun getSampleA(): String = "SampleA"
fun getSampleB(): String = "SampleB"
```

- 애초에 `Kotlin`에서는 Class 외부에 `function`을 정의 할 수 있기때문에 비공개 class 생성자를 만들어 접근을 막는 일을 하지 않을 수 있습니다.

---
### data 클래스

Java 에서는 데이터를 관리하는 객체를 사용하기위해 일반적으로 Lombok `@Data`를 활용해 사용하고 있습니다. 

```java
@Data
public class SampleData{
    private final String name;
    private final String nickName;
}
```

`@Data`는 프로퍼티들의 `getter`,`setter`,`hashCode`,`equals`,`tostring`를 만들어주며 `Java16`부터는 `record` 타입이 추가되어 명시적으로 데이터 타입객체 분리해 사용 할 수 있습니다.

```java
public record SampleData(String name, String nickName){

}
```

위와 비슷하게 `Kotlin` 에서도 `Data class`를 지원합니다.

```java
data class SampleData(val name: String, val nickName: String) {
}
```

---

### 클래스 위임

클래스 상속을 없애거나, 상속을 허용하지 않는 클래스를 확장하기 위한 방법으로 `Composition`을 사용하는 경우가 많습니다.

```java
// SampleDelegate.java

public class SampleDelegate<E> implements Collection<E> {
    private final List innerList = new ArrayList<>();

    public int size() {return innerList.size();}
    public boolean isEmpty() {return innerList.isEmpty();}
    public boolean contains(Object o) {return innerList.contains(o);}
    public Iterator<E> iterator() {return innerList.iterator();}
    public Object[] toArray() {return innerList.toArray();}
    ...
}
```

```java
// SampleDelegateKt.kt

class SampleDelegateKt<E>(innerList: Collection<E> = ArrayList<E>()) : Collection<E> by innerList{
}
```

- `Kotlin` 에서는 클래스 위임을 통해 간단하게 `composition` 객체를 생성 할 수 있습니다.

---

### 싱글턴, 동반객체, 무명객체

`Kotlin`에서는 `Object` 키워드를 통해 싱글턴 객체를 쉽게 만들 수 있습니다.

```java
// Singleton.java

public enum Singleton {
    INSTANCE;
    private final List innerList = new ArrayList();

    private List getList(){
        return innerList;
    }
}
```

```java
// SingletonKt.kt

object SingletonKt {
    val innerList: List<String> = ArrayList()
}
```

`Koltin`에서는 정적멤버가 없습니다. 즉, `static`키워드를 지원하지 않습니다.  
`Kotlin`은 최상위 함수 선언 혹은 객체선언을 통해 정적 메서드 혹은 정적필드를 대신합니다.  

 최상위 함수선언의 경우 대부분의 정적메서드를 대체할수 있지만, 클래스멤버의 `private 생성자`에 접근하는것은 불가능해 동반객체를 사용하기도 합니다. 

```java
// Companion.java

public class Companion {
    private String name;
    
    private Companion(String name){
        this.name = name;
    }
    static public Companion getCompanion(String name){
        return new Companion(name);
    }
}
```

```java
// CompanionKt.kt 

class CompanionKt private constructor(private val name:String){
    companion object {
        fun getCompanionKt(name:String) = {
            CompanionKt("name")
        };
    }
}
```

또한 동반객체를통해 상속 및 확장 할 수도 있습니다.

```java
// CompanionKt.kt

class CompanionKt private constructor(private val name:String){
    companion object : SampleInterfaceKt{
        fun getCompanionKt(name:String) = {
            CompanionKt("name")
        };

        override var name: String
            get() = name
            set(value) {name = value}

        override fun func0() {
           println("func0 in CompanionKt companion")
        }
    }
}
```

`Kotlin`에서는 익명클래스 또한 `object`를 활용해 조금더 깔끔하게 구현 할 수 있습니다.

```java
// Anonymous.java

public class Anonymous {
    public void aa(){
        SampleInterface a = new SampleInterface(){
            @Override
            public void func0() {
                System.out.println("func0 in anonymouse");
            }

            @Override
            public void func1() {
                SampleInterface.super.func1();
            }
        };
    }
}
```

```java
// AnonymousKt.kt

class AnonymousKt {
    fun getSample() {
        object : SampleInterfaceKt {
            override var name: String
                get() = name
                set(value) {name = value}

            override fun func0() = println("func0 in anonymouse")
        }
    }
}

```
---

### 마무리

여기까지 `Kotlin`에서 `Class`, `Interface`, `Instance`를 사용하는 방법에 대해서 `Java`와의 비교를 통해 알아보았습니다. 대부분의 사용 방법은 큰 차이는 없지만 `Kotlin`의 설계 의도에 맞게끔 `default modifier`가 변경된 부분들이 많기 때문에 이러한 부분들을 주의하시면 금방 `Kotlin`에도 적응 하실 수 있을거라 생각됩니다.

---

### Reference

- 코틀린인액션 (https://search.shopping.naver.com/search/all?query=kotlin%20in%20action&frm=NVSHATC&prevQuery=%EC%BD%94%ED%8B%80%EB%A6%B0%20%EC%9D%B8%20%EC%95%A1%EC%85%98)



