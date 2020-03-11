---
layout: post
comments: true
title: Netty, Spring WebFlux의 동작 과정
tags: [java, tomcat, spring]
---


---

지난 포스팅에서는 Spring의 기본조합, Tomcat + SpringMVC 에서의 동작 처리과정을 알아보았습니다.  

2017년 Spring5가 공개된후 Spring은 기존 SpringMVC 이외에 Reactive를 위한 Spring WebFlux를 지원하고 있습니다. Spring WebFlux를 사용하게 되면 기존의 Tomcat 서버가 아닌 Netty 임베디드 서버를 지원하여 기존과 Tomcat과는 다른 동작 프로세스를 가지게 되는데 이에대해 알아보도록 하겠습니다.

http://wiki.sys4u.co.kr/pages/viewpage.action?pageId=8552586

--

### Netty  

Netty는 자바기반 논블록킹 네트워크 프레임워크.
웹서버의 역할을 합니다.



일반적으로 탐캣(Tomcat)은 'WAS(Web Application Server)'의 대표적인 미들웨어 서비스로 알려져있습니다.  

하지만 탐캣은 일반적으로 아파치 탐캣(Apache Tomcat)이라 불리며 회사명이자 웹서버의 대표적인 미들웨어인 아파치(Apache)의 기능 일부분을 가져와 함께 사용되면서 웹서버(Web Server)의 기능과 웹 애플리케이션 서버(Web Application Server) 모두를 포함하고 있다고 생각하셔도 무방합니다.

이번 포스팅에서는 개발자가 Spring으로 작성한 웹 프로그램이 Apache Tomcat을 이용해 웹 서비스로 등록되게 되면 어떤 프로세스로 Client의 요청을 처리하는지에 대해서 알아보도록 하겠습니다.

---

### Process

![1]({{ site.images | relative_url }}/posts/2020-02-16-servlet-container-spring-container/1.png)    

일반적으로 자바 웹프로그래밍을 할때 사용하는 Spring + Tomcat 조합으로 서비스를 올리게되면 위와같은 구조를 통해 클라이언트와 통신 하게 됩니다.  

---

### Web server

웹서버는 사용자가 웹 브라우저에서 URL 입력했을때 사용자에게 응답을 처리하는 http 통신의 일련의 과정을 진행합니다. 이 통신을 위해 소켓 연결 등의 네트워크 처리를 해주며 WAS와 비교하자면 html, css, js 등의 정적 소스에 대한 요청을 처리합니다.  

정적 소스를 별도로 처리하는 이유는 단순히 정적 파일만 전달 하면 되는 서비스에서 굳이 동적 컨텐츠를 처리하는 WAS에 부담을 줄 필요는 없기때문에 WAS와 업무를 나누어 처리하여 각 서비스의 목적에 따른 성능상의 장점을 살리기 위한 것으로 이해하시면 될 것 같습니다.  
  
또한 웹서버는 클라이언트와의 연결을 WAS로 전달하여 WAS가 클라이언트와 직접 통신하지 못하도록 중계역할을 해주어 WAS에게 클라이언트와의 연결에대한 독립성을 보장해주며, 보안적으로도 한단계 안전 할 수 있도록 해줍니다. 

Apache tomcat 5.5 이후 부터는 위에서 언급한대로 Web server의 기능인 httpd(웹서비스 데몬) native 모듈을 가지고와서 정적파일을 처리하기 때문에 별도의 Web server 기능에 뒤쳐지지 않는 정적파일 처리를 할 수 있습니다.

---

### Servlet

다음은 `Servlet container`를 알아보기 전에 `서블릿(Servlet)`에 대해서 알아보겠습니다.  

일반적으로 웹 프로그래밍을 한다고 하면 정의된 클라이언트의 요청에 대해 상응하는 결과를 return해 주어야 하는데 웹 페이지 혹은 결과값을 동적으로 생성 해 주기위한 역할을 하는 자바 프로그램을 서블릿 이라고 합니다.  

위와같이 자바 웹 프로그래밍을 하는데 있어서 반드시 필요한 역할을 하는 서블릿 이지만, 옛날 CGI(Common Gateway Interface)를 Java로 구현하기 위해 서블릿 프로그래밍을 하시던 분이 아니시라면 서블릿을 직접적으로 다루어 보신분은 적으실거라 생각됩니다. 

아래는 java8에서 제공하는 `Servlet 인터페이스` 입니다. 
```java
// javax.servlet.Servlet.java

public interface Servlet {

    public void init(ServletConfig config) throws ServletException;

    public ServletConfig getServletConfig();

    public void service(ServletRequest req, ServletResponse res)
            throws ServletException, IOException;

    public String getServletInfo();

    public void destroy();
}
```
위의 인터페이스를 보면 서블릿의 라이프사이클을 확인 할 수 있습니다.  

`init > service > destroy`  

javaEE로 웹서비스를 직접 구현할때에는 서블릿을 만들기 위해 위의 `Servlet 인터페이스`의 구현체를  직접 만들어 사용 했지만, 스프링MVC 에서는 `Dispatcher Servlet`이라는 모든 요청을 담당하는 서블릿을 두고 컨트롤러에 위임을 하여 요청을 처리합니다.  

![2]({{ site.images | relative_url }}/posts/2020-02-16-servlet-container-spring-container/2.png)  

이와같은 `프론트 컨트롤러 디자인 패턴`이 적용된 SpringMVC를 통해 개발자는 별도의 서블릿 개발없이, Controller의 구현만으로도 동적인 response를 클라이언트에게 줄 수 있습니다. 

---

### DispatcherServlet

다음은 Spring framework에 구현되어있는 DispatcherServlet.java 입니다.  

```java
// package org.springframework.web.servlet;
@SuppressWarnings("serial")
public class DispatcherServlet extends FrameworkServlet {
	
    ...

	public void setDetectAllHandlerMappings(boolean detectAllHandlerMappings) {
		this.detectAllHandlerMappings = detectAllHandlerMappings;
	}

	protected void initStrategies(ApplicationContext context) {
		initMultipartResolver(context);
		initLocaleResolver(context);
		initThemeResolver(context);
		initHandlerMappings(context);
		initHandlerAdapters(context);
		initHandlerExceptionResolvers(context);
		initRequestToViewNameTranslator(context);
		initViewResolvers(context);
		initFlashMapManager(context);
	}

	@Override
	protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
        ~~~
	}

	protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        ~~~
	}

    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException {
        ~~~
    }

    protected void service(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException {
        ~~~
    }

    ...
}


```
자세한 소스코드는 [이곳](https://github.com/spring-projects/spring-framework/blob/master/spring-webmvc/src/main/java/org/springframework/web/servlet/DispatcherServlet.java)에서 확인 하실수 있습니다.  

SpringMVC에서 제공해 주고 있는 `DispatcherServlet`은 [`FrameworkServlet.java` > `HttpServlet.java` > `Servlet.java`] 를 상속받아 구현한 서블릿입니다.  

위의 코드로 유추 하실 수 있듯이 클래스 내부에 여러 핸들러(Handler), 어댑터(Adapter), 리졸버(Resolver) 등을 가지고 클라이언트의 요청에 따라 개발자가 정의해 둔 내용을 응답 해 줄수있도록 `front-controller`의 역할을 하고 있음을 확인 하실수 있으실 겁니다. 그중 주요 역할을하는 몇가지를 살펴보고 가겠습니다.  

1. `HandlerMapping`  
Client로 부터 들어온 Request를 분석하여 매핑된 Controller가 있는지 확인합니다.  

2. `HandlerAdapter`  
매핑 대상 Controller에게 Request 처리요청을 보냅니다.  

3. `ViewResolver`  
Controller에서 view를 return 했을경우 해당하는 view를 찾아 client에게 return 합니다.


![3]({{ site.images | relative_url }}/posts/2020-02-16-servlet-container-spring-container/3.png)  

---

### Servlet container

탐캣의 메인 기능이라고 할 수 있는 `서블릿 컨테이너`(Servlet container)의 역할을 알아보독 하겠습니다. 첫째로 서블릿 컨테이너라는 말 그대로 서블릿을 관리하는 역할을 하게 됩니다. 위의 `서블릿`을 알아보면서 서블릿 라이프사이클(
`init > service > destroy`)을 알아보았는데 서블릿 클래스의 로드, 초기화, 호출, 소멸까지의 라이프사이클을 직접적으로 관리해주는 역할을 하는것이 바로 서블릿컨테이너입니다.  서블릿으로 구현된 `DispatcherServlet` 역시 서블릿 컨테이너에서 수행됩니다.

둘째, 상단 Process 그림에서 보셨던것 처럼 서블릿 컨테이너는 웹서버와 통신을 통해 클라이언트의 request를 전달받아 동적 서비스를 response를 해야하는데, 해당 통신을 위해 소켓을 만드는등의 역할을 진행합니다.   

셋째, 클라이언트로부터 request를 받을때마다 쓰레드를 생성해 요청을 처리합니다. 해당 쓰레드는 서블릿 컨테이너에서 쓰레드풀을 별도로 관리하여 실행하게됩니다.


---

### Spring container

![4]({{ site.images | relative_url }}/posts/2020-02-16-servlet-container-spring-container/4.png)    

위 그림은 Spring Document에서 제공하는 이해를 위한 그림 입니다.  `DispatcherServlet` 내부에 `Servlet WebApplicationContext`와 `Root WebApplicationContext`가 동작하는것으로 보이지만 이 두 ApplicationContext가 바로 process 그림에서 보셨던 `스프링 컨테이너`(Spring container) 에서 동작하는 컨텍스트라고 이해해주시면 될 것 같습니다.

서블릿 컨테이너는 서블릿의 생명주기를 관리했다면, 스프링컨테이너는 Java object인 `빈`(Bean)의 라이프 사이클 관리하여 Spring 프레임워크의 특징인 `IOC(제어역전)`와 `DI(의존성주입)`을 제공해주는 역할을 합니다.  

---

### 총정리

위의 내용들을 바탕으로 클라이언트의 request부터 response 받는 흐름을 총정리 해보도록 하겠습니다.
   
![6]({{ site.images | relative_url }}/posts/2020-02-16-servlet-container-spring-container/6.png)  


먼저, Tomcat을 실행하면 로그를 통해 서비스를 위해 어떤 순서로 세팅이 이루어지는지 확인 하실수 있습니다. (Debug log)

![5]({{ site.images | relative_url }}/posts/2020-02-16-servlet-container-spring-container/5.png) 

*Server start 단계*  

> 1. Web server init
> 2. Root WebApplicationContext 로딩
> 3. Web server start 



*Client 호출 단계*  

> 1. Client -> Web server 으로 request 보냄
> 2. 동적 Web server -> Servlet container로 전달
> 3. Servlet container 쓰레드 생성
> 4. DispatcherServlet init (서블릿 생성 안되어 있을경우) 
> 5. 생성된 쓰레드에서 DispatcherServlet service() 메서드 호출 
> 6. HandlerMapping을 통해 매핑 컨트롤러 조회
> 7. HandlerAdapter를 통해 매핑 컨트롤러에 request 전달
> 8. 개발자가 구현한 Controller -> Service -> Repository ... 동작

---

### 마무리

Spring 이라는 프레임워크가 매우 잘 만들어져있는 탓에, 내부적으로 진행되는 프로세스에 대한 내용들을 전혀 모르더라도 매우 훌륭한 프로그래밍을 해 오셨을거라 생각합니다.  

사실 스프링 개발팀의 목적 또한 스프링을 사용하는 개발자가 서비스 로직에만 집중 할 수 있도록 실행 로직들이 내부적으로 동작 하게 만들었기에 해당 내용을 처음 접하신 독자분들도 계실텐데, 내가 작성한 코드가 서버에서 어떤 과정을 통해 동작하는지에 대해서 한번더 생각 할 수 있는 기회가 되었기를 희망합니다. 

---