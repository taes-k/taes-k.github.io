---
layout: default
comments: true
title: (요령과 기본) 1.2 Spring은 어떻게 동작하는가?
parent: 요령과 기본
date: 2019.05.11
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

# 1. 당신은 왜  Spring을 사용 하는가?

---

## 1.2 Spring은 어떻게 동작하는가?
앞서서 스프링을 사용하는 이유에대해서 알아보았는데, 과연 스프링은 어떻게 동작하길래 그러한 특징들을 가지는것인지 알아보도록 하겠습니다.  

---

## 1.2.1 IoC Container

---

### IoC와 DI
스프링프레임워크를 말할때 IoC와 DI를 주요 핵심 개념으로 이야기합니다.  
먼저 IoC(Inversion of Control)는 서비스 제어의 흐름을 역전시켰다는 뜻으로, 개발자가 아닌 프레임워크가 흐름을 제어하는 주체가되어 필요할때 코드를 호출하며 사용하게됩니다.   

다음은 DI(Dependency Injection)를 알아보도록 합시다. 하나의 객체가 다른객체를 참조할때 객체간의 의존성을 갖는다고 말을합니다.  

```c
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
    this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```
  
위의 SimpleMovieLister 오브젝트는 생성자에서 MovieFinder 오브젝트를 받으면서 MovieFinder 오브젝트에 의존성을 갖게되었습니다. 의존성을 갖게되었다는 말은 하나의 객체의 변화에 다른객체에 영향을 끼칠수 있게 되었다는 뜻입니다. 위의 예시 오브젝트로 설명을 한다면, News 오브젝트가 변경되면 SportsNews 오브젝트와 기타 모든 News 오브젝트에 의존하는 모듈까지 함께 변경되어야 할 수 있습니다.  

스프링은 이러한 객체들간의 의존성을 줄이기위한 방법으로 IoC/DI를 하나로 묶어서 제공을 하고 있습니다. 객체 선언과정을 스프링 프레임워크가 미리 대신 해줌으로써(IoC) 그저 객체를 가져다가 (DI) 사용 할수 있게 해줍니다. 이 과정을 설명하기 위한 도식도를 스프링 공식문서에서는 제공하고 있습니다.  
  
<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/intro/1.png" style="height:250px;">
</div>
  
위의 도식도를 보시면 'Your Business Objects'를 'Configuratioin Metadata'를 통해 Spring Container에 등록시켜 본 시스템에 제공해주는 도식도 내용을 확인하실수 있습니다.  여기서 사용되는 Container가 바로 IoC Container 입니다.
  
---
  
### DI의 장점  

그렇다면, 프레임워크에서 의존성주입을 제공해줌으로써 어떤 장점이 있을까요?  

> Code is cleaner with the DI principle, and decoupling is more effective when objects are provided with their dependencies. The object does not look up its dependencies and does not know the location or class of the dependencies. As a result, your classes become easier to test, particularly when the dependencies are on interfaces or abstract base classes, which allow for stub or mock implementations to be used in unit tests.  

본문 : <https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-collaborators>  
  
- 코드가 깔끔해진다.  
- 객체들간의 종속성이 감소한다.  
- 재사용성이 증가한다.  
- 유닛 테스트가 쉬워진다.   

--- 

### DI를 하기위한 준비

스프링의 DI를 활용하기위해서는 DI를 사용할 객체를 프레임워크에 알려줘야 합니다. 위의 IoC/DI의 도식도에서 보여지는 'Configuration Metadata'의 설정과정인데요 이렇게 IoC Container에 등록되어지는 객체들을 'Bean'이라고 합니다.  
먼저 Bean 을 등록하기 위한 방법으로는 3가지 방법이 있습니다.  

- XML-Based configuration
- Annotation-based configuration : after Spring 2.5
- Java-based configuration : after Spring 3.0

현재 많은 개발자들이 사용하는 방법은 Java기반 설정방식이지만, 상황 혹은 취향에따라 어느방법을 쓰셔도 관계는 없습니다.

> Are annotations better than XML for configuring Spring?  
>  
> The introduction of annotation-based configuration raised the question of whether this approach is “better” than XML. The short answer is “it depends.” The long answer is that each approach has its pros and cons, and, usually, it is up to the developer to decide which strategy suits them better. Due to the way they are defined, annotations provide a lot of context in their declaration, leading to shorter and more concise configuration. However, XML excels at wiring up components without touching their source code or recompiling them. Some developers prefer having the wiring close to the source while others argue that annotated classes are no longer POJOs and, furthermore, that the configuration becomes decentralized and harder to control.  
>  
>  No matter the choice, Spring can accommodate both styles and even mix them together. It is worth pointing out that through its JavaConfig option, Spring lets annotations be used in a non-invasive way, without touching the target components source code and that, in terms of tooling, all configuration styles are supported by the Spring Tool Suite.  

본문 : <https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-annotation-config>  

예시를 통해 알아보기 위해 객체를 빈으로 등록시켜보도록 하겠습니다.   
```c
// Java-based Configuration

@Configuration
    public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```
```c
// XML-based Configuration

<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```
어떤 방식으로 설정을 하든 똑같이 bean으로 등록이 가능합니다. 위의 설정 과정을 통해 등록된 빈은 IoC Container를 생성할때 함께 생성되어집니다. 하지만 모든 bean들이 모두 Spring의 초기화 과정에서 등록되지는 않습니다. 해당내용은 다음 섹션에서 알아보도록 하겠습니다.   

---

### Bean

---

사실 저는 IoC와 DI에대해 잘 몰랐을때도 스프링에서 제공해주는 이 기능들을 잘 활용해서 사용해 왔습니다. 잘 모르고 썼지만 잘 썼던 요령들을 통해 실제 프로젝트에서 IoC/DI 가 어떻게 사용되는지 살펴보도록 하겠습니다.  
`Spring Web service - Controller, Service, Dao`

```c
@Configuration
public class DatasourceConfig { 

    @Autowired
    ApplicationContext applicationContext;

    @Primary
    @Bean(name="dataSource")
    @ConfigurationProperties(prefix = "spring.datasource") 
    public DataSource dataSource() { 
        return DataSourceBuilder.create().build(); 
    } 

    @Primary
    @Bean(name = "sqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory() throws Exception { 
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean(); 
        sqlSessionFactoryBean.setDataSource(dataSource()); 
        sqlSessionFactoryBean.setMapperLocations(applicationContext.getResources("classpath:mybatis/mapper/*/*-sql.xml")); 
   
        return sqlSessionFactoryBean.getObject(); 
    } 

    @Primary
    @Bean(name = "sqlSessionTemplate")
    public SqlSessionTemplate sqlSessionTemplate() throws Exception { 
        return new SqlSessionTemplate(sqlSessionFactory()); 
    } 



}
```


아직은 이코드들이 어떤 동작을 하는지 모르셔도 좋습니다. 다만, 



IoC Container의 역할
- Bean 객체 관리
- 


---
