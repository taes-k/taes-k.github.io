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

### 1.4.1 MVC의 이해
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

### 1.4.2 Spring MVC Process
위 섹션을 통해 MVC의 정의와 개념에대해서 알아보았습니다. 그렇다면 Spring MVC에서 Client의 Request를 처리하는 과정을 알아보면서 실제로 MVC가 어떤 Process로 동작하는지 알아보도록 하겠습니다.

### DispatcherServlet
가장먼저, Spring MVC는 하나의 컨트롤러로 Client들의 Request를 처리해주는 패턴인 Front controller pattern으로 구성되어있습니다. Spring에서 사용되는 Front controller가 바로 `DispatcherServlet` 으로, 요청을 처리하기위한 공유 알고리즘을 제공하지만 실제작업은 적절한 컴포넌트들에게 위임하여 수행하게 됩니다.   
  
DispatcherServlet의 구성은 다음과 같습니다.

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/about_spring_mvc/dispatcherServlet.png" style="height:250px;">
</div>
  
위 모식도를 확인해보면 이전, '스프링은 어떻게 동작하는가?' 챕터에서 알아본 어플리케이션 컨텍스트가  DispatcherServlet에 위치하게 됩니다. 어플리케이션 컨텍스트의 경우 Dispatcher 서버에서 생성되는 Servlet-Context와  ContextLoaderListner에서 생성되는 Root-Context 2가지 종류가 있습니다. 두 컨텍스간의 관계를 나타내어 주는것이 바로 위의 모식도로써, 계층형으로 구성되어 Bean들이 종류에 따라 각각 다른 위치에 존재하는것을 확인 할 수 있습니다.  

**Root Context**   

- ContextLoaderListner에서 생성 
- Spring 최상단에 위치한 Context  
- 공통적인 기능들을 가진 Bean을 생성하여 다른 Servlet Context에서 참조가 가능합니다. (@Service, Repository, @Configuration, @Component)    
  
**Servlet Context**  

- DispatcherServlet에서 생성
- Servlet 단위로 생성되는 Context  
- 서블릿 내에서만 사용하는 Bean 생성 (@Controller, Interceptor) 

Bean을 찾을때 Servlet Context를 먼저 스캔 후 Root Context를 스캔하여 Bean을 찾게되며 Servlet Context에서는 Root Context의 Bean을 참조 할수 있지만 반대는 불가능 합니다.   
위와 같은 이유로 @Autowired로 Bean을 불러 사용 하려해도 `Not found bean` 에러가 날수도 있으니 본인이 사용하고자 하는 Bean의 목적과 저장위치를 잘 알고 사용 하실수 있기를 바랍니다.  

--- 

### Special Bean Type
DispatcherServlet은 특수한 요청을 처리하기위한 Bean들이 있습니다. 그중, 위의 모식도에서 Servlet Context가 가지고 있는 ViewResolver와 HandlerMapping Bean 의 역할에대해서 알아보도록 하겠습니다.  
  
**HadlerMapping**  

요청에따른 pre- ,post- 처리를위한 interceptor 핸들링을 위한 Bean 으로써 RequestMappingHandlerMapping (@RequestMapping), SimpleUrlHandlerMapping (URI path patterns handler)가 포함되어 있습니다. 위 기능들을 통해 Controller에서 URI 매핑이 가능합니다.

**ViewResolver**  

String 기반 view 이름 검색을 통해 파일을 찾아 DispatcherServlet에 전달해주는 기능을 합니다.

---
### DispatcherServlet Request 처리과정


<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/about_spring_mvc/dispatcherServletProcess.png" style="height:250px;">
</div>

---

### MVC 구성해보기

---


--- 

### 마무리


--- 

### 참조 문서
{: .no_toc }
Spring 5.1.6 release docs  
<https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-introduction>


---
