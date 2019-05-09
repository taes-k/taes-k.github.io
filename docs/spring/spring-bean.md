---
layout: default
comments: true
title: Spring Bean
parent: spring
date: 2019.05.09
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### Spring Bean 
'Bean (빈)'이란 Spring 공식문서에서 다음과 같이 정의하고있다.  

> In Spring, the objects that form the backbone of your application and that are managed by the Spring IoC container are called beans. A bean is an object that is instantiated, assembled, and otherwise managed by a Spring IoC container Otherwise, a bean is simply one of many objects in your application. Beans, and the dependencies among them, are reflected in the configuration metadata used by a container.
  
스프링의 특징중 하나인 'IoC'의 기본 개념으로, 개발의 흐름을 개발자가아닌 스프링 프레임워크가 주도하게 되는데 이때 스프링 IoC Container는 객체를 생성하고 객체간 의존성을 이어주는 역할을 한다.  Bean은 자바에서 사용하는 다른 어떤 객체들과 같은 객체이나, 다른점이 있다면 IOC Container에 의해 생성및 관리되는 객체를 의미한다.  

그렇다면, 왜 Bean을 사용할까?  
이를 알기위해 스프링의 또다른 특징중 하나인 'DI'에대해 알아볼 필요가 있다. 

'Dependency Injection (DI)'를 Spring 공식문서에서는 다음과 같이 이야기 하고 있다.
> Dependency injection (DI) is a process whereby objects define their dependencies (that is, the other objects with which they work) only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method. The container then injects those dependencies when it creates the bean. This process is fundamentally the inverse (hence the name, Inversion of Control) of the bean itself controlling the instantiation or location of its dependencies on its own by using direct construction of classes or the Service Locator pattern.
> Code is cleaner with the DI principle, and decoupling is more effective when objects are provided with their dependencies. The object does not look up its dependencies and does not know the location or class of the dependencies. As a result, your classes become easier to test, particularly when the dependencies are on interfaces or abstract base classes, which allow for stub or mock implementations to be used in unit tests  
  
글이 길지만 눈여겨봐야할 부분만 본다면, 
> 1. The container then injects those dependencies when it creates the bean.
> 2. This process is fundamentally the inverse (hence the name, Inversion of Control) of the bean itself controlling the instantiation or location of its dependencies on its own by using direct construction of classes or the Service Locator pattern.
> 3. decoupling is more effective when objects are provided with their dependencies
  
1. IoC 컨테이너는 빈을 생성할때 의존성을 주입한다.
2. DI Process는 bean 자체 인스턴스화의 제어 역전, 종속 위치의 역전으로 서비스 중재자 패턴을 의미한다.
3. 의존성을 제공받을때, 결합도를 낮추는데 더욱 효과적이다.

결국, 객체를 Bean 으로 등록시켜 컴포넌트간의 직접적인 연결(종속)을 시키지 않고 제3의 컨테이너를 통해 만들어 진 객체를 사용하고자 하는 컴포넌트에 주입시킴으로써, 컴포넌트들 간의 결합도를 낮추어 주기 위함이다.  
또한, 싱글톤으로 한번의 객체생성을 통해 재사용하기 위함에도 목적이 있다.

---

### Spring Bean 등록
- xml을 이용한 등록
현재는 잘 안쓰이는 방법이지만 `applicationContext.xml` xml 파일에 빈을 등록시켜 사용 하는 방법

```c
//ThingOne.java

package x.y;

    public class ThingOne {

    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```  
```c
//applicationContext.xml
<beans>
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>

    <bean id="beanTwo" class="x.y.ThingTwo"/>

    <bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```  

- Annotation을 이용한 등록
```c
//ThingOne.java

@Configuration
public class ThingOne {
    @Autowired 
    ThingTwo thingTwo;
    
    @Autowired
    ThingThree thingThree;

    @Bean
    public ThingOne() {
        // ...
    }
}
```  

---

### Spring Bean Scope

|:--:|:--:|
|singleton|f|


### 참조 문서
Spring 5.1.6 release docs  
(https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-introduction)
