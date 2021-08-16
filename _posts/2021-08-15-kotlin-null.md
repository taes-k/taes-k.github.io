---
layout: post
comments: true
title: Java vs Kotlin null 처리 비교
tags: [kotlin, null]
---

### 자바의 `Null` 처리

아마도 자바를 다뤄보신 분 이라면, 객체의 `Null`을 다루는데 많은 코드라인을 소비하고 계실것이라 생각이 듭니다.  

물론 `Java8`부터는 `Optional`을 통해 `Null이 될 수 있는 객체`를 명시적으로 타입화 시키고 비교적 간편하게 처리를 해 줄 수 는 있지만, 설계 의도상 모든상황에서 `Optional`을 사용할 수 있는것도 아닙니다.

---

### 코틀린과 자바의 타입

우선 코틀린이 갖는 자바와의 가장 큰 차이는 `타입`이 기본적으로 `NotNull` 이라는 것 입니다.

```java
// java

public void printSample(String sample){ // sample -> Nullable
    System.out.println(sample);
}
```

```java
// kotlin

fun printSample(sample:String){ // sample -> NotNull
    println(sample);
}
```

만약 위 코틀린코드에서 `printSample(null)`을 호출하게 된다면 다음과같은 컴파일 오류가 발생하게 됩니다.

```
ERROR: Null can not be a value of a non-null type String
```

따라서 Null이 필요한 경우에는 `Null이 될 수 있음`을 명시 해주어야 합니다.

```java
// kotlin

fun printSample(sample:String?){ // sample -> Nullable
    println(sample);
}
```

---

### 코틀린과 자바의 Null 체크

자바에서 코들르 작성하다보면 `NPE(Null Point Exception)`을 방지하기위한 방어로직을 사용하는 경우가 많습니다. 자주 사용되다보니 몇줄되지 않는 코드지만, 피로감이 오기도 합니다. 코틀린에서는 간단하게 사용 할 수 있는 안전한 호출 연산자 (`?.`)를 제공합니다.

```java
// java

public void printString(String str){
    var capStr = null;
    if(str != null){
        capStr = str.toUpperCase();
    }
    System.out.println(capStr)
}
```
```java
// kotlin

fun printString(str:String?){
    val capStr: String? = str?.toUpperCase();
    println(capStr)
}
```

또한, 엘비스 연산자 (`?:`)를 통해서 좀 더 편하게 `Null`에 대한 처리를 직접적으로 다룰 수 있습니다.


```java
// java

public int getStringLength(String str){
    var len = 0;
    if(str != null){
        capStr = str.length();
    }
    return len;
}
```
```java
// kotlin

fun getStringLength(str:String?){
    val len = str?.length() ? : 0;
    return len;
}
```

만약 위와같은 처리또한 불편하고, 변수에 대한 `NotNull`이 로직적으로 보장되다고 하면 (`!!`) 단언문을 통해 간단하게 처리를 할 수도 있습니다.

```java
// kotlin
fun getStringLength(str:String?){
    return str!!.length();
}
```

---

### Reference

- 코틀린인액션 (https://search.shopping.naver.com/search/all?query=kotlin%20in%20action&frm=NVSHATC&prevQuery=%EC%BD%94%ED%8B%80%EB%A6%B0%20%EC%9D%B8%20%EC%95%A1%EC%85%98)



