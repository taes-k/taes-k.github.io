---
layout: post
comments: true
title: 몰라도 되는 Spring - 리플렉션을 이용하는 Spring container 
tags: [spring, aop, proxy, reflection]
---

### Spring container

이번 포스팅에서는 `Spring container`의 주요 기능인 `Bean 등록`과 `DI`가 내부적으로 어떻게 수행되는지 알아보려고 합니다.  

우선 Spring에서 설명하는 `Bean`과 `DI (Dependency Injection)`에 대해 알아보겠습니다.

> **`Bean Initializing`**  
> 
> A Spring IoC container manages one or more beans. These beans are created with the configuration metadata that you supply to the container (for example, in the form of XML <bean/> definitions).
> 
> Within the container itself, these bean definitions are represented as BeanDefinition objects, which contain (among other information) the following metadata:  
> ...
> 
> Typically, to specify the bean class to be constructed in the case where the container itself directly creates the bean by calling its constructor reflectively, somewhat equivalent to Java code with the new operator.
> 
> To specify the actual class containing the static factory method that is invoked to create the object, in the less common case where the container invokes a static factory method on a class to create the bean. The object type returned from the invocation of the static factory method may be the same class or another class entirely.
> 
> [링크](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class)

`Spring container`에서 bean은 `BeanDefinition` 객체들로 정의해두고 객체를 생성합니다.  
`Bean`생성시 `BeanDefinition` 정의에 따라 객체 생성에 대한 정보를 참조하며, 일반적으로 `리플렉션`을 통해 객체를 생성합니다.  


> **`Dependency injection`**
> 
> IoC is also known as dependency injection (DI).   
> 
> It is a process whereby objects define their dependencies (that is, the other objects they work with) only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method.  
> 
> The container then injects those dependencies when it creates the bean.   
> 
> This process is fundamentally the inverse (hence the name, Inversion of Control) of the bean itself controlling the instantiation or location of its dependencies by using direct construction of classes or a mechanism such as the Service Locator pattern.  
> [링크](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-introduction)

`Container`가 `Bean` 생성시 `Service-locator 패턴`과 비슷한 매커니즘을 통해 의존성을 주입하며 생성합니다.

즉, 객체를 사용하는곳에서 직접 생성해 객체간에 강한 결합도를 갖는것이 아닌 외부(`Container`)에서 생성된 객체를 주입 해 줌 으로서 결합도를 낮추는 효과가 있습니다.

그렇다면 위에서 알아본 `DI`가 Spring 내부에서는 어떤 과정을 통해 사용자가 생성한 객체들을 의존성을 주입하면서 생성 할 수 있는 걸까요?

---

### Spring bean initializing 더 살펴보기

아시다시피, Spring framework를 시작 하게되면 `Spring container`가 초기화되고 `ComponentScan`을 통해 정의해준 `base-package`에서 `Component`들을 찾아 `Bean`으로 등록하는 과정이 수행됩니다. 실제 어떤 코드를 통해 `Bean`이 생성되고 `DI`가 일어나는지 직접 디버깅을 해보도록 하겠습니다.  

먼저, `Scan`을 통해 Bean 타겟들을 찾아 `BeanDefinition`을 정의합니다. 

![1]({{ site.images | relative_url }}/posts/2021-05-23-spring-di-reflection/1.png)

이후, `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory` 클래스에서 BeanInstance 생성 메서드를 찾을 수 있습니다.  

![2]({{ site.images | relative_url }}/posts/2021-05-23-spring-di-reflection/2.png)

실제 `Bean`을 생성하는 코드는 `org.springframework.beans.BeanUtil`에 정의되어 있습니다.  

![3]({{ site.images | relative_url }}/posts/2021-05-23-spring-di-reflection/3.png)

코드를 보면, `Reflection`을 활용해 생성자 (`Constructor`)를 넘겨서 Bean을 생성해주고 있는것을 확인 할 수 있습니다. Spring 코드가 꽤나 복잡하게 짜여있어 리플렉션이 정확히 어떻게 활용되는지 이해하기 힘들수 있습니다. 

---

### 리플렉션을 활용한 예제 Sample Container

`SampleContainer` 예제코드를 통해 좀더 자세히 알아보도로 하겠습니다.

```java
// Inject.java

@Retention(RetentionPolicy.RUNTIME)
public @interface Inject
{
}
```

```java
// SampleContainer.java

public class SampleContainer
{

    public static <T> T getObject(Class<T> clazz)
    {
        T instance = createInstance(clazz);

        for (Field field : clazz.getDeclaredFields())
        {
            if (field.getDeclaredAnnotation(Inject.class) != null)
            {
                Object filedInstance = createInstance(field.getType());
                try
                {
                    field.setAccessible(true);
                    field.set(instance, filedInstance);
                }
                catch (IllegalAccessException e)
                {
                    throw new RuntimeException("fail to set field", e);
                }
            }
        }

        return instance;
    }

    private static <T> T createInstance(Class<T> clazz)
    {
        try
        {
            return clazz.getConstructor().newInstance();
        }
        catch (InstantiationException | InvocationTargetException | NoSuchMethodException | IllegalAccessException e)
        {
            throw new RuntimeException("fail to create instance", e);
        }
    }
}
```

```java
// SampleComponent.java

public class SampleComponent
{
    @Inject
    private SampleRepository sampleRepository;

    public void doSomething()
    {
        List<String> results = sampleRepository.findAll();

        for (String str : results)
        {
            System.out.println("result : " + str);
        }
    }
}

}
```

```java
// SampleRepository.java

public class SampleRepository
{
    public List<String> findAll()
    {
        return Arrays.asList("AA", "BB", "CC");
    }
}
```
```java
// Main.java

public class Main{

    public static void main(String[] args)
    {
        SampleComponent sampleComponent = SampleContainer.getObject(SampleComponent.class);
        sampleComponent.doSomething();
    }
}
```

`SampleContainer` 에서 `SampleComponent` 객체를 생성할때, 리플렉션을 통해 객체를 생성하고 `SampleRepository`의존객체 또한 리플렉션으로 넣어주는것을 확인하실수 있습니다.  
위 코드의 수행 결과 `SampleComponent`객체가 의존성을 주입받은 채로 정상적으로 생성되어 아래와 같이 원하던 결과를 얻을수 있음을 확인하실수 있습니다.

![4]({{ site.images | relative_url }}/posts/2021-05-23-spring-di-reflection/4.png)

위 코드는 `Field injection`을 구현한 코드로서, 아시다시피 Spring에서는 `Constructor injection`, `Setter injection`의 방식도 지원을 하고 있습니다.    

또한, 예제에서는 target object를 직접 지정을 해 주었지만 일반적으로 `Bean`생성 과 동시에 `DI`가 이루어 지기 때문에 `Component` Scan 이후 타겟 Object들 역시 `Reflection`을 통해 초기화 한 후 `DI`가 이루어 집니다.

위 코드를 조금 더 `Spring container`처럼 만들어 보겠습니다.

```java
// TaesComponent.java

@Retention(RetentionPolicy.RUNTIME)
public @interface TaesComponent
{
}
```

```java
// SampleComponent.java

@TaesComponent
public class SampleComponent
{
    @Inject
    private SampleRepository sampleRepository;

    public void doSomething()
    {
        List<String> results = sampleRepository.findAll();

        for (String str : results)
        {
            System.out.println("result : " + str);
        }
    }
}
```

```java
// SampleContainer.java


public class SampleContainer
{
    private Map<String, Object> beans = new HashMap<>();

    public SampleContainer()
    {
        initBeans();
    }

    private void initBeans()
    {
        for (Class<?> targetClazz : doScan())
        {
            Object obj = getObject(targetClazz);
            beans.put(targetClazz.getSimpleName(), obj);
        }
    }

    private Set<Class<?>> doScan()
    {
        Reflections reflections = new Reflections("com.taes.demo",
            new SubTypesScanner(false));

        Set<Class<?>> allClasses = reflections.getSubTypesOf(Object.class);

        return allClasses.stream()
            .filter(clazz -> clazz.isAnnotationPresent(TaesComponent.class))
            .collect(Collectors.toSet());
    }

    public Optional<Object> getBean(String beanName)
    {
        return Optional.ofNullable(beans.get(beanName));
    }

    private <T> T getObject(Class<T> clazz)
    {
        T instance = createInstance(clazz);

        for (Field field : clazz.getDeclaredFields())
        {
            if (field.getDeclaredAnnotation(Inject.class) != null)
            {
                Object filedInstance = createInstance(field.getType());
                try
                {
                    field.setAccessible(true);
                    field.set(instance, filedInstance);
                }
                catch (IllegalAccessException e)
                {
                    throw new RuntimeException("fail to set field", e);
                }
            }
        }

        return instance;
    }

    private <T> T createInstance(Class<T> clazz)
    {
        try
        {
            return clazz.getConstructor().newInstance();
        }
        catch (InstantiationException | InvocationTargetException | NoSuchMethodException | IllegalAccessException e)
        {
            throw new RuntimeException("fail to create instance", e);
        }
    }
}
```

```java
// Main.java

public class Main
{
    public static void main(String[] args)
    {
        SampleContainer container = new SampleContainer();

        SampleComponent sampleComponent = (SampleComponent) container.getBean("SampleComponent").get();
        sampleComponent.doSomething();
    }
}
```

`Spring container`와 비교하면 완벽하지는 않지만, `singleton`으로 관리되는 `Bean`을 생성하기 위한 과정 (ComponentScan -> Bean initialize -> getBean ...)을 대략적으로 구현해 보았습니다.  

위 코드를 통해 `Spring container`에서 `Bean`을 생성하는 전 과정에서 리플렉션이 어떻게 사용되고 있는지 파악하시는데 조금더 도움이 됬을거라 생각됩니다.

---

### 마무리

개발자들이 어플리케이션을 개발하는데에는 `리플렉션`이 직접 사용되는 일은 많이 없지만 `Spring framework`의 주요로직에서는 많이 사용되고 있습니다.  

예제코드에서는 `Dependency ordering`, `Injection 방법`, `Find bean rule`, `Bean Injection` 등 고려하지 않은 사항들이 많지만 `리플렉션`이 어떻게 사용되고 있는지에 대해서 이해하시는데 도움이 되셨기를 바랍니다.

---

### Reference

- [더 자바, 코드를 조작하는 다양한 방법](https://www.inflearn.com/course/the-java-code-manipulation#)