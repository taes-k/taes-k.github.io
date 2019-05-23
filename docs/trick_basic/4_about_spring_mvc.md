---
layout: default
comments: true
title: (요령과 기본) 1.4 Spring MVC의 진짜 의미
parent: 요령과 기본
date: 2019.05.15
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### 1.4 Spring MVC의 진짜 의미
{: .no_toc }
Spring Web MVC는 Spring 프레임워크가 처음 나왔을때부터 함께나온 웹 프레임워크로 spring-webmvc 소스 모듈의 이름을 따온 모델 입니다. 실제  MVC 모델이 어떻게 구성되어 있고 어떤 과정을 통해 클라이언트의 request 를 처리하는지 알아보도록 하겠습니다. 


## Contents List
{: .no_toc .text-delta }

1. TOC
{:toc}


---

### <1.4.1 MVC의 이해>
### MVC
 먼저 MVC란 Model - View - Controller의 약자로써, 구글에 MVC에 대해 검색하면 일반적으로 다음과같은 다이어그램을통해 설명하고 있습니다.  

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/about_spring_mvc/mvc1.png" style="height:250px;">
</div>

위 다이어그램은 Spring MVC에서 실제로 운영되는 프로세스와는 조금 다르지만 MVC각자의 역할을 설명하기 위한 그림으로 이해해 주시면 좋을것 같습니다.  

MVC는 위 다이어그램과 같이 비지니스 처리 로직과 사용자 인터페이스 요소를 분리하여 개발 및 운영에 있어 유연성이 높은 장점을 가지고 있습니다. 

---

### MVC - Model
어떤 데이터를 보여줄지 정의해 줍니다.  
Model은 '데이터'를 정의하는 부분으로, 데이터를 조회하는 것 뿐만 아니라 삽입, 변경, 삭제등의 작업을 수행하고 그 실제적인 과정이 일어나는 비즈니스 로직 전반적인 부분을 정의해 줍니다.    

---

### MVC - View
데이터의 보여주는 방식을 정의해 줍니다.   
View는 Client들이 실제로 보는 화면을 표시해주는 부분으로, 보여줄 데이터나 텍스트 데이터를 표시해주거나 Client들의 입력을 받기위한 입력 폼들을 정의해 줍니다.

---

### MVC - Controller
Client의 Request에 따른 Response를 정의해 주며 모델 및 뷰를 업데이트 하는 로직을 포함하고 있습니다.  
Controller는 Client의 요청을 받아들여 각 요청에따라 데이터를 설정해주거나 화면을 출력해주는 등의 전체적인 연결을 관리해주는 응답을 정의해 줍니다.
 


--- 

### <1.4.2 Spring MVC Process>
위 섹션을 통해 MVC의 정의와 개념에대해서 알아보았습니다. 그렇다면 Spring MVC에서 Client의 Request를 처리하는 과정을 알아보면서 실제로 MVC가 어떤 Process로 동작하는지 알아보도록 하겠습니다.

### DispatcherServlet
가장먼저, Spring MVC는 하나의 컨트롤러로 Client들의 Request를 처리해주는 패턴인 Front controller pattern으로 구성되어있습니다. Spring에서 사용되는 Front controller가 바로 `DispatcherServlet` 으로, 요청을 처리하기위한 공유 알고리즘을 제공하지만 실제작업은 적절한 컴포넌트들에게 위임하여 수행하게 됩니다.   
  
DispatcherServlet의 구성은 다음과 같습니다.

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/about_spring_mvc/dispatcherServlet.png" style="height:350px;">
</div>
  
위 모식도를 확인해보면 이전, '스프링은 어떻게 동작하는가?' 챕터에서 알아본 어플리케이션 컨텍스트가  DispatcherServlet에 위치하게 됩니다. 어플리케이션 컨텍스트의 경우 Dispatcher 서버에서 생성되는 Servlet-Context와  ContextLoaderListner에서 생성되는 Root-Context 2가지 종류가 있습니다. 두 컨텍스간의 관계를 나타내어 주는것이 바로 위의 모식도로써, 계층형으로 구성되어 Bean들이 종류에 따라 각각 다른 위치에 존재하는것을 확인 할 수 있습니다.  

***Root Context***   

- ContextLoaderListner에서 생성 
- Spring 최상단에 위치한 Context  
- 공통적인 기능들을 가진 Bean을 생성하여 다른 Servlet Context에서 참조가 가능합니다. (@Service, Repository, @Configuration, @Component)    
  
***Servlet Context***  

- DispatcherServlet에서 생성
- Servlet 단위로 생성되는 Context  
- 서블릿 내에서만 사용하는 Bean 생성 (@Controller, Interceptor) 
  
DispatcherServlet은 설정에 따라 여러개 생성이 가능하며 url pattern 을 통해 목적에 맞게 나누어 관리합니다. 예를들자면 웹 페이지를 위한 서블릿과 Rest 기반 웹서비스를 따로 관리하기위한 DispatcherServlet을 생성할수 있고 생성한 DispatcherServlet 마다 Servlet Context가 생성됩니다.  
  
Bean을 찾을때 Servlet Context를 먼저 스캔 후 Root Context를 스캔하여 Bean을 찾게되며 Servlet Context에서는 Root Context의 Bean을 참조 할수 있지만 반대로는 불가능 합니다.    

위와 같은 이유로 참조가 불가능한 컨텍스트(Root Context-> Servlet Context , Servlet Context -> 다른 Servlet Context)에 설정된 Bean을 참조하려고 하면 `Not found bean` 에러가 날수도 있으니 본인이 사용하고자 하는 Bean의 목적과 저장위치를 잘 알고 사용 하실수 있기를 바랍니다.  

--- 

### Special Bean Type
DispatcherServlet은 특수한 요청을 처리하기위한 Bean들이 있습니다. 그중, 위의 모식도에서 Servlet Context가 가지고 있는 ViewResolver와 HandlerMapping Bean 의 역할에대해서 알아보도록 하겠습니다.  
  
***HadlerMapping***

요청에따른 pre- ,post- 처리를위한 interceptor 핸들링을 위한 Bean 으로써 RequestMappingHandlerMapping (@RequestMapping), SimpleUrlHandlerMapping (URI path patterns handler)가 포함되어 있습니다. 위 기능들을 통해 Controller에서 URI 매핑이 가능합니다.

***ViewResolver***  

String 기반 view 이름 검색을 통해 파일을 찾아 DispatcherServlet에 전달해주는 기능을 합니다.

---
### DispatcherServlet Request 처리과정

위에서 정리해본 DispatcherServlet의 역할을 토대로 request처리 플로우를 그려보면 다음과 같습니다.  

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/about_spring_mvc/dispatcherServletProcess.png" style="height:350px;">
</div>
  
Spring MVC에서는 Dispatcher Servlet의 Special Bean들을 통해 Request들을 핸들링 해주어 개발자가 MVC에 필요한 코드만 작성 할수 있도록 지원 하고 있음을 볼 수 있습니다. 

---

### 실제 MVC 프로젝트 구성

사실 제가 처음 Spring 을 접하여 웹 프로젝트를 진행하였을때는 웹어플리케이션 서블릿에대한 지식이 전혀 없었는데요, 그저 어떻게든 돌아가는 서비스를 만들기위해 구글링을 해서 다른분들의 설정들을 그대로 따라 작성하면서 프로젝트를 구성했었습니다. 이렇게 따라하는 요령으로 잘 구동되는 서비스를 만들긴 했으나 기본을 제대로 모르다 보니 나중에 Bean이 설정이 꼬여 참조가 안되는데도 원인을 알수 없었습니다.  
  
그렇다면 예전에 제가 직접 구현했던 서블릿 설정 소스를 통해 어떤 설정 역할을 하는지 알아보도록 하겠습니다.  
```java
    <context:component-scan base-package="controller" use-default-filters="false">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!-- Enables the Spring MVC @Controller programming model -->
    <mvc:annotation-driven>
        <mvc:message-converters register-defaults="true">
            <bean class = "org.springframework.http.converter.json.MappingJacksonHttpMessageConverter">
                <property name="supportedMediaTypes" value = "text/plain;charset=UTF-8" />
            </bean>
        </mvc:message-converters>        
    </mvc:annotation-driven>
    
    <!-- Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views directory -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/" />
        <property name="suffix" value=".jsp" />
        <property name="contentType" value="text/html; charset=UTF-8"/>
    </bean>
```

`<context:component-scan>` : component-scan을 통해 특정 패키지 내의 클래스를 스캔하여 어노테이션을 확인해 Bean을 설정해 줍니다. 위 설정에서는 'controller' 패키지 내 stereotype.Controller annotation(@Controller)를 스캔하여 Bean 설정을 해줍니다.  
`<mvc:annotation-driven>`  : 스프링 MVC를 위한 Special Bean들을 설정해주는 태그로, Handler mapping에 대한  Bean들을 설정해줍니다. 위 설정에서는 핸들러 어댑터등의 빈을 설정 해주고 'message-converter' 태그 를 통해 @ResponseBody 어노테이션에 대하여 response를 UTF-8 charset으로 JSON 형식 결과값을 보내줄수 있도록 설정해 줍니다.  
`<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">` : 해당 클래스는 클래스이름에서 볼수 있듯이 위에서 DispatcherServlet구조에서 알아보았던 Viewresolver 클래스로써, view 파일 설정을 위한 태그입니다. 위 설정에서는 /WEB-INF/jsp/ 폴더내의 .jsp로 끝나는 파일을 String기반 파일 이름으로 서치가 가능하도록 설정해 줍니다.  
  
위 소스의 경우 Spring3.1 버젼의 MVC 프로젝트에서 사용한 소스입니다. 해당 프로젝트의 경우 위와같이 모두 설정을 해주어야 했습니다. 그렇다면 최신 버전에서 MVC 프로젝트를 구성하는 예제를 만들어 보도록 하겠습니다.  
  
- 구동환경  
Spring boot 2.1.5  
  
- 디렉토리 구조 

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/about_spring_mvc/mvc_directory.png" style="height:400px; border:1px solid #d0d0d0;">
</div>  


- 소스   

1\. Controller 구현  

```java
//example.mvc.spring.controller.ExampleController

@Controller
public class ExampleController {

    @Autowired
    private ExampleService exampleService;

    @RequestMapping(value="/example", method = RequestMethod.GET)
    public String example(Model model) {
        model.addAttribute("data",exampleService.getExample());

        return "example";
    }
}
```  
  
2\. Service 구현  

```java
//example.mvc.spring.service.ExampleService

@Service
public class ExampleService {

    public List getExample() {
        List result = new ArrayList();

        result.add("Example1");
        result.add("Example2");
        result.add("Example3");
        result.add("Example4");

        return result;
    }
}

```  

3\. View 설정  

```c
//application.properties

spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
```

```c
// pom.xml

<!-- JSP를 위한 dependency 추가 -->
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
</dependency>
```   
  
4\. View 구현  

```c
<%@ page language="java" contentType="text/html; charset=UTF-8"
pageEncoding="UTF-8"%>

<!DOCTYPE html>

<html>
<head></head>
<body>
    [Example Data List]<br>
    ${data }
</body>
</html>
```

- 결과  
<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/about_spring_mvc/example_mvc_run.png" style="height:350px; border:1px solid #d0d0d0;">
</div>  
  
    
Springboot 프로젝트에서는  통해 직접 작성한 소스 50줄 정도로 MVC 웹 프로젝트를 구현했습니다.
<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/about_spring_mvc/spring_boot_init.png" style="height:250px; border:1px solid #d0d0d0;">
</div>  
  
  
 위의 프로젝트 스타트 로그를 확인해보면 스프링부트에서는 기본적으로  default 서블릿을 구성하여 Annotation 설정을 자동으로 해주어 간단한 프로젝트에서는 따로 설정을 하지 않아도 웹 MVC 프로젝트 구현이 가능합니다.  물론, 따로 `WebApplicationInitializer` 를 구현하여 서블릿 설정을 할 수 있습니다.
   
 ```java
 
public class MyWebApplicationInitializer implements WebApplicationInitializer {

@Override
public void onStartup(ServletContext servletCxt) {
     // Load Spring web application configuration
     AnnotationConfigWebApplicationContext ac = new AnnotationConfigWebApplicationContext();
     ac.register(AppConfig.class);
     ac.refresh();
     
     // Create and register the DispatcherServlet
     DispatcherServlet servlet = new DispatcherServlet(ac);
     ServletRegistration.Dynamic registration = servletCxt.addServlet("app", servlet);
     registration.setLoadOnStartup(1);
     registration.addMapping("/app/*");
     }
}
```  
  
  
- 샘플 코드
<https://github.com/taes-k/spring-example/tree/master/spring-mvc-example>
  
--- 

### <마무리>

Spring MVC 구조에서 실제로 request를 어떻게 처리하는지 알아보았습니다. MVC구조가 단순히 Model, View, Controller 로써 구성된다는것 뿐만아니라 DispatcherServlet을 통해 어떠한 프로세스로 동작하고 컨텍스트의 계층구조까지 이해하고 개발을 할 수 있기를 바랍니다.  

--- 

### 참조 문서
{: .no_toc }
Spring 5.1.6 release docs  
<https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc>


---
