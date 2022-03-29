---
layout: post
comments: true
title: 스프링 통합테스트 성능향상을 위한 여정
tags: [java, spring, integrationtest]
---

### 개요

테스트 성능개선에 대한 이야기는 다루고있는곳이 많지만 대부분 유닛테스트를 타겟으로 이야기를 하고있습니다. 유닛테스트의 경우 어느정도 숙달되면 빠르고 독립적인 테스트코드를 만드는것이 어렵지 않지만 대부분의 통합테스트는 DB, API Connection, SpringContext 등 연계된 시스템들이 많아 고려할것도 많고 유닛테스트에 비해 실행속도도 느릴 수 밖에 없습니다.

가능하면 유닛테스트를 통해 모든 코드를 검증 할 수 있다면 좋겠지만, 로직 흐름을 정확하게 테스트하기 위해서는 잘짜여진 통합테스트만한것이 없을것입니다. 오늘은 테스트중에서도 통합테스트 성능을 높이기위한 방법들에 대해서 이야기를 해보고자 합니다.

먼저, 오늘 다룰 통합테스트에 대한 용어정리를 하고 가도록 하겠습니다.  
통합테스트는 보통 로직이 수행되는 여러 모듈들을 통합하여 테스트하는것을 의미할텐데 여기에서는 간단하게 `SpringBoot`기반에서 `@SpringBootTest`를 통해 테스트하는 모든 테스트코드들을 통합테스트로 정의하도록 하고 본론으로 들어가도록 하겠습니다.

---
### 스프링 통합테스트 성능 향상을 위한 여정
#### 1. 통합테스트는 느리다!

통합테스트 성능 향상을 소개드린다고 했는데 갑자기 통합테스트는 느리다고하니 무슨말인가 싶을수 있으실텐데 통합테스트는 어떤수를 쓰더라도 단위테스트보다 느릴수밖에 없습니다. 따라서 단위테스트로 검증이 가능한 테스트일 경우 가능하면 통합테스트대신 단위테스트로 변경하는것이 가장 효과적이고 바람직한 성능향상의 방향입니다.

### 2. 테스트 실행을 분리하자.

두번째 방법은 `느린테스트는 자주 수행시키지 않는다`는것 입니다.
위에서 `통합테스트는 느리다` 라고 말씀을 드렸으니 결국, 통합테스트는 자주 수행하지 말아야한다의 뜻으로 전달드릴수 있겠습니다. 
테스트 수행은 코드를 작성하는 과정에서 수시로 실행되어야 할텐데, 매번 오래걸리는 통합테스트가 함께 실행된다면 그만큼 비효율적이지 않을수 없을것입니다. 특히나 대규모시스템에서의 잦은 통합테스트 실행은 상당한 시간이 소요되니 로컬 테스트뿐만 아니라 CI를 통한 테스트를 수행시에도 많은 시간이 소요 될 수 있습니다. 

따라서 제안드리는 방법은 `통합테스트`실행과 `단위테스트`실행을 분리해 통합테스트는 필요할때만 테스팅을 수행 시키는 것 입니다.  
`JUnit5 Tag` 기능을 사용해 간단하게 Intellij, Gradle 에서 실행을 분리시킬수 있습니다.

### 3. 외부 호출 Mocking

통합테스트를 수행하는 목적중 하나는 전체 로직 실행 프로세스를 테스트하는 것 이긴 하지만 단순 정보조회 혹은 상태변경등 충분히 예측가능하고 검증된 API를 호출하는 경우에는 과감하게 `Mocking`을 통해 외부시스템 호출을 제거하고 성능을 향상시킬수 있습니다. 

### 4. Spring Context 캐시 활용

위 테스트 실행 예시들을 보시면서 느끼셨겠지만 통합테스트 수행에서 많은 시간을 차지하는것은 `SpringContext`를 초기화하는 작업입니다. 따라서 통합테스트의 성능향상에서 가장 고려해야할 부분은 `SpringContext` 초기화를 줄이는데 신경을 써야합니다. 

다행히도 `SpringBootTest`에는 `ContextCaching` 캐싱전략이 반영되어 있기때문에 캐시를 잘 활용한다면 통합테스트 성능향상에 큰 도움이 될 수 있습니다. 컨텍스트 캐시키를 구성하는 구성요소들을 살펴보면 아래와같습니다.

> An ApplicationContext can be uniquely identified by the combination of configuration parameters that is used to load it. Consequently, the unique combination of configuration parameters is used to generate a key under which the context is cached. The TestContext framework uses the following configuration parameters to build the context cache key:
> 
> - locations (from @ContextConfiguration)
> - classes (from @ContextConfiguration)
> - contextInitializerClasses (from @ContextConfiguration)
> - contextCustomizers (from ContextCustomizerFactory) – this includes @DynamicPropertySource methods as well as various features from Spring Boot’s testing support such as @MockBean and @SpyBean.
> - contextLoader (from @ContextConfiguration)
> - parent (from @ContextHierarchy)
> - activeProfiles (from @ActiveProfiles)
> - propertySourceLocations (from @TestPropertySource)
> - propertySourceProperties (from @TestPropertySource)
> - resourceBasePath (from @WebAppConfiguration)

doc : https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-ctx-management-caching

위 도큐먼트 내용을 참고해 컨텍스트 캐시를 활용하지 못하는상황은 아래와 같습니다.

- `@MockBean`, `@SpyBean`의 사용
- `@TestPropertySource`, `@ContextConfiguration`, `@ActiveProfiles`등 테스트 클래스에서 특정 속성 변경 사용
- `@WebAppConfiguration` resource base 변경
- `@DirtiesContext` 사용으로 의도적인 dirtyContext 사용

대규모 시스템에서 위와같은 케이스의 통합테스트가 많이 있는경우 잦은 컨텍스트 초기화가 수행되면서 통합테스트의 성능이 나빠질 수 있기 때문에 주의해서 사용해야합니다.


### 5. Spring Context 초기화를 최소한으로 만들기

통합테스트 성능개선을 위해 `@MockBean`을 활용하라고 했던 3번째 방법과 컨텍스트 캐시를 사용하기 위해 `@MockBean`을 사용하지 않아야한다는 4번째 방법에서 모순이 생겼습니다. 

가능하면 한번의 컨텍스트초기화로 전체 테스트에서 컨텍스트를 재사용 할 수 있다면 좋겠지만 특정 테스트를 수행하기 위해서는 `@MockBean`을 사용하거나 테스트수행에서 의도적으로 설정값을 바꿔야하는 상황도 분명히 존재 할것이기 때문에, 가능한 Context 초기화가 일어나는 상황을 최소한으로 만들어 주는것이 좋습니다.

#### MockBean, Configuration 설정 통일시키기

특정 테스트클래스들에서 비슷한 설정값들을 사용하고 있다면 테스트 클래스마다 통일된 설정을 통해 컨택스트 캐시를 활용 할 수 있도록하는것이 중요합니다.


#### 테스트 클래스 통합

여러 테스트 클래스들의 설정값들을 통일시켜 관리하는것은 관리 리소스상 어려울수 있습니다. 이 경우에는 `@Nested`를 활용해 여러 테스트클래스들을 하나로 통일시켜 관리하는것 또한 좋은 방법입니다. 

#### LazyMockBean

사실 일반적인 통합테스트에서 컨텍스트 초기화를 자주 일으키는 주범은 `@MockBean` 일 것 입니다. 특성상 특정 테스트에서만 사용되는 경우가 대부분이라 여러 클래스 설정을 통일시키기 어려운 경우가 많습니다.  

위 페인포인트 개선을 위해 `LazyMockBean` 라이브러리를 만들어보았습니다. 이 시도의 주요 포인트는 캐싱된 컨텍스트를 그대로 사용하면서 해당 테스트 클래스가 사용되는 동안에만 타겟 `Bean`을 `MockBean` 혹은 `SpyBean`으로 교체하여 사용하고 테스트가 완료 된 후에는 기존의 Bean으로 원복시키는것 입니다.


link : https://github.com/taes-k/lazy-mock-bean

위 라이브러리는 안정화를 거쳐서 조만간 사내라이브러리로 공개예정이니 많은관심부탁드립니다.


### 6. 통합 테스트 병렬실행?

아마도 테스트 성능개선이라는 타이틀에서 가장 먼저 떠올리셨을 병렬실행입니다. Junit5 에서는 간단한 설정을 통해서 병렬실행을 지원하고있고 통합테스트도 역시 같은 설정으로 병렬수행을 할 수 있습니다.

```properties
# test/resources/junit-platform.properties

junit.jupiter.execution.parallel.enabled=true
junit.jupiter.execution.parallel.mode.default=concurrent
junit.jupiter.execution.parallel.mode.classes.default=concurrent
```

의도했던대로 테스트들이 병렬로 시작해서 동시에 실행되지만 생각보다 기대한 성능이 나오지 않을 수 있습니다. 이는 병렬수행 오버헤드와 컨텍스트 캐싱이 적용되지 않는점 때문에 오히려 캐싱을 활용한 테스트코드의 성능보다 더 나쁘게 나올수도 있습니다. 또한 병렬수행이 고려되지 않은 기존의 코드들 (`Mocking 설정`)이 호환이 되지 않을 수 있어 수정이 필요 할 수 있다는 단점이 있습니다.

따라서 통합테스트 병렬수행은 각자의 프로젝트에서 직접 비교해보고 적용하는것을 추천드립니다. 가벼운 시스템에서 독립적인 환경이 잘 보장된 테스트코드에서는 성능향상을 위해 충분히 고려해볼법한 옵션이 될 것입니다.

---

### 마무리

오늘 소개드린 방법들은 뭔가 특별한 기술들이 아니기에 이미 많은 프로젝트에서 적용하고 계셨을수도 있을거라 생각합니다. 또한 10분걸리던 통합테스트가 1분만에 끝나는것과 같은 드라마틱한 변화를 기대하신분이라면 적용결과를 보고 많이 실망하셨을거라 생각도 드는데, 저희 시스템 기준으로는 약 30%정도의 테스트 수행속도 개선을 시킨 방법을 공유드렸습니다.

마지막으로 통합테스트 수행에서는 `컨텍스트 캐시`를 잘 활용하는 테스트코딩을 해야 한다는점을 한번 더 말씀드리며 마무리하겠습니다.







