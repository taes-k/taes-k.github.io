---
layout: post
comments: true
title: Kotlin suppress 예약어 모음
tags: [suppress, suppress-warning]
---

### @Suppress, @SuppressWarning

인텔리제이의 큰 장점중 하나는 실시간 코드 inspection을 통해 간단한 정적분석기능이 내장되어 있다는것 입니다. 하지만 가끔은 의도한 `warning`이 파일에 계속해서 Warning 표시가 뜨게된다면 다른 코드들이 묻혀버리게 될 수 있습니다. (~~깨진 창문의 법칙~~) 이때 유용하게 사용되는것이 `@SuppressWarning(java)`, `@Suppress(kotlin)` 인데 이 어노테이션들을 붙인 코드에 대해서는 원하는 종류의 warning을 무시 할 수 있도록 해주는 역할을 합니다.  

간단한 사용법은 아래와 같습니다.
```kotlin
@Suppress("unused")
fun doSomething(){
	val unusedValue = 1
}
```

위와 같이 annotation value으로 무시하고자 하는 warning을 명시해줌으로서 해당 종류에대해 무시 할 수 있습니다. 이번 포스팅에서는 `unused`와 같이 `@Suppress()`에서 사용되는 예약어들을 정리해보려고합니다.

--- 

### Suppress 예약어 모음

#### `Unused`

- 미사용 필드 혹은 변수가 있을때 사용 할 수 있습니다.
- 메서드 호출이 없는경우에 사용 할 수 있습니다.

```kotlin
@Suppress("Unused")
fun unusedMethod(){
}
```

#### `Unused_parameter`

- 미사용 파라미터가 있을때 사용 할 수 있습니다.

```kotlin
@Suppress("Unused_parameter")
fun doSomething(unusedParam: String){
}
```

#### `SameParameterValue`

- 항상 동일한 파라미터값이 전달될때 사용 할 수 있습니다.

```kotlin
fun doSomething(){
	val param = "AlwaysSame"
	doSomething2(param)
}

@Suppress("SameParameterValue")
fun doSomething2(alwaysSameValue: String){
}
```

#### `Deprecation`

- `@deprecated`된 클래스,메서드,필드 등을 사용하는 코드에서 사용 할 수 있습니다.

```kotlin
@Suppress("Deprecation")
fun doSomething(){
	doSomething2()
}

@Deprecated
fun doSomething2(){
}
```

#### `Unchecked_cast`

- 체크되지 않은 타입 캐스팅에서 사용 할 수 있습니다.

```kotlin
@Suppress("Unchecked_cast")
fun doSomething(param: Any){
	val paramString = param as String
}
```

#### `NOTHING_TO_INLINE`

- 인라인 함수에서 인라인 가능 함수가 없는경우에 사용 할 수 있ㅡ니다.

```kotlin
@Suppress("NOTHING_TO_INLINE")
inline fun doSomething(param: Any){
	doSomething2()
}
```

#### `SimplifyBooleanWithConstants`

- boolean 로직을 간단히 할 수 있는경우 사용 할 수 있습니다.

```kotlin
@Suppress("SimplifyBooleanWithConstants")
fun doSomething(){
	val somethings = getSomethings()
	if(somethings.isEmpty() == false){
		...
	}
}
```

#### `BlockingMethodInNonBlockingContext`

- non-blocking 컨텍스트에서 blocking이 발생 할 수 있는 경우 사용 할 수 있습니다.

```kotlin
@Suppress("BlockingMethodInNonBlockingContext")
fun doSomething(param: Any){
	Flux { doSomething2() }
}
```

### `MoveVariableDeclarationIntoWhen`

- when 선언절에 넣고싶지 않은 value가 있을 경우 사용 할 수 있습니다.

```kotlin
fun doSomething(param: Any){
	@Suppress("MoveVariableDeclarationIntoWhen")
	val type = doSomething2()
	when(type){
		A -> ...
		B -> ...
		else -> ...
	}
}
```

### `SpringJavaInjectionPointsAutowiringInspection`

- Spring bean이 동적으로 등록되는 경우 autoWired에서 bean을 찾지 못할때 사용 할 수 있습니다.

```kotlin
class SomethingClass{
 	@Suppress("SpringJavaInjectionPointsAutowiringInspection")
    @Autowired
    private lateinit var somethingClass2: SomethingClass2
}
```