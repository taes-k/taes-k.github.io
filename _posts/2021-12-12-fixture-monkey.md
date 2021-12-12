---
layout: post
comments: true
title: 테스트 객체를 더쉽게 만들어보자, Fixture-monkey
tags: [test, fixturemonkey]
---

### 테스트코드 작성의 어려움

테스트코드를 작성해야하는 이유는 이미 많이 알려져있기 떄문에 우리는 테스트코드를 열심히 작성하고 있습니다. 이번포스팅에서는 `테스트코드를 왜 작성해야하는가` 보다는 `테스트코드를 어떻게 쉽게 작성할수 있을까`에 대한 주제로 이야기를 해보겠습니다.  

테스트코드를 작성할때 가장 어려운것이 무엇인가에대한 질문에 많은 개발자분들이 다음과같은 어려움을 이야기를 할것에 일반적으로 공감하실것이라고 생각합니다.

- 테스트 객체 생성에 의미없는 시간이 너무 많이 사용된다.
- 다양한케이스의 테스트를 위해서 테스트 코드가 의미없이 길어진다.
- 테스트 엣지케이스 커버리지를 높이기위한 경계가 불분명하다.
- ...

---

### 테스트객체를 쉽게 만들수 없을까?

테스트코드 작성의 어려움의 가장 큰 원인이 테스트 객체생성에서 온다면, 테스트객체를 쉽게 만들수 있다면 테스트 코드 작성이 한결 쉬워질 것 입니다.

먼저, 아래 테스트코드를 살펴보도록 하겠습니다.

```java
@Test
public void doSomething_whenCaseA(){
    // given
    var something1Params = Something1Params.builder()
        .somethingField1(1)
        .somethingField2("A")
        .somethingField3(RandomStringUtils.random(10))
        .somethingField4(RandomStringUtils.random(10))
        .somethingField5(RandomStringUtils.random(10))
        .somethingField6(SomethingEnum.SOMETHING_1)
        .somethingField7(RandomStringUtils.randomNumeric(10))
        .somethingField8(RandomUtils.nextLong())
        .somethingField9("ABC")
        .build();

    var something2Params = Something2Params.builder()
        .somethingField1(1)
        .somethingField2("A")
        .somethingField3(RandomStringUtils.random(10))
        .somethingField4(RandomStringUtils.random(10))
        .somethingField5(RandomStringUtils.random(10))
        .somethingField6(SomethingEnum.SOMETHING_2)
        .build();

    var doActionResponse = SomethingResponse.builder()
        .somethingField1(something1Params.getSomethingField1())
        .somethingField2(something1Params.getSomethingField2())
        .somethingField3(something2Params.getSomethingField3())
        .build();

    given(SomethingService.doAction(any(), any())).willGiven(doActionResponse)

    // when
    var result = sut.doSomething(something1Params, something2Params);

    // then
    then(result.getSomethingField1())).equalsTo(doActionResponse.getSomethingField1());
    then(result.getSomethingField2())).equalsTo(doActionResponse.getSomethingField2());
    then(result.getSomethingField3())).equalsTo(doActionResponse.getSomethingField3());
}

@Test
public void doSomething_whenCaseB(){
    // given
    var something1Params = Something1Params.builder()
        .somethingField1(2)
        .somethingField2("B")
        .somethingField3(RandomStringUtils.random(10))
        .somethingField4(RandomStringUtils.random(10))
        .somethingField5(RandomStringUtils.random(10))
        .somethingField6(SomethingEnum.SOMETHING_1)
        .somethingField7(RandomStringUtils.randomNumeric(10))
        .somethingField8(RandomUtils.nextLong())
        .somethingField9("ABC")
        .build();

    var something2Params = Something2Params.builder()
        .somethingField1(2)
        .somethingField2("B")
        .somethingField3(RandomStringUtils.random(10))
        .somethingField4(RandomStringUtils.random(10))
        .somethingField5(RandomStringUtils.random(10))
        .somethingField6(SomethingEnum.SOMETHING_2)
        .build();

    var doActionResponse = SomethingResponse.builder()
        .somethingField1(something1Params.getSomethingField1())
        .somethingField2(something1Params.getSomethingField2())
        .somethingField3(something2Params.getSomethingField3())
        .build();

    // when
        ...

    // then
        ...
}
```

예시를 테스트코드지만 아마도 익숙하게 사용중이신 테스트코드 패턴일것이라 생각합니다. 이미 위 코드가 복잡하다고 생각하시는 분도 있을테지만 실무 코드에서는 이정도면 심플하다고 할 수 있을것 같습니다.

보시다시피 위 예제코드에서 벌써 테스트를 위한 객체생성이 코드의 90%이상 차지하는것을 확인 하실수 있으실겁니다. 위와 같은 코드를 반복적으로 작성하는 과정에서 많은 개발자분들을 `안티-테스터`의 길로 인도할지도 모른다라는 생각이 들기도합니다.

따라서 테스트객체를 쉽게 만들어 줄 수 있다면 우리의 테스트코드는 좀 더 발전 할 수 있을거라 생각합니다.  

이러한 고민을 저희가 처음한것이 아니기에, 테스트객체를 쉽게 만들어주기 위해 이미 몇가지 라이브러리가 존재합니다. 그중, 최근에 네이버에서 공개한 [Fixture-monkey](ttps://naver.github.io/fixture-monkey/) 오픈소스 라이브러리를 적용해 위 테스트코드를 수정해 보도록 하겠습니다.

```java
var fixtureMonkye = FixtureMonkey.create();

@RepeatedTest(100) // 매번 임의의 테스트객체를 생성해 테스트가능
public void doSomething_whenCaseA(){
    var something2Params = fixtureMonkey.giveMeBuilder(Something1Params.java)
        .set("somethingField1", 1)
        .set("somethingField2", "A")
        .sample();
    var something2Params = fixtureMonkey.giveMeOne(Something2Params.java);

    var doActionResponse = fixtureMonkey.giveMeOne(SomethingResponse.java, 
        it -> {
            it.toBuilder()
                .somethingField1(something1Params.getSomethingField1())
                .somethingField2(something1Params.getSomethingField2())
                .somethingField3(something2Params.getSomethingField3())
    })

    given(SomethingService.doAction(any(), any())).willGiven(doActionResponse)

    // when
    var result = sut.doSomething(something1Params, something2Params);

    // then
    then(result.getSomethingField1())).equalsTo(doActionResponse.getSomethingField1());
    then(result.getSomethingField2())).equalsTo(doActionResponse.getSomethingField2());
    then(result.getSomethingField3())).equalsTo(doActionResponse.getSomethingField3());
}
```

객체에 정의된 type과 constraints(`@NotNull`, `@Size`, `@Max` ...) 를 활용해 value들을 임의의 값들로 정의해 테스트객체를 손쉽게 만들고, 매 테스트마다 임의의값을 설정해줌으로서 엣지 케이스를 포함한 다양한 케이스를 제대로 검증 할 수 있도록 테스트객체를 설정 해 줄 수 있습니다. 물론 `builder`및 `customizer`등의 유연한 설정을 통해 테스트에서 고정이 필요한 값 혹은 다른 객체로부터 사용되야 할 값들은 직접 지정해 줄 수도 있습니다.

테스트코드를 쉽게 작성 할 수 있도록 도와주는것은 물론 이 `날뛰는 원숭이`는 여러분이 작성한 코드의 예상치못한 오류들을 검출하고 수정할 수 있도록 해 줄 것 입니다.

이 라이브러리에 대한 자세한 설명은 [Deview2021](https://tv.naver.com/v/23650158) 영상에서 확인하실수 있습니다.

---

### Reference

- Fixture-monkey (https://naver.github.io/fixture-monkey/)
- Deview 2021 (https://tv.naver.com/v/23650158)


