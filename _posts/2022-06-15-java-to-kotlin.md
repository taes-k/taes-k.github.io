---
layout: post
comments: true
title: java 프로젝트 kotlin 으로 전환하기
tags: [java, kotlin, test]
---

### java to kotlin 전환시작 - 전환의 이유

최근 팀에서 kotlin을 도입하고 명확한 장점에대해 공감하게 되어 기존 java 프로젝트를 kotlin 으로 전환하는 작업을 진행했고, 성공적인 전환을 완료했습니다. 작업 과정과 그 과정에서 얻은 경험들을 이번 포스팅을 통해 공유드리고자 합니다.

가장 먼저, 팀에서 kotlin 전환의 공감을 얻고 진행 할 수 있었던 이유에 대해서 공유드리자면 아래와 같습니다.

1. Null 안정성
    - Kotlin에서는 타입 선언시에 `Nullable`/`NotNull`을 명시적으로 표현해야하기 때문에 런타임에서 `NullPoinException`에 대한 방지를 명확하게 할 수 있습니다.
2. Immutalbe 객체
    - java에서도 물론 immutable 객체들이 지원되지만, kotlin 에서는 immutable 객체들이 default 으로 정의되는 경우가 많아 객체 변경에 대해 좀 더 보수적으로 코딩 할 수 있게 해줍니다.
3. Decimal operation
    - 돈 관련 계산을 하는 경우에서 부동소수점등의 이슈를 피하고자 `Decimal` 타입을 사용하게되는데, 해당 타입은 일반적인 연산자 계산이 되지 않습니다.
    - kotlin 에서는 operator override 기능을 통해 연산자 계산을 정의해 줄 수 있어 객체간 연산을 편하고 가독성 좋게 작성 할 수 있습니다.
        ```java
        // java

        BigDecimal money1 = new BigDecimal("100.5");
        BigDecimal money2 = new BigDecimal("8.3");

        // 덧셈
        BigDecimal sum = money1.add(money2);

        // 뺄셈
        BigDecimal subtract = money1.subtract(money2);

        // 곱셈
        BigDecimal multiply = money1.multiply(money2);

        // 나눗셈
        BigDecimal divide = money1.divide(money2, 2, BigDecimal.ROUND_HALF_UP);
        ```

        ```kotlin
        // kotlin

        val money1 = BigDecimal("100.5")
        val money2 = BigDecimal("8.3")

        // 덧셈
        val sum = num1 + num2

        // 뺄셈
        val subtract = num1 - num2

        // 곱셈
        val multiply = num1 * num2

        // 나눗셈
        val divide = num1 / num2
        ```
4. 가독성 좋은 문법 작성
    - kotlin 에서는 코드의 가독성이 좋게 만드는 다양한 기능들이 지원됩니다.
    - 스마트캐스팅
        ```java
        public void doSomething(Object param){
            if (param instanceOf String){
                ((String)param).length() 
            }
            param.doAny() // nullPointException 위험 존재
            if (param != null) {
                param.doAny()
            }
            ...
        }
        ```

        ```kotlin
        fun doSomething(param: Any?){
            if (param is String){
                param.length() // param ==> String (String type smartCasting)
            }
            param?.doAny()
            if (param != null) {
                param.doAny() // param ==> Any (not null smartCasting)
            }
        }
        ```
    - Scoping function
        ```java
        val param: String? = "string"
        val length = param?.let {String::length}
        ```

        ```kotlin
        String param = "string";
        Integer length = (param != null) ? param.length() : null;
        ```
    - 간편한 람다 프로그래밍 
        ```java
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
        List<Integer> squaredNumbers = numbers.stream()
                .map(num -> num * num)
                .collect(Collectors.toList());
        }
        List<Integer> evenNumbers = numbers.stream()
                .filter(num -> num % 2 == 0)
                .collect(Collectors.toList());
        ```

        ```kotlin
        val numbers = listOf(1, 2, 3, 4, 5)
        val squaredNumbers = numbers.map { it * it }
        val evenNumbers = numbers.filter { it % 2 == 0 }
        ```
5. java와 상호운용가능
    - 어쩌면 kotlin으로 전환할수 있었던 가장 큰 이유이기도 합니다. 
    - 일부 코드들에 대해서만 kotlin 전환을 하더라도 java와 코틀린 코드들이 상호운용 가능하기 때문에 점진적 전환이 가능합니다.
    
물론 몇가지 단점도 존재합니다.
1. 컴파일 성능 하락?
    - 일반적으로 kotlin의 컴파일 성능이 java에 비해 떨어진다고 알려져있어 POC 테스틀 진행해보았는데, 크게 체감될 정도는 아니었습니다.
2. 구성원들의 러닝커브?
    - kotlin이 팀 구성원 대부분이 주로 써봤던 언어는 아니었기에 능숙하게 사용하기 위한 러닝커브는 존재 했습니다.
    - 다만, kotlin의 문법이 java와 크게 다른 부분이 없고 구성원들이 새로운 언어에 습득에 대한 열정이 높았기때문에 학습에 대한 문턱이 매우 낮았고 오히려 좋은 동기부여가 될 수 있었습니다.

위와같은 점들로 kotlin 전환작업이 구성원들 전체적으로 공감을 얻어 큰 이견없이 전환을 진행 해 볼수 있었습니다.

---

### 전환 방법

이제부터는 실제 전환 했던 방법에 대해서 공유드려보려고 합니다.  

#### 전환 전략 세우기

가장 먼저 할 일은 프로젝트에 맞는 전환전략을 세우는 일 입니다.  
전환을 고민하고 계시는 프로젝트들은 대부분 운영 서비스가 진행중이고, 규모가 작지않은 프로젝트들일것 이라고 생각합니다. 때문에 전체 코드를 한번에 전환시키기보다는 부분적으로 전환시키면서 이슈 발생시 롤백 할 수 있도록 전략을 세우고 운영서비스의 영향도를 확인해 나가는 방식으로 점진적 전환전략을 세우는게 중요합니다.

이는 프로젝트의 규모나 형태에 따라 전략이 다를것이지만, 저희 프로젝트를 기준으로 세운 전략에 대해 공유드리겠습니다.
저희 프로젝트는 멀티모듈 구조에 Spring 프로젝트의 일반적인 layer 구조로 구성되어 있는 케이스으로, 코드간 의존성이 가장낮은 모듈 및 파일을 우선적으로 전환 하는 방식으로 전략을 세웠습니다.


> 1\. 빌드 스크립트 (`build.gradle` -> `build.gradle.kt`)  
> 2\. 테스트 코드 (`SomethingTest.java` -> `SomethingTest.kt`)  
> 3\. 유틸리티 클래스 (`SomethingUtils.java` -> `SomethingUtils.kt`)  
> 4\. 데이터 layer (`SomethingEntity.java` -> `SomethingEntity.kt`, `SomethingRepository.java` -> `SomethingRepositoy.kt` )  
> 5\. 서비스 layer (`SomethingService.java` -> `SomethingService.kt`, `SomethingController.java` -> `SomethingController.kt`, `SomethingRequest.java` -> `SomethingRequest.kt`)

위와같이 의존성이 있는 layer별로 전환대상을 나눈다는 큰 전략을 세우고 3,4,5 전환시에는 기능별로 vertical 하게 전환하면서 검증을 진행하는 방식을 택했습니다. 

> 1\. 빌드 스크립트 (`build.gradle` -> `build.gradle.kt`)  
> 2\. 테스트 코드 (`SomethingTest.java` -> `SomethingTest.kt`)

> 3-1. 유틸리티 클래스 (`Something1Utils.java` -> `Something1Utils.kt`)  
> 4-1. 데이터 layer (`Something1Entity.java` -> `Something1Entity.kt`)  
> 5-1. 서비스 layer (`Something1Service.java` -> `Something1Service.kt`)

> 3-2. 유틸리티 클래스 (`Something2Utils.java` -> `Something2Utils.kt`)  
> 4-2. 데이터 layer (`Something2Entity.java` -> `Something2Entity.kt`)  
> 5-2. 서비스 layer (`Something12Service.java` -> `Something2Service.kt`)


#### auto converting

이제 실제 전환을 위해 가장 먼저 할 일은 auto converting 기능을 사용하는 것 입니다. 아시다시피 kotlin 은 intellij 사에서 만든 언어이기 때문에 java -> kotlin 컨버팅을 위한 지원이 매우 잘 되어있습니다. 경험상 java 코드를 직접 kotlin으로 옮기게되면 로직 전환과정에서 human-fault 실수가 있을 수 있기 때문에 auto converting 기능을 적극적으로 사용하여 작업공수와 실수 가능성을 최소한으로 줄일 수 있었습니다.

`intellij IDEA`을 사용한다면 다음과 같은 방법으로 auto converting 기능을 사용 할 수 있습니다.

1. java 코드를 kotlin 파일에 copy & paste
    - 가장 쉬운 방법으로 java 코드를 kotlin 파일에 복사 & 붙여넣기만 해도 kotlin 전환 confirm 창이 뜨며 자동으로 kotlin 코드로 변환을 해 줍니다.
2. 상단 `Code` 탭에서 `Covert Java to Kotlin` 기능 사용
    - 파일단위로 전환을 수행 할 수 있습니다.

#### 세부 코드 수정

auto converting 기능이 대부분의 코드들은 잘 지원해 주지만, 문맥을 고려하지 않고 기계적으로 java의 로직을 그대로 옮기다보니 몇몇 코드들의 경우에는 `kotlin스럽지 않게` 변경되는 경우도 있고 오히려 가독성이 떨어지고 성능상 단점을 갖는 코드가 만들어질수도 있습니다. 

이러한 코드들에 대해서는 검토후에 직접 수정이 필요합니다. `kotlin스럽지 않게 변환된 코드`들은 아래 예시와 같습니다. 

```java
// java

List<String> params = Arrays.asList("param1", "param2", "param3");
params.stream().forEach(param -> {
    String uppercasedParam = (String) param.toUpperCase();
    System.out.println("Uppercased Language: " + uppercasedLanguage);
});
```

```kotlin
// kotlin

val params: List<String> = listOf("param1", "param2", "param3")
params.forEach { param: String? ->
    val uppercasedParam: String = param?.toUpperCase() as String
    println("Uppercased Language: $uppercasedParam")
}
```

```kotlin
// kotlin

val params: List<String> = listOf("param1", "param2", "param3")
languages.forEach { param: String ->
    val uppercasedParam = param.toUpperCase() 
    println("Uppercased Language: $uppercasedParam")
}
```

또다른 예시코드


```java
// java

List<String> params = new ArrayList<>();
params.add("param1");
params.add("param2");
params.add("param3");
```

```kotlin
// kotlin

val params: MutableList<String> = ArrayList() // -> mutableList
params.add("param1");
params.add("param2");
params.add("param3");
```

```kotlin
// kotlin

val params: List<String> = listOf("param1", "param2", "param3") // -> immutableList
```

kotlin에 익숙하지 않거나, 전환 초반의 경우에는 세부 수정대상코드를 선정하는것부터도 속도가 나지 않을수도 있지만, 어느정도 숙달된 경우에는 수정 필요 대상들을 타입화 해서 꽤나 속도감있게 작업이 가능 하실것으로 예상합니다.  

제가 작업시에 세부수정 대상으로 많이 검토된던 코드들은 아래와 같습니다.

- `var` -> `val`, `lateinit var`
- `Nullable type` -> `NotNull type`
- `MutableCollections` -> `ImmutableCollections`
- `java stream 연산` -> `kotlin stream 연산`
- `java 분기 처리 구문` -> `kotlin scope function`
- `kotlin 전용 라이브러리 사용`
- 등...

#### 자동변환되지 않는 코드들

위단계까지 처리 성공하셨을 경우에는 사실 굉장히 수월하게 코드전환을 마무리 하신것이라고 생각이 듭니다. 모든 코드가 위 단계로 전환 완료가 된다면 좋겠지만, 일 부코드들에서는 자동변환 단계에서부터 실패하는 경우가 있을것이라고 생각합니다. 

해당코드들은 직접 kotlin으로 전환하거나 별도의 해결책이 필요합니다.

저희 팀에서 작업하면서 가장 애먹었던것은 java 프로젝트에서 필수적으로 사용하는 라이브러리인 `Lombok`을 사용하는 경우가 대표적인 케이스 였습니다.

`Lombok`의 경우 autoConfigure 과정을 통해 builder 클래스 등을 자동 생성해 손쉽게 사용할수 있게 만들어주는데 Kotlin과 java 를 함께 사용하는 경우에는 Kotlin 에서 `Lombok` 생성 코드들을 사용 할 수가 없어 java+kotlin 혼용된 중간 과정에서 객체 생성 코드를 사용하는데 많은 애를 먹었습니다. 

이 케이스는 lombok 에서 만들어주는 자동생성 코드들을 decompile 하여 코드를 직접 정의해줌으로서 문제를 해결 할 수 있었습니다.

---

### 전환시 주의할점

위 전환 과정 공유에서는 간단한게 전환을 완료한것 같지만, 전환과정에서 얻게된 주의하거나 도움이 될만한 lesson learn 을 공유드려보겠습니다.

#### builder 설정
- `org.jetbrains.kotlin.plugin.spring`
    - 클래스들에 자동으로 `open`을 붙여줌으로서 상속 가능하도록 해줍니다.

#### NotNull, Nullable type 설정
- kotlin 특성상 타입들의 nullable 여부를 정의해줘야합니다.
- 기존 자바 프로젝트에서 `@NotNull`, `@Nullable` 어노테이션을 잘 활용하셨다면 전환하실때 큰 도움이 될 수 있을테지만 그렇지 않다면 `NotNull` type 으로 정의하기 전 확인이 필요합니다.
- 특히나 Controller requestParameter 단에서 기존 Nullable field로 들어오고있는 파라미터가 NotNull으로 설정되는 경우 바로 에러가 발생 할 수 있습니다.

#### nullCheck
- 로직상 NotNull이 보장되는 변수들의 경우 `requireNotNull`, `checkNotNull` 함수 등을 통해 NotNull type 으로 smartCasting을 해주면 좀 더 깔끔한 코드로 구현이 가능합니다.

#### kotlin 전용모듈
- `ObjectMapper.registerModule(new KotlinModule())`
    - objectMapper 사용시 kotlin 전용 코드들이 정상적으로 deserialize 되지 않는 경우가 있습니다. 이를위해 kotlin module을 등록시켜줘야합니다.

#### lombok 대체 기능
- `@builder` -> `Something(param1 = ?, parame2 = ?) // namedArgument 생성자`
- `@toBuilder` -> `something.copy(param2 = "fixed")`
- `@with` -> `something.copy(param3 = "addedParam")`

---

### 마무리

java to kotling 프로젝트 전환을 진행하고자 하시다면 일부 코드들에 대해서는 freezing이 필요 할 수 있기때문에, 경험상 과감한 결단과함께 가능한 많은 구성원들과 함께 빠르게 진행 하는것을 추천드립니다.

저희가 작업한 프로젝트는 10만 라인 정도 되는 꽤나 대규모 프로젝트 였기 때문에 코틀린 전환작업만으로도 약 12 man-month 투자가 필요한 작업이었습니다.  
그렇지만 kotlin 전환으로 얻을 수 있는 생산성이 그 이상의 결과를 낼 수 있을것이라고 판단했고, 실제로도 구성원들이 체감 할수 있을정도로 좋은 결과를 얻었다고 생각합니다. 특히나 단순히 언어 변경만으로 기존 10만라인 되던 코드 사이즈가 5만 라인정도로 50% 감소된 결과를 얻을수 있었습니다.

아직 kotlin 전환을 망설이고 계신 분이 계시다면, 신규 프로젝트 진행시 kotlin을 도입해보시고 어느정도 익숙해지신 뒤 kotlin의 장점이 명확해졌다고 판단되신다면 그 이후에 기존 프로젝트를 kotlin으로 전환하는것도 좋은 도입 프로세스라고 생각합니다.

---