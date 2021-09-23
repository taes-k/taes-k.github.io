---
layout: post
comments: true
title: 코틀린을 더 코틀린 답게, DSL
tags: [kotlin, dsl]
---

### DSL       

먼저, 개념에 대해서 설명하자면 `DSL(Domain-Specific Language)`은 `도메인 특화 언어`로 풀이되며 말 그대로 특정 영역에 대한 연산 및 작업을 간결하게 기술 할 수 있는 언어입니다. 

그냥 이해하자니 헷갈릴 수 있는데 우리가 흔히 쓰는 `SQL`, `정규식`이 바로 대표적인 `DSL` 이라고 할 수 있습니다. 모든 문제를 풀 수 있는 `범용 프로그래밍 언어`과는 달리, 특정 영역(`도메인`)에 초점을 맞추고 기능을 제한함으로써 효율적으로 목표를 달성하기 위한 언어입니다.  

이 목표를 위해 `DSL`은 `명령적`인 일반 프로그래밍 언어와는 달리 `선언적`인 방식의 문법을 가지고 있습니다. (`SELECT * FROM t_sample`) `선언적`으로 원하는 결과를 기술하면 실행엔진에서 세부로직을 통해 실행하는 방식으로 동작합니다.

위와같은 특성들덕에 `DSL`을 사용하면 `깔끔한 코드작성`이 가능합니다. 일반적으로 `깔끔한 코드작성`이라고 하면 다음과같은 코드를 이야기 합니다.

- 해당 코드가 어떤 로직을 가지고 작업을 할 지 명확하게 이해 할 수 있고 예측 할 수 있는 코드
- 불필요한 구문이나 번잡한 준비 코드가 가능한 적고 간결한 코드

---

### 코틀린 내부 DSL

`내부 DSL`이란, 독립적인 문법 구조를 가진 `SQL`과 같은 `외부 DSL`과는 달리 `DSL`의 핵심 장점을 유지하면서 범용언어(여기에서는 `코틀린`)를 동일한 문법으로, 특별하게 사용하는것을 말합니다. 즉, 코드는 어떤 `구체적인 과업`을 달성하기위한 것 이지만 `범용언어 라이브러리`로 구현을하는 것 입니다.

예시로 아래와같은 코드를 들 수 있습니다. 코틀린 코드에서 `HTML` 페이지를 생성하는 `내부 DSL` 입니다.

```java
fun createSimpleTable() = createHTML().
    table{
        tr{
            td{ 
                +"cell"
            }
        }
    }
```

위 코드는 아래와같이 작성한것과 동일 할 것 입니다.

```java
fun createSimpleTable() = """
    <table>
        <tr>
            <td>cell</td>
        </tr>
    </table>
"""
```

위 코드는 평범한 코틀린 코드로서, 별도의 특별한 템플릿 언어가 아닙니다. 따라서 위 예시를 보았을때 `내부 DSL`으로 생성된 코드는 다음과 같은 장점을 갖게될 것 이라는것을 알 수 있을것 입니다.

- 정해진 `DSL`문법으로 깔끔한 코드 작성이 가능하다.
- 정해진 `DSL`룰에 따른 컴파일 시점에 문법적 체크가 가능하다.
- 반복되는 코드의 효율적인 재사용이 가능하다.


그렇다면 위와같은 `내부 DSL`을 코틀린에서 어떻게 구현 할 수 있을지 알아보도록 하겠습니다.

```java
// HtmlTag.kt
open class Tag(val name: String) {
    private val children = mutableListOf<Tag>()
    protected fun <T : Tag> doInit(child: T, init: T.() -> Unit) {
        child.init()
        children.add(child)
    }

    override fun toString() = "<$name>${children.joinToString("")}</$name>"
}

fun table(init: TABLE.() -> Unit) = TABLE().apply(init)

class TABLE : Tag("table") {
    fun tr(init: TR.() -> Unit) = doInit(TR(), init)
}

class TR : Tag("tr") {
    fun td(init: TD.() -> Unit) = doInit(TD(), init)
}

class TD : Tag("td")

fun createTable() =
    table{
        tr{
            td{
            }
        }
    }

```

```java
// Main.kt

fun main(){
    print(createTable())
}
```

```
// result

<table><tr><td></td></tr></table>
```

위 코틀린 `내부DSL`를 구성하면서 사용된 코틀린 특화 코드에대해서 알아보도록 하겠습니다.

---

### 수신객체 지정 람다

`수신객체 지정 람다`는 구조화된 API를 만들때 도움이 되는 강력한 코틀린 기능중 하나로서 `with`, `apply` 등의 표준 라이브러리 메서드로 만들어 줄 수 있습니다. 간략한 예제코드를 통해 알아보도록 하겠습니다.

```java
fun alphabet(): String {
    val result = StringBuilder()
    for (letter in 'A'...'Z'){
        result.append(letter)
    }

    result.append("Now I know the alphabet!")
    return result.toString()
}
```

```java
fun alphabet(): String {
    val stringBuilder = StringBuilder()
    return with(stringBuilder) {
        for (letter in 'A'...'Z'){
            append(letter)
        }
        append("Now I know the alphabet!")
        toString()
    }
}
```

위와같이 반복적으로 사용되어지는 객체를 계속 사용할것이라고 암시를 해 줌으로서, 객체변수의 선언없이 간략하게 메서드를 호출 할 수 있는것을 확인 하실수 있습니다. (`result.append -> append`)

위에서 알아본 `수식객체 지정 람다`가 함수의 인자로 사용될때에는 조금 다르게 사용되어 집니다. 이 또한 예제를 통해 알아보도록 하겠습니다.

```java
fun buildString(builderAction: (StringBuilder) -> Unit)):String{
    val sb = StringBuilder()
    builderAction(sb)
    return sb.toString()
}

fun main(){
    val s = buildString{
        it.append("Hello, ")
        it.append("World!")
    }
}
```

```java
fun buildString(builderAction: StringBuilder.() -> Unit):String{
    val sb = StringBuilder()
    sb.builderAction()
    return sb.toString()
}

fun main(){
    val s = buildString{
        append("Hello, ")
        append("World!")
    }
}
```

인자부분이 어떻게 변경되었는지 살펴도록 하겠습니다.

```
    StringBuilder.() -> Unit
    (수신객체타입).(파라미터타입) -> (반환타입)
```

이는 `수식객체타입`이 미리 지정된 상태로서 (`StringBuilder`) 외부 타입의 멤버를 다른 수식자없이 사용하는 즉, `확장함수`형태로 인자로 받아 사용하는 방법입니다.   
예제코드를 확인해보면 인자로 받은 람다를 사용하는 코드가 변경된것을 확인 하실 수 있습니다. (`builderAction(sb)` -> `sb.builderAction()`)

또한 호출부 람다 선언에서도 보면, 위 `수식객체 지정 람다`에 대한 설명에서 알아 보았던 것 처럼 `it.append(...) -> append(...)`으로 변경되어 객체 선언이 생략된것을 보실수 있습니다.

이전 `HtmlTag.kt` 예제코드에서도 `확장함수타입`을 인자로 받도록 선언해 줌으로서 `DSL`을 깔끔하게 선언 할 수 있도록 도움을 주었습니다.

```
class TABLE : Tag("table") {
    fun tr(init: TR.() -> Unit) = doInit(TR(), init)
}
```

만약 일반 `람다`로 선언을 했다면 `DSL`이라는 이름이 무색하게도 아래와같이 수신 객체를 직접 명시해주면서 작성하기도, 이해하기도 쉽지 않은 코드가 구성되었을 것 입니다.

```
table{
    (this@table).tr{
        (this@tr).td{

        }
    }
}
```

---

### 코틀린 빌더 구성

`내부 DSL`을 구성하면서 갖게되는 가장 큰 장점으로, 특화 코드를 코틀린 코드로 표현 할 수 있기때문에 `추상화`와 `재사용`이 가능하게 된다는 점 일 것입니다.  

아래 예시 코드를 통해 알아보도록 하겠습니다.

```java
class TABLE : Tag("table") {
    fun tr(init: TR.() -> Unit) = doInit(TR(), init)
    fun singleCell(title: String) =
        tr{
            td{
                title
            }
        }
}

fun createTable() =
    table{
        singleCell("firstLine")
        singleCell("secondLine")
    }
```

위 `singleCell`과 같이 의미를 나타낼수 있는 코드를 묶어서 추상화를 통해 직관적인 코드구현을 가능하게 하며 재사용에도 편리하게 만들어 줄 수 있습니다.

---

### invoke 관례

코틀린의 `관례`라고 하면 특별한 이름이 붙은 함수들을 일반적인 함수호출이아닌, 좀 더 간단한 구문으로 호출 할 수 있게 지원하는 기능으로서 `invoke`관례를 사용하게 되면 좀 더 유연한 블록 중첩문 `DSL`을 구성할 수 있습니다. 

`FunctionN` 인터페이스안에도 포함되어있는 `invoke` 함수는 메서드 호출문 없이 호출이 가능하다는 `관례`를 가지고 있습니다.

```java
// CustomFunction.kt

class CustomFunction : Function1<String, String> {
    override fun invoke(p1: String): String {
        return "invoked-"+p1
    }
}
```

```java
// Main.kt

fun main(){
    val customFunction = CustomFunction();
    println(customFunction("param1"));
    println(customFunction("param2"));
}
```

```
// result

invoked-param1
invoked-param2
```

위처럼 메서드 호출을 생략할수 있다는 `관례`를 가지고 `DSL`에서는 아래와같이 활용 할 수 있습니다. 예시코드는 `Gradle DSL`에서 사용되는 예제 코드입니다.

```java
class DependencyHandler {
    fun compile(coordinate: String){
        println("Added dependency on $coordinate")
    }

    operator fun invoke(body: DependencyHandler.() -> Unit){
        body()
    }
}
```

```gradle
// flattend 호출 구조
dependencies.compile("junit:junit:4.11")

// block DSL 구조
dependencies{
    compile("junit:junit:4.11")
}
```

위와 같은 `Block DSL` 구조를 사용 할 수 있는 이유는 `invoke`의 `관례`덕분 입니다.  
만약, `invoke`의 관례를 사용하지 못했다면 다음과같이 표현될 것 입니다.

```java
class DependencyHandler {
    fun compile(coordinate: String){
        println("Added dependency on $coordinate")
    }

    fun ordiniaryInvoke(body: DependencyHandler.() -> Unit){
        body()
    }
}
```

```gradle
// flattend 호출 구조
dependencies.compile("junit:junit:4.11")

// block DSL 구조
dependencies.ordiniaryInvoke{
    compile("junit:junit:4.11")
}
```


---

### 중위호출

위 예제에서는 블록표현식을 이용한 코틀린 `DSL`을 알아보았지만, `DSL`은 꼭 블록표현식으로만 사용되어지는것은 아니고 표현하고자 하는 목적에 맞추어 더 편리한 방식으로 표현 할 수 있습니다.  

다른 형식으로 코틀린`DSL`을 구성하고 있는 `코틀린테스트DSL`을 예제로 살펴보도록 하겠습니다.

```
"kotlin" should start with "kot"
```

위 코드가 실제로 동작가능한 `코드`인가 의문을 가질수도 있겠지만, 코틀린의 `중위호출`을 `연계`해 사용함으로서 마치 영어 문장처럼 사용 가능한 실제로 동작이 가능한 코틀린 코드입니다.  

어떤로직을 통해 어떻게 구성되었는지 몰라도, 어떤 결과를 가지고 올것이라는 것을 쉽게 추측이 가능하실거라 생각듭니다. 이것이 `DSL`을 구성하는 이유중 하나일것입니다.  

위와 같은 `DSL`이 어떻게 만들어졌는지 살펴보도록 하겠습니다.

```java
object start

infix fun String.should(x: start) : StartWrapper = StartWrapper(this)

class StartWrapper(val value: String) {
    infix fun with(prefix: String) =
        if (!value.startsWith(prefix))
            throw AssertionError("String does not start with $prefix: $value")
        else
            Unit
}

```

```
"kotlin".should(start).with("kot")
-> "kotlin" should start with "kot"
```

`오버로딩`과 `확장함수`, `중위호출`선언을 통해 함수를 구성하여 원하는 `DSL`을 구성 할 수 있습니다. 위 방법을 활용해 다른 예제코드를 만들어보도록 하겠습니다.

```java
// DateDsl.kt

object ago
infix fun Int.days(x: ago) : AgoWrapper = AgoWrapper(Period.ofDays(this))

class AgoWrapper(val value: Period) {
    infix fun from(standard: LocalDate): LocalDate = standard - value
}
```

```java
// Main.kt

fun main(){
    val now = LocalDate.now()
    val yesterday = 1 days ago from now

    println(yesterday)
}
```

위코드는 `date`를 다루기 위한 `DSL`입니다. `중위호출`을 사용해 직관적으로 날짜데이터를 다룰수있도록 구성했습니다. 좀더 확장해서 사용한다면 `1 months ago from yeterday`, `5 years later from now` 등과 같이 날짜를 직관적으로 다룰 수 있을것입니다.


---

### 멤버확장

`확장함수`혹은 `확장프로퍼티`는 선언된 클래스의 멤버이자 확장하는 다른 타입의 멤버이기도 합니다. 이것을 `멤버 확장`이라고 부르는데, 위에서 `DSL`을 구성하기 위한 예제에서 보았듯이 `DSL`구현을 위해 많이 사용되어집니다.

코틀린에서 `SQL을 다루는 DSL`을 예제로 이에대한 설명을 좀더 자세히 알아보도록 하겠습니다.

```java
// CountryEntity.kt

object Country: Table(){
    val id = integer("id").autoIncrement().primaryKey()
    val name = varchar("name", 50)
}
```

```java
// Table.kt

class Table{
    fun integer(name: String): Column<Int>
    fun varchar(name: String, length: Int): Column<String>
    ...
    fun <T> Column<T>.primaryKey(): Column<T>
    fun Column<Int>.autoIncrement(): Column<Int>
    ...
}
```

위코드에서 보시면 `Table` 클래스에 선언된 `멤버함수`와 `확장함수`를 이용해 `Country entity`의 속성들을 정의하는것을 확인하실수 있습니다. `칼럼 속성정의 DSL`에서 가장 특징적인것은 메서드를 연쇄호출하여 다중 속성정의를 하는것을 보실 수 있는데, 각 메서드는 수신객체를 다시 반환함으로서 메서드를 연속해서 호출 할 수 있도록 하고 있습니다.

또한, 여기서 `멤버확장`의 특징을 확인하실수 있습니다.

- 확장함수로 정의하여 메서드를 제한적인 범위 안에서만 사용하도록 한다.(`Table` 클래스 내부에서만 사용 가능)
- 특정 메서드들에 대해서는 수신객체타입을 제한하여 사용 할 수 있도록 한다.(`autoIncrement`는 `columnType = Int`일때만 사용 가능)

위와같이 `멤버확장`을 통해 유연한 `DSL` 사용하면서도, `scope`혹은 `수신객체`에대한 정의를 함으로서 `DSL`을 의도한대로 사용 할 수 있도록 제한을 줄 수 있습니다.


---

### Reference

- 코틀린인액션 (https://search.shopping.naver.com/search/all?query=kotlin%20in%20action&frm=NVSHATC&prevQuery=%EC%BD%94%ED%8B%80%EB%A6%B0%20%EC%9D%B8%20%EC%95%A1%EC%85%98)


