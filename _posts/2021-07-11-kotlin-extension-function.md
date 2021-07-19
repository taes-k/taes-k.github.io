---
layout: post
comments: true
title: Kotlin Extensionfunction 주의할점
tags: [kotlin, extensionfunction, 확장함수]
---

### Kotlin function 상속 

Kotlin은 함수형 프로그래밍 언어인 만큼, 일급객체로 다루어지는 `function` 역시 상속하여 사용이 가능합니다.  

편리하게 사용 할 수 있지만 잘 알지못하면 (저포함...) 의도치 않은 결과를 얻게 될 수 있는데 아래 예제를 통해 정리해보도록 하겠습니다.

```kotlin
// Rectangle.kt
 
open class Rectangle {
    open fun getShape() = "Rectangle"
}
 
// Square.kt
 
class Square: Rectangle() {
    override fun getShape() = "Square"
}
 
// shapeTest.kt
 
class ShapeTest {
    @Test
    fun testShapes(){
        Assertions.assertEquals("Rectangle", Rectangle().getShape())     // 성공
        Assertions.assertEquals("Square", Square().getShape())     // 성공
    }
}
```

Class 에서 상속해서 function을 재정의하는 일반적인 방법으로 수행하면, 위와같이 우리가 예상하는대로 결과를 얻을 수 있을것 입니다.

Kotlin에서는 사용 편의성을 위해 클래스에서 상속하거나 Decorate 패턴을 사용하지 않고도 function을 재정의 하는 `ExtensionFunction(이하. 확장함수)` 기능을 제공합니다.

> 확장함수 (https://kotlinlang.org/docs/extensions.html#extensions-are-resolved-statically)


```kotlin
// Rectangle.kt
 
open class Rectangle {
    open fun getShape() = "Rectangle"
}
 
// Square.kt
 
class Square: Rectangle() {
    override fun getShape() = "Square"
}
 
// shapeTest.kt
 
class ShapeTest {
 
    fun Square.countSide() = 4
 
    @Test
    fun testShapes(){
        Assertions.assertEquals("Rectangle", Rectangle().getShape())     // 성공
        Assertions.assertEquals("Square", Square().getShape())     // 성공
        Assertions.assertEquals(4, Square().countSide())     // 성공
    }
}
```

위처럼 클래스에 정의되지 않은 function을 직접 추가해서 사용 할 수 있습니다.  
주로 직접 수정할 수 없는 라이브러리 클래스들을 확장해서 사용할때 필요한 기능을 편리하게 확장하는 용도로 사용 할 수 있습니다.  

그러나, 이때 주의할 사항이 있습니다.

```kotlin
// Rectangle.kt
 
open class Rectangle {
    open fun getShape() = "Rectangle"
}
 
// Square.kt
 
class Square: Rectangle() {
    fun countSide() = 1
}
 
// shapeTest.kt
 
class ShapeTest {
    fun Square().getShape() = "Square"
    fun Square().countSide() = 4
 
    @Test
    fun testShapes(){
        Assertions.assertEquals("Rectangle", Rectangle().getShape())    // 성공
        Assertions.assertEquals("Square", Square().getShape())     // 실패 
        Assertions.assertEquals(8, Square().getShape())     // 실패 
    }
}
```

위 실패 테스트 케이스들을 정리해보도록 하겠습니다.

- 상속 클래스에 멤버함수가 이미 존재하는 경우 멤버함수 수행이 우선시 됩니다.
- 클래스에 멤버함수가 이미 존재하는 경우 멤버함수 수행이 우선시 됩니다.



![1]({{ site.images | relative_url }}/posts/2021-07-11-kotlin-extension-function/1.png)   

![2]({{ site.images | relative_url }}/posts/2021-07-11-kotlin-extension-function/2.png)   

결국 두케이스 모두 확장함수를 사용하는 member에 이미 존재하는 함수를 확장하는것은 불가능하다는것입니다.  
즉, 확장함수를 사용하는것이 override의 개념이 아니라는것을 인지해야 합니다.  
단, 다음과같은 형태로는 사용 될 수 있습니다.

```kotlin
// Rectangle.kt
 
open class Rectangle {
}
 
// Square.kt
 
class Square: Rectangle() {
    fun countSide() = 1
}
 
// shapeTest.kt
 
class ShapeTest {
    fun Rectangle.getShape() = "Rectangle"
    fun Square().getShape() = "Square"
    fun Square.countSide(i: Int) = i
 
    @Test
    fun testShapes(){
        Assertions.assertEquals("Rectangle", Rectangle().getShape())    // 성공
        Assertions.assertEquals("Square", Square().getShape())     // 성공  
        Assertions.assertEquals(4, Square().countSide(4))     // 성공  
}
```

- 부모, 자식 클래스 모두에 동일한 멤버함수가 존재하지 않다면, 먼저정의된 부모클래스의 확장함수가 있더라도 확장함수간의 우선순위는 자식 클래스의 확장함수가 우선됩니다.
- 확장함수로 오버라이딩은 지원하지 않지만, 파라미터가 다른 오버로딩 확장에는 사용 될 수 있습니다.

확장함수를 사용할 때 또 한가지 주의사항으로는, 멤버변수 사용시 scope가 불명확 할 수 있다는 것 입니다.  
아래 예제를 통해 확인해보도록 하겠습니다.

```kotlin
// Rectangle.kt
 
open class Rectangle {
    val name = "Rectangle"
    val side = 4
    open fun getShape() = name
}
 
// Circle.kt
 
class Circle {
    val name = "Circle"
    val postFix = "_taes"
    val side = 1
 
    fun countSide() {
        println(side)
    }
    fun Rectangle.name() = "$name$postFix"
    fun Rectangle.countSide() {
        countSide()
    }
 
    fun callName() = Rectangle().name()
    fun callCountSide() = Rectangle().countSide()
}
 
// ShapeTest.kt
 
class ShapeTest {
    @Test
    fun testCircle(){
        println(Circle().callName())    // 성공  - (출력: Rectangle_taes)
        println(Circle().callCountSide())    // 에러 - Stackoverflow
    }
}
```

위 예제에서 문제가 되는 부분은 아래 코드 입니다.

```kotlin
// Circle.kt
 
class Circle {
    ...
    fun Rectangle.name() = "$name$postFix"
    fun Rectangle.countSide() {
        countSide()
    }
    ...
}
```

먼저, 확장함수에서 사용된 `$name`이 `Rectangle.name` 을 사용하게 될지 혹은 `Circle.name`을 사용하게 될지가 불명확한데, 출력결과(`Rectangle_taes`)를 보면 확장함수의 멤버변수가 우선시 사용되어 진다는것을 확인 하실수 있습니다.  

즉, 확장함수에서 사용되는 Scope는 확장함수를 정의한 클래스가 됨을 알 수 있습니다. (`$this = Rectangle`)  
이것을 정확하게 구분해주기 위해서는 다음과 같이 명시적으로 표기할 수 있습니다.

```kotlin
// Circle.kt
 
class Circle {
    ...
    fun Rectangle.name() = "${this.name}${this@Circle.name}${this@Circle.postFix}"
    ...
}
 
-- 결과 : RectangleCircle_taes
```

다음 `countSide` 호출시 stackoverflow 에러가 발생하는 문제를 살펴보도록 하겠습니다.  

우선, `Rectangle.countSide()`로 호출한 확장함수는 `Rectangle` 클래스에  countSide() 멤버함수가 존재하지 않기때문에 그다음우선순위로, 확장함수인 `Rectangle.countSide()`를 호출하게 됩니다.  
즉, 확장함수 내부에서 자기자신 function을 호출하는 재귀호출이 일어나고 있는 것 입니다.  

 확장함수를 정의하는 부분에서의 scope는 `$this = Rectangle`이기 때문에 Circle.function() 호출을 의도하고 사용했다면 잘못된 코드를 작성했다고 할 수 있겠습니다.  
이 또한 아래와같이 타겟 class를 명시해준다면 의도했던대로 동작 할 수 있을것 입니다.

```kotlin
// Circle.kt
 
class Circle {
    ...       
    fun Rectangle.countSide() {
        this@Circle.countSide()
    }
  ...
}
 
-- 결과 : 1
```

---

### Reference

- Kotlin doc (https://kotlinlang.org/docs/extensions.html#declaring-extensions-as-members)
- Kotlin effective (https://thdev.tech/kotlin/2020/10/27/kotlin_effective_08/)