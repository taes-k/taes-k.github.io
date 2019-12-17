---
layout: post
comments: true
title: Spring annotation @Component, @Configuration
tags: [spring]
---

### 개요  

Spring 프로젝트를 진행하면서 Bean으로 등록하기위해 `@Component`, `@Configuration`, `@Controller` ... 등의 여러 어노테이션을 사용합니다.  
그중에서도 `@Component`와 `@Configuration`은 사용상에 있어 차이가 있는데, 정확히 어떠한 차이로 이 어노테이션들을 구분하여 사용하는지 알아보도록 하겠습니다.

---

### Spring Bean

**Bean** : Spring bean container에 존재하는 객체로써, default 설정으로는 싱글턴으로 존재합니다.    
  
일반적으로 bean 으로 등록하기위해서는 다음과 같은 방법들이 존재합니다.
  
- xml 등록   
레거시 방식으로, xml 방식으로 등록하는 방법입니다.  

```java
<bean id="sampleController" class="com.sample.controller.SampleController" />
<bean id="sampleService" class="com.sample.controller.SampleService" />
<bean id="sampleDao" class="com.sample.controller.SampleDao" />
```

위와같이 xml에 class 경로와 id 를 직접 지정해주어 bean으로 등록해줄수 있었습니다.  

- 어노테이션 등록   

`@Component` 어노테이션을 이용하여 bean을 등록합니다.   
해당 어노테이션이 붙이면, 해당 class는 component-scan 과정의 대상이되어 bean 으로 등록될 수 있습니다.  
가장 기본이 되는 어노테이션은 `@Component` 이지만, 특수한 용도에 따라서 `@Controller`, `@Service`, `@Repository`, `@Configuration` (@Component를 메타 어노테이션으로 가지고있는 어노테이션)들도 Component-scan의 대상이 됩니다.  

```java
//Configuration.java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {
 
   /**
    * Explicitly specify the name of the Spring bean definition associated with the
    * {@code @Configuration} class. If left unspecified (the common case), a bean
    * name will be automatically generated.
    * <p>The custom name applies only if the {@code @Configuration} class is picked
    * up via component scanning or supplied directly to an
    * {@link AnnotationConfigApplicationContext}. If the {@code @Configuration} class
    * is registered as a traditional XML bean definition, the name/id of the bean
    * element will take precedence.
    * @return the explicit component name, if any (or empty String otherwise)
    * @see AnnotationBeanNameGenerator
    */
   @AliasFor(annotation = Component.class)
   String value() default "";
 
}
```

```java
// ComponentExample.java

@Component
public class A { .... }
 
public class B {
    @Autowired
    A a;
    .....
    .....
}
```

```java
// ConfigurationExample.java

@Configuration
public class ConfigClass {
 
    @Bean
    public UserClass getObject() {
        return new UserClass();
    }
}
```

---

### @Configuration vs @Component

위의 예제코드에서는 `@Configuration + @Bean` , `@Compoent` 형태로 코드를 작성했지만, 사실 `@Component +@Bean`도 사용이 가능합니다.  
따라서 `@Configuration`과 `@Component` 를 비교하는것이아닌, `@Bean`과 `@Component`를 비교해봐야하는데, 두 어노테이션 모두 Bean으로 등록하도록 하는 어노테이션인데 무엇이 다를까요?    

우선적으로, `@Component`는 개발자가 작성한 클래스에 선언되어 빈으로 등록 할 수 있습니다.   
그렇다면, 라이브러리 혹은 내장 클래스등 개발자가 직접 제어가 불가능한 클래스들을 Bean으로 등록하기위해서는 어떻게해야 할까요?  
이럴때 @Bean을 사용하여 등록해줄수 있습니다.  
  
예를 들어, 외부 라이브러리로 부터 받아온 User라는 class가 있을때 

```java
@Configuration
public class UserConfig {
 
    @Bean
    public User user() {
        return new User();
    }
}
```

위와같이 선언하여 User class를 Bean으로 등록할수 있습니다.  

---

### 마무리  

`@Configuration`은 `@Component`와 사용성을 위해 구분되어 일반적으로 위와같이 @Bean을 구성하는 Class임을 알려주는 명시적인 Component로써 사용되어집니다.