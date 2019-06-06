---
layout: default
comments: true
title: (요령과 기본) 1.1 Spring을 사용하는 이유
parent: 요령과 기본
date: 2019.05.11
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

## 1. 당신은 왜  Spring을 사용 하는가?
{: .no_toc}
이글을 읽고 계신 독자들의 대부분은 스프링을 이미 사용하고 계신 분들일 거라 생각합니다. 그렇다면 '나는 왜 스프링을 사용하게 되었는가?' 에 대한 질문에 답을 해 보시기 바랍니다.  
  
저의 답은 '스타트업 창업을 했을때 내가 할수있는건 자바였고, 우리는 웹 서비스를 만들어야 했기 때문에 자연스럽게 스프링을 사용하게 되었다' 입니다. 대부분은 대학 혹은 학원에서 스프링을 배웠거나 회사에서 이미 스프링을 사용하고 있었기에 자연스럽게 스프링을 사용하게 되셨을거라 생각합니다.   
    
사실 제가 스프링에 입문할때는 Django나 Node.js 등의 선택권이 있었지만, 단순히 자바를 사용하고자 스프링을 선택하게 되었습니다. 저와 같이 그저 자바기반으로 개발을 하고자 스프링을 사용하게 되셨거나 반강제적으로 스프링에 입문하신 분들의 경우 스프링에대해 잘 알지 못하고 프로젝트 생성하는법 부터 공부하셨던 분들이 많을것 같습니다. 그래서 먼저 스프링에대해 알아보고자 합니다.

---

## 1.1 Spring을 사용하는 이유

'스프링을 왜 사용하는가?'에 대한 답변을 위해 Spring을 사용하는 이유를 알아보도록 하겠습니다.  

## Contents List
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Spring이 생겨난 이유  

우리가 스프링을 사용하는 이유는 많은 기업들이 스프링을 사용하기때문일것입니다. 그렇다면, 많은 기업들은 왜 스프링을 사용하는지 알아보기위해 스프링에서 말하는 스프링을 사용해야하는 이유를 알아보도록 합시다. 스프링을 사용해야 하는 이유는 아마도 스프링이 생겨난 이유와도 같을것입니다. 스프링이 생겨난 목적을 알아보기위해 초기 스프링 공식문서를 살펴보았습니다. (1.1.1 Realease)
> Developing software applications is hard enough even with good tools and technologies. Implementing applications using platforms which promise everything but turn out to be heavy-weight, hard to control and not very efficient during the development cycle makes it even harder. Spring provides a light-weight solution for building enterprise-ready applications, while still supporting the possibility of using declarative transaction management, remote access to your logic using RMI or webservices, mailing facilities and various options in persisting your data to a database. Spring provides an MVC framework, transparent ways of integrating AOP into your software and a well-structured exception hierarchy including automatic mapping from proprietary exception hierarchies.  
>  
> Spring could potentially be a one-stop-shop for all your enterprise applications, however, Spring is modular, allowing you to use parts of it, without having to bring in the rest. You can use the bean container, with Struts on top, but you could also choose to just use the Hibernate integration or the JDBC abstraction layer. Spring is non-intrusive, meaning dependencies on the framework are generally none or absolutely minimal, depending on the area of use..    
>  
> [원문보기](https://docs.spring.io/spring/docs/1.1.1/spring-reference.pdf)  
  

위 초기 스프링 문서를 통해 알수있는 스프링이 생겨난 이유들을 정리해본다면 다음과 같습니다.  

- Spring은 애플리케이션을 구축하기위한 경량화된 솔루션을 제공하여 효율적인 개발 환경을 제공한다.
- Spring은 MVC 패턴, AOP, 잘 구조화된 예외계층구조를 제공한다.
- Spring은 많은 기능들을 제공 하지만 모듈화되어 있어 필요한 모듈들을 선택적으로 사용 할 수 있다.
  
---
  
## 스프링이 말하는 스프링 

위에서 소개하는 내용들은 사실 지금에와서는 대부분의 프레임워크들이 가지고 있는 특징들 일 수 있습니다. 그렇다면 최신버전에서 소개하는 스프링은 어떨까요? 최신버전 스프링 공식문서에서 소개하는 스프링 프레임워크는 다음과 같습니다. (5.1.7 Realease)
> Spring makes it easy to create Java enterprise applications. It provides everything you need to embrace the Java language in an enterprise environment, with support for Groovy and Kotlin as alternative languages on the JVM, and with the flexibility to create many kinds of architectures depending on an application’s needs  
>  
> Spring supports a wide range of application scenarios. In a large enterprise, applications often exist for a long time and have to run on a JDK and application server whose upgrade cycle is beyond developer control. Others may run as a single jar with the server embedded, possibly in a cloud environment. Yet others may be standalone applications (such as batch or integration workloads) that do not need a server.  
>  
> Spring is open source. It has a large and active community that provides continuous feedback based on a diverse range of real-world use cases. This has helped Spring to successfully evolve over a very long time.  
>  
> [원문보기](https://docs.spring.io/spring/docs/current/spring-framework-reference/overview.html#overview-spring)  
  
  
- 스프링은 자바 엔터프라이즈 애플리케이션 생성을 쉽게 해준다.  
- 그루비, 코틀링 등 JVM내의 대체언어들 까지 지원이 가능하다.
- 만들고자 하는 애플리케이션의 목적에따라 유연하게 아키텍쳐를 구성 할 수 있다.
- 스프링은 오픈소스로써, 오랜시간동안 성공적인 프레임워크로써 진화해 왔다.
  
  
스프링 공식문서에서는 프레임워크를 공부할때 해당 프레임워크를 따르는 원칙을 함께 알아야된다고 명시하고 있습니다. 다음은 스프링에서 소개하는 스프링 프레임워크의 개발 철학입니다.  
> - Provide choice at every level. Spring lets you defer design decisions as late as possible. For example, you can switch persistence providers through configuration without changing your code. The same is true for many other infrastructure concerns and integration with third-party APIs.  
>  
>  - Accommodate diverse perspectives. Spring embraces flexibility and is not opinionated about how things should be done. It supports a wide range of application needs with different perspectives.  
>  
>  - Maintain strong backward compatibility. Spring’s evolution has been carefully managed to force few breaking changes between versions. Spring supports a carefully chosen range of JDK versions and third-party libraries to facilitate maintenance of applications and libraries that depend on Spring.  
>  
> - Care about API design. The Spring team puts a lot of thought and time into making APIs that are intuitive and that hold up across many versions and many years.  
>  
> - Set high standards for code quality. The Spring Framework puts a strong emphasis on meaningful, current, and accurate javadoc. It is one of very few projects that can claim clean code structure with no circular dependencies between packages.   
>  
> [원문보기](https://docs.spring.io/spring/docs/current/spring-framework-reference/overview.html#overview-history)  
  
  
- 코드의 변경없이 환경설정만으로 provider를 변경 할 수 있다.
- 다양한 애플리케이션의 요구사항들을 수용한다.
- 이전버전들과의 호환성이 강력하게 유지된다.
- 직관적인 API를 제공한다.
- 깔끔한 코드 구조를 만들기위한 높은 기준이 설정되어 있다.  

---

## <마무리>
위에서 스프링에서 소개하는 스프링에대해 알아보았는데 스프링을 사용하게되는 목적 요소들을 종합해보자면  다음과 같을것입니다.  
- 스프링은 자바 및 JVM 환경의 대체언어들의 효율적이고 쉬운 엔터프라이즈 애플리케이션 개발 환경을 제공한다.  
- 스프링은 만들고자하는 애플리케이션의 요구사항과 목적에 따라 유연하게 적용시킬수 있습니다.
- 스프링은 패키지들간의 순환 의존성이없는 깨끗한 프로젝트구성을 만들수 있다.
- 스프링은 직관적인 API를 제공한다.
- 스프링은 오픈소스로써 지속적인 업데이트가 되고 있으며 이전 버전들에대한 강력한 호환성을 지원한다.  
  
물론 위에서 정리한 내용들 이외에도 많은 이유들이 있겠지만 스프링 공식문서에서 이야기 하고 있는 스프링을 사용해야하는 이유들을 정리하면서 내가 스프링을 사용하는 이유에대해서 한번쯤 생각해보길 바랍니다.
  
  
  ---
  
## 참조 문서
{: .no_toc}
Spring 5.1.6 release docs  
<https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-introduction>
  
---
