---
layout: post
comments: true
title: 스프링 부트 테스트 성능높이기
tags: [spring, springboot, test]
---

### 스프링 부트 테스트 성능높이기  

이전에 [SpringBootTest @MockBean 관련 포스트](https://taes-k.github.io/2022/02/02/spring-mockbean/)와 연계된 내용으로 `@MockBean` 을 사용하고있는 SpringBoot 테스트 환경에서 성능을 높일 수 있는 방법들에 대해 몇가지 적어보려고합니다. 

이전포스팅에서 공유드렸던것 처럼 SpringBootTest를 사용하고있는 환경에서는 테스트시에 사용하는 별도의 Application Context를 갖게되며, 테스트 수행시 Context 내 객체들에 변동이 생길때마다 ContextReload 과정을 거쳐 Context를 캐싱한다고 공유드렸습니다. 이 과정에서 테스트 수행성능의 하락이 발생 할 수도 있게 되는데 이러한 과정을 생각해서 테스트 성능을 높일수 있는 방법들에 대해 소개 드려보도록 하겠습니다.

---

### Context reload가 발생 할 수 있는 케이스

> An ApplicationContext can be uniquely identified by the combination of configuration parameters that is used to load it. Consequently, the unique combination of configuration parameters is used to generate a key under which the context is cached. The TestContext framework uses the following configuration parameters to build the context cache key:
> 
> - locations (from @ContextConfiguration)
> 
> classes (from @ContextConfiguration)
> 
> - contextInitializerClasses (from @ContextConfiguration)
> 
> - contextCustomizers (from ContextCustomizerFactory) – this includes @DynamicPropertySource methods as well as various features from Spring Boot’s testing support such as @MockBean and @SpyBean.
> 
> - contextLoader (from @ContextConfiguration)
> 
> - parent (from @ContextHierarchy)
> 
> - activeProfiles (from @ActiveProfiles)
> 
> - propertySourceLocations (from @TestPropertySource)
> 
> - propertySourceProperties (from @TestPropertySource)
> 
> - resourceBasePath (from @WebAppConfiguration)

doc : https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-ctx-management-caching

위 spring 도큐먼트 내용을 참고해 컨텍스트 캐시를 활용하지 못하는상황은 아래와 같습니다.

- `@MockBean`, `@SpyBean`의 사용
- `@TestPropertySource`, `@ContextConfiguration`, `@ActiveProfiles`등 테스트 클래스에서 특정 속성 변경 사용
- `@WebAppConfiguration` resource base 변경
- `@DirtiesContext` 사용으로 의도적인 dirtyContext 사용

---

### ContextReload 를 회피하여 성능개선을 위한 방안


```kotlin
@SpringBootTestclass Sample4_1ServiceIT {    
    @Autowired    
    private lateinit var sut: SampleService    
    @MockBean    
    private lateinit var externalSampleClient: ExternalSampleClient    
    
    @Test    
    fun runSampleTakes2Sec() {
                …    
    }
}
```

```kotlin
@SpringBootTest
class Sample4_2ServiceIT {
    @Autowired
    private lateinit var sut: SampleService

    @MockBean
    private lateinit var SampleDao: SampleDao

    @Test
    fun runSampleTakes2Sec() {
        …
    }
}
```

```kotlin
@TestPropertySource(properties = ["taes.test.enabled=true"])
@SpringBootTest
class Sample4_3ServiceIT {
    @Autowired
    private lateinit var sut: SampleService

    @Test
    fun runSampleTakes2Sec() {
        …
    }
}
```

위 3개 테스트 코드들을 예시로 들어 테스트 성능개선을 위한 방법들을 알아보겠습니다.

#### 성능개선 방법 1 - MockBean, Configuration 설정 통일시키기

```kotlin
@TestPropertySource(properties = ["taes.test.enabled=true"])
@SpringBootTest
class Sample5A_1ServiceIT {
    @Autowired
    private lateinit var sut: SampleService

    @SpyBean
    private lateinit var externalSampleClient: ExternalSampleClient

    @Suppress (”unused")
    @SpyBean
    private lateinit var SampleDao: SampleDao

    @Test
    fun runSampleTakes2Sec() {
        …
    }
}
```

```kotlin
@TestPropertySource(properties = ["taes.test.enabled=true"])
@SpringBootTest
class Sample5A_2ServiceIT {
    @Autowired
    private lateinit var sut: SampleService

    @Suppress (”unused")
    @SpyBean
    private lateinit var externalSampleClient: ExternalSampleClient

    @SpyBean
    private lateinit var SampleDao: SampleDao

    @Test
    fun runSampleTakes2Sec() {
        …
    }
}
```

```kotlin
@TestPropertySource(properties = ["taes.test.enabled=true"])
@SpringBootTest
class Sample5A_3ServiceIT {
    @Autowired
    private lateinit var sut: SampleService

    @Suppress (”unused")
    @SpyBean
    private lateinit var externalSampleClient: ExternalSampleClient

    @Suppress (”unused")
    @SpyBean
    private lateinit var SampleDao: SampleDao

    @Test
    fun runSampleTakes2Sec() {
        …
    }
}
```

테스트 클래스들이 각기 다른 설정값들을 가지고있다면 Spring Context 캐시를 활용하지 못하고 테스트 클래스마다 Context를 새로 초기화 하면서 성능이 하락하게 될것입니다. 만약 테스트 클래스들이 비슷한 설정 값들을 사용하고 있다면 통일된 설정을 통해 캐싱된 Context를 재활용 할 수 있도록 해줄수 있습니다.  

다만 불필요하게 들어간 세팅들이 테스트 클래스 내부에 세팅되어 테스트에 의도치 않은 상황을 만들어낼수 있다는점을 주의해줘야합니다.

#### 성능개선 방법 2 - 테스트 클래스 통합

```kotlin
@TestPropertySource(properties = ["taes.test.enabled=true"])
@SpringBootTest
class Sample5CServiceIT {
    @Autowired
    private lateinit var sut: SampleService

    @SpyBean
    private lateinit var externalSampleClient: ExternalSampleClient

    @SpyBean
    private lateinit var SampleDao: SampleDao

    @Nested
    inner class Sample5A_1ServiceIT {
        @Test
        fun runSampleTakes2Sec() {
            …
        }
    }
 
    @Nested
    inner class Sample5A_2ServiceIT {
        @Test
        fun runSampleTakes2Sec() {
            …
        }
    }

    @Nested
    inner class Sample5A_3ServiceIT {
        @Test
        fun runSampleTakes2Sec() {
            …
        }
    }

}
```

이전 방법이 테스트 클래스들이 분리되어있어 테스트 환경을 통합시키는게 조금 어려운 측면이 있는데, 이를 보완할수 있는 방법으로 테스트 클래스를 통합하는 방법도 있습니다. 통일된 설정을 가진 테스트 클래스들을 하나의 클래스로 통합해서 관리한다면 Context 재활용은 물론이고 클래스 관리가 용이 할 수 있다는 장점이 있습니다.

#### 성능개선 방법 3 - LazyMockBean

```kotlin
@TestPropertySource(properties = ["taes.test.enabled=true"])
@SpringBootTest
class Sample5A_1ServiceIT {
    @Autowired
    private lateinit var sut: SampleService

    @LazyMockBean
    private lateinit var externalSampleClient: ExternalSampleClient

    @Test
    fun runSampleTakes2Sec() {
        …
    }
}
```

```kotlin
@TestPropertySource(properties = ["taes.test.enabled=true"])
@SpringBootTest
class Sample5A_2ServiceIT {
    @Autowired
    private lateinit var sut: SampleService

    @LazyMockBean
    private lateinit var SampleDao: SampleDao

    @Test
    fun runSampleTakes2Sec() {
        …
    }
}
```

```kotlin
@TestPropertySource(properties = ["taes.test.enabled=true"])
@SpringBootTest
class Sample5A_3ServiceIT {
    @Autowired
    private lateinit var sut: SampleService

    @Test
    fun runSampleTakes2Sec() {
        …
    }
}
```

사실 일반적인 통합테스트에서 컨텍스트 초기화를 자주 일으키는 주범은 `@MockBean` 일 것 입니다. 특성상 특정 테스트에서만 사용되는 경우가 대부분이라 여러 클래스 설정을 통일시키기 어려운 경우가 많습니다.  

이전 포스팅에서 한번 공유드렸던 내용인데, 위 페인포인트 개선을 위해 `LazyMockBean` 라이브러리를 만들어 보았습니다. 이 시도의 주요 포인트는 캐싱된 컨텍스트를 그대로 사용하면서 해당 테스트 클래스가 사용되는 동안에만 타겟 `Bean`을 `MockBean` 혹은 `SpyBean`으로 교체하여 사용하고 테스트가 완료 된 후에는 기존의 Bean으로 원복시키는 방식으로 context를 훼손시키지 않고 재활용 하는 방식입니다.

link : https://github.com/taes-k/lazy-mock-bean

---

### 마무리

Spring boot test의 성능 하락에 악영향을 끼치는 `context reload` 를 회피하는 방법으로 성능향상을 꾀할수 있는 방법들을 알아보았습니다. 물론 이 방법들이 테스트의 안정성에 있어서는 조금은 훼손을 시킬수 있는 방법일수 있지만 적당하게 사용 할 수 있는 환경이라면 트레이드 오프를 통해 여러분의 테스트 성능을 향상시킬수 있는 좋은 방법이 될 수 있을거라 생각합니다.