---
layout: post
comments: true
title: SpringBootTest @MockBean의 실행과정과 context reload
tags: [springtest, mockbean, spybean]
---

### MockBean, SpyBean

보통 테스트 수행시 타겟 코드에 대해서만 검증을 하기 위해서 의존되는 코드를 원하는 대로 처리 할 수 있게 `Mocking`을 설정하게됩니다. `Mocking`은 아래와같은 상황에서 주로 사용되게 됩니다.

- 단위테스트시 테스트 범위를 제한시키고자 할때
- 외부 API 호출 결과에 따라 테스트 성공여부를 결정짓지 않고자 할때
- 롤백 시키기 어려운 수행 로직을 대체하고자 할때
- API 호출 결과 및 파라미터등을 검증하고자 할때
- ...

[Mockito Features](https://github.com/mockito/mockito/wiki/Mockito-features-in-Korean)

위와같이 `Mocking`을 통해 테스트코드를 작성함에서 오는 어려움들을 해결 할 수 있기 때문에 자바코드에서 `Mockito`사용은 필수라고 봐도 무방합니다. 이러한 이유로 `Spring`에서도 `Spring-boot-test-starter` 라이브러리에 `Mockito`가 포함되어있습니다.

그중에서도 이번 포스팅에서는 `@MockBean`, `@SpyBean`에 대해서 알아보려고 합니다. 위 두개의 annotation은 `Mockito`에서 제공되는것은 아니고 `Spring-test`에서 제공되는 어노테이션 입니다. 

어떤 역할을 하는지는 이름을 통해 어느정도 유추 하실수 있으실거라 생각됩니다. Spring 에서는 보통 객체생성시 직접 생성하지 않고 `Dependency Injection(DI)`을 통해 프레임워크에 위임하여 생성하기 때문에 직접적으로 `Mocking` 테스트환경을 만드는것이 쉽지 않습니다.  

이러한 어려움을 도와주는것이 바로 `@MockBean`과 `@SpyBean`입니다. `Bean`등록 과정에서 테스트에 필요한 `Mocking`객체를 기존 객체 대신에 Bean으로 등록시켜 사용할수 있게 만들어주기때문에 해당 Bean을 의존 하는 모든 다른 객체들에 `DI`하여 손쉽게 `Mocking`객체를 사용 할 수 있도록 해줍니다.

```java
@SpringBootTest
public class ExampleTests {
  
	@MockBean
	private ExampleService service;

	@Autowired
	private UserOfService userOfService;

	@Test
	public void testUserOfService() {
		given(this.service.greet()).willReturn("Hello");
		String actual = this.userOfService.makeUse();
		assertEquals("Was: Hello", actual);
	}
}
```

---

### SpringBootTest MockBean 등록과정

위에서 알아본것과 같이 `@MockBean`은 Spring에서 테스트코드 작성을 손쉽게 해주는 매우 유용한 기능인데 어떤로직을 통해 MockBean을 등록 할 수 있는지 코드로 알아보도록 하겠습니다.  
우선 테스트코드가 수행될때는 `TestExecutionListener`에 의해 이벤트를 감지하고 처리할수 있도록 되어있습니다.

```java
// org.springframework.test.context.TestExecutionListener.java

public interface TestExecutionListener {

	default void beforeTestClass(TestContext testContext) throws Exception {
	}

	default void prepareTestInstance(TestContext testContext) throws Exception {
	}

	default void beforeTestMethod(TestContext testContext) throws Exception {
	}

	default void beforeTestExecution(TestContext testContext) throws Exception {
	}

	default void afterTestExecution(TestContext testContext) throws Exception {
	}

	default void afterTestMethod(TestContext testContext) throws Exception {
	}

	default void afterTestClass(TestContext testContext) throws Exception {
	}
}
```

`@MockBean`을 처리할때는 위 `TestExecutionListener`를 구현한 `MockitoTestExecutionListener`를 통해 처리되고 있습니다.

```java
// org.springframework.boot.test.mock.mockito.MockitoTestExecutionListener.java

/**
	TestExecutionListener to enable @MockBean and @SpyBean support. Also triggers MockitoAnnotations.openMocks(Object) when any Mockito annotations used, primarily to allow @Captor annotations.
*/
public class MockitoTestExecutionListener extends AbstractTestExecutionListener {

	@Override
	public void prepareTestInstance(TestContext testContext) throws Exception {
		initMocks(testContext);
		injectFields(testContext);
	}

	@Override
	public void beforeTestMethod(TestContext testContext) throws Exception {
		if (Boolean.TRUE.equals(
				testContext.getAttribute(DependencyInjectionTestExecutionListener.REINJECT_DEPENDENCIES_ATTRIBUTE))) {
			initMocks(testContext);
			reinjectFields(testContext);
		}
	}

	@Override
	public void afterTestMethod(TestContext testContext) throws Exception {
		Object mocks = testContext.getAttribute(MOCKS_ATTRIBUTE_NAME);
		if (mocks instanceof AutoCloseable) {
			((AutoCloseable) mocks).close();
		}
	}

	private void initMocks(TestContext testContext) {
		if (hasMockitoAnnotations(testContext)) {
			testContext.setAttribute(MOCKS_ATTRIBUTE_NAME, MockitoAnnotations.openMocks(testContext.getTestInstance()));
		}
	}

	private boolean hasMockitoAnnotations(TestContext testContext) {
		MockitoAnnotationCollection collector = new MockitoAnnotationCollection();
		ReflectionUtils.doWithFields(testContext.getTestClass(), collector);
		return collector.hasAnnotations();
	}

	private void injectFields(TestContext testContext) {
		postProcessFields(testContext, (mockitoField, postProcessor) -> postProcessor.inject(mockitoField.field,
				mockitoField.target, mockitoField.definition));
	}

	private void reinjectFields(final TestContext testContext) {
		postProcessFields(testContext, (mockitoField, postProcessor) -> {
			ReflectionUtils.makeAccessible(mockitoField.field);
			ReflectionUtils.setField(mockitoField.field, testContext.getTestInstance(), null);
			postProcessor.inject(mockitoField.field, mockitoField.target, mockitoField.definition);
		});
	}

	private void postProcessFields(TestContext testContext, BiConsumer<MockitoField, MockitoPostProcessor> consumer) {
		DefinitionsParser parser = new DefinitionsParser();
		parser.parse(testContext.getTestClass());
		if (!parser.getDefinitions().isEmpty()) {
			MockitoPostProcessor postProcessor = testContext.getApplicationContext()
					.getBean(MockitoPostProcessor.class);
			for (Definition definition : parser.getDefinitions()) {
				Field field = parser.getField(definition);
				if (field != null) {
					consumer.accept(new MockitoField(field, testContext.getTestInstance(), definition), postProcessor);
				}
			}
		}
	}
	...
}
```

`prepareTestInstance` 테스트 준비단계에서 `injectFields`메서드를 통해 `@MockBean` 필드를 찾아 `Mocking Bean`등록을 위한 구성을 수행하는것을 내부로직을 통해 확인 하실 수 있습니다.

---

### context reload

아마 대규모 시스템에서 `@MockBean`을 수행해보셨다면 경험해보셨을거라 생각하는데, `@MockBean`, `@SpyBean`가 포함된 테스트클래스를 수행 할 때 마다 `Spring Context reload`가 수행됩니다.

시스템 규모, 테스트실행환경, Spring프로젝트 환경등에 따라 다르겠지만 `Spring context`를 reload하는 과정은 짧게는 1초 길게는 1분까지도 걸리는 경우도 있습니다. 100개의 테스트 클래스를 수행하면서 10번의 reload가 일어난다고 가정한다면 테스트코드 수행이 10분이나 delay 되는 현상이 발생 될 수 있습니다.

실제 개발진에서는 다음과같이 comment를 남기기도 했습니다. [링크](https://github.com/spring-projects/spring-boot/issues/10015#issuecomment-322634989)

```
The Spring test framework will cache an ApplicationContext whenever possible between test runs. In order to be cached, the context must have an exactly equivalent configuration. Whenever you use @MockBean, you are by definition changing the context configuration.
```

위 comment에서 확인하실수 있다시피 이는 의도된 방향이며, 그도 그럴것이 `BeanFactory`에서 해당 `Bean`만 `Mocking`객체로 교체하면 되는것이 아니라 해당 Bean을 의존하고 있는 모든 Bean들을 모두 교체해서 주입해주어야 하기 때문에 이미 등록된 `Context`상에서 처리하기에는 쉽지 않은 작업입니다. 

내부로직을 조금더 들어가서 살펴보면 `Mocking Bean` 등록과정 중 `applicationContext`를 새롭게 동작시키면서 refresh 하는 로직이 포함되어있는것을 확인 할 수 있습니다.

```java
// org.springframework.boot.test.mock.mockito.MockitoTestExecutionListener.java

public class MockitoTestExecutionListener extends AbstractTestExecutionListener {

	private void postProcessFields(TestContext testContext, BiConsumer<MockitoField, MockitoPostProcessor> consumer) {
		...
		if (!parser.getDefinitions().isEmpty()) {
			MockitoPostProcessor postProcessor = testContext.getApplicationContext()
					.getBean(MockitoPostProcessor.class);
			...
		}
	}
	...
}
```

```java
// org.springframework.test.context.support.DefaultTestContext

	@Override
	public ApplicationContext getApplicationContext() {
		ApplicationContext context = this.cacheAwareContextLoaderDelegate.loadContext(this.mergedContextConfiguration);
		...
		return context;
	}

```

```java
// org.springframework.test.context.cache.DefaultCacheAwareContextLoaderDelegate

protected ApplicationContext loadContextInternal(MergedContextConfiguration mergedContextConfiguration)
			throws Exception {
		...

		if (contextLoader instanceof SmartContextLoader) {
			SmartContextLoader smartContextLoader = (SmartContextLoader) contextLoader;
			applicationContext = smartContextLoader.loadContext(mergedContextConfiguration);
		}
		else {
			...
		}

		return applicationContext;
	}

```

```java
// org.springframework.boot.test.context.SpringBootContextLoader

	@Override
	public ApplicationContext loadContext(MergedContextConfiguration config) throws Exception {
		...
		return application.run(args);
	}
```

```java
//  org.springframework.boot.SpringApplication

	public ConfigurableApplicationContext run(String... args) {
		...
		try {
			...
			context.setApplicationStartup(this.applicationStartup);
			prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
			refreshContext(context);
			...
		}
		...
		return context;
	}
```

```java
// org.springframework.context.support.AbstractApplicationContext

@Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");

			// Prepare this context for refreshing.
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBean
```

---

### Lazy-MockBean

잦은 `MockBean` 사용은 테스트 성능에 영향을 끼칠수 있고 이는 대규모시스템일수록 영향이 커질수 밖에 없습니다. 이러한 점을 해결하기 위해 [Lazy-mock-bean](https://github.com/taes-k/lazy-mock-bean)을 시도해봤습니다. 

이 시도의 주요 포인트는 위에서 Bean을 의존하는 모든 객체들을 찾아 교체하는작업이 쉽지 않아 reload를 수행한다고 언급했지만 사실 특정 테스트에서 사용되는 Bean은 한정적이기 때문에 꼭 모든 의존 객체들에 영향을 끼쳐야 하는것이 아니라 테스트에 사용되는 Bean에만 제한적으로 Mocking Bean을 사용하도록 교체되면 된다는 점 입니다.

```java
@SpringBootTest
public class ExampleTests {
  
	// UserOfService에서 의존되어 사용되는 ExampleService만 Mocking 객체로 대체
	@LazyMockBean
	private ExampleService service;

	@LazyInjectMockBeans
	@Autowired
	private UserOfService userOfService;

	@Test
	public void testUserOfService() {
		given(this.service.greet()).willReturn("Hello");
		String actual = this.userOfService.makeUse();
		assertEquals("Was: Hello", actual);
	}
}
```

위 라이브러리는 아래 링크에서 확인 가능합니다

- https://github.com/taes-k/lazy-mock-bean