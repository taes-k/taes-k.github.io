---
layout: default
comments: true
title: (요령과 기본) 3.2 Spring Tomcat
parent: 요령과 기본
date: 2019.05.26
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### 3.2 Spring Tomcat
{: .no_toc }
Spring boot에서는 내장 웹 어플리케이션 서버로 Apache Tomcat을 포함하고있습니다. 이번 챕터에서는 Spring 웹 프로젝트를 구동시키는 Tomcat을 통해 WAS의 역할과 구조를 알아보도록 하겠습니다.

## Contents List
{: .no_toc .text-delta }

1. TOC
{:toc}

---

### Tomcat이란?

Tomcat은 Apache사에서 개발한 오픈소스 WAS 소프트웨어로써,  JSP와 자바 서블릿이 실행하기 위한 서블릿 컨테이너를 제공합니다.  서블릿 컨테이너란 서블릿의 life-cycle을 관리해주고 request와 response를 위한 통신을  제공 해주는 컨테이너 이며 Tomcat은 Spring에서 만든 웹 어플리케이션의 Dispatcher Servlet, 기타 Servlet 및 JSP를 관리하고 실행 시킬수 있는 환경을 제공해 주는 소프트웨어 라고 할 수 있습니다.  

---

### Servlet Container

위에서 나온 서블릿 컨테이너에대해 좀더 자세히 알아보도록 하겠습니다.  
서블릿은 서블릿 자체로써 실행 및 동작될수 없기때문에 서블릿을 관리해주는 서블릿 컨테이너가 필요합니다. 서블릿 컨테이너의 역할은 다음과 같습니다.  
  
1\. 서블릿 life-cycle 관리  
서블릿을 로드해 초기화하고, 클라이언트의 요청이 있을때 서블릿을 호출한뒤 종료시 소멸시킵니다. 이는 init() -> service() -> destroy() 메소드 호출들을 통해서 이루어집니다.  
  
2\. 통신 지원
일반적으로 네트워크 통신을 위해서는 웹소켓을 만들어 통신을 해야하지만, 서블릿 컨테이너에서는 기본적으로 제공하여  웹서버와 손쉽게 통신을 할 수 있도록 해줍니다.  
  
3\. 멀티쓰레딩 지원  
클라이언트의 요청에따라 서블릿을 실행해주는데, 이때 새로운 쓰레드를 생성하여 서블릿을 실행시켜주는 멀티쓰레드 방식으로 요청을 처리해줍니다.  
  
---
  
### Tomcat의 서비스 실행 process

서비스 컨테이너의 역할까지 알아보았는데요, 이제 탐캣이 어떻게 서비스를 실행 시키는지 알아보도록 하겠습니다.  

1\. 기본 Class 로드  
가장먼저, `Bootstrap`, `System`, `Common`, `WebappX` 클래스 로더들을 통해 기본 클래스들을 로드시킵니다. 여기에 포함되는 클래스들은 $JAVA_HOME, $CATALINA_HOME, $CATALINA_BASE, WEB-INF/lib의 클래스로써 탐캣의 JVM 구동을 위해 기본적으로 필요한 기본이되는 클래스들을 로드시킵니다.  
  
2\. Server 초기화  
server.xml파일을 파싱하여 설정해둔 내용 기반으로 서버를 초기화합니다. 해당 설정파일을 보면 Server -> Service -> Connector -> Engine -> Host -> Context 순으로 구조화 되어있음을 확인 할 수 있습니다. 이 과정을 통해 실제 Web Application Server가 활성화 됩니다.

```c
//server.xml

<?xml version='1.0' encoding='utf-8'?> 
<Server port="8005" shutdown="SHUTDOWN"> 
    <Listener className="org.apache.catalina.startup.VersionLoggerListener" /> 
    <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" /> 
    <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" /> 
    <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" /> 

    <GlobalNamingResources> 
        <Resource name="UserDatabase" auth="Container" 
        type="org.apache.catalina.UserDatabase" 
        description="User database that can be updated and saved" 
        factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
        pathname="conf/tomcat-users.xml" />
    </GlobalNamingResources>

    <Service name="Catalina">
        <Connector port="8080" protocol="HTTP/1.1"
        connectionTimeout="20000"
        URIEncoding="UTF-8"
        redirectPort="8443" />
        <Engine name="Catalina" defaultHost="localhost">
            <Realm className="org.apache.catalina.realm.LockOutRealm">
                <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
                resourceName="UserDatabase"/>
            </Realm>

            <Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">
                <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
                prefix="localhost_access_log" suffix=".txt"
                pattern="%h %l %u %t &quot;%r&quot; %s %b" />
                <Context path="" docBase="/EXAMPLE"/>
            </Host>
        </Engine>
    </Service>
</Server>
```  
  
3\. 초기 서블릿 로드  
web.xml에서 설정해둔 서블릿들을 로드하여 객체화 합니다. web.xml에는 DispatcherServlet에대한 정보가 있어 이 과정을 통해 Dispatcherservlet이 활성화 되게 되며 서버의 초기화는 완료됩니다.
  
4\. HTTP request  
이제 클라이언트로부터 request가 오면 서블릿 컨테이너는 요청 URL을 분석하여 DispatcherServlet > HandlerMapping 과정을 통해  어느 서블릿에대한 요청인지 찾고 해당 서블릿을 로드하여  초기화한뒤 쓰레드를 하나 생성해 서블릿 서비스를 호출하는 과정으로 reponse를 보내줍니다. 해당 과정에대해 이해를 돕기위한 모식도를 첨부합니다.  

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_tomcat/tomcat_process.png" style="height:400px;">
</div>  
  
---

### <마무리>
이전 챕터들에서 Spring 의 동작 프로세스를 알아보았었는데, 이때 DispatcherServlet 등의 서블릿들이 탐캣 WAS를 통해서 실제적으로 실행되고 관리가 되어짐을 알게 되었고 서블릿들의 life-cycle을 알게되었습니다. 이제는 Spring boot 내장탐캣을 많이 사용하면서 더더욱 편리하게 탐캣을 이용하고 있어 몰라도 잘 사용하실수 있겠지만 지만, 탐캣이 실제로 어떻게 서비스를 구동시키는지 알아두시면 분명히 큰도움이 될것입니다.  
  
---
