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

### <3.2.1 Apache Tomcat> 
### Tomcat이란?

Tomcat은 Apache사에서 개발한 오픈소스 WAS 소프트웨어로써,  JSP와 자바 서블릿이 실행하기 위한 서블릿 컨테이너를 제공합니다.  서블릿 컨테이너란 서블릿의 life-cycle을 관리해주고 request와 response를 위한 통신을  제공 해주는 컨테이너 이며 Tomcat은 Spring에서 만든 웹 어플리케이션의 Dispatcher Servlet, 기타 Servlet 및 JSP를 관리하고 실행 시킬수 있는 환경을 제공해 주는 소프트웨어 라고 할 수 있습니다.  

---

### Servlet Container

위에서 나온 서블릿 컨테이너에대해 좀더 자세히 알아보도록 하겠습니다.  
서블릿은 서블릿 자체로써 실행 및 동작될수 없기때문에 서블릿을 관리해주는 서블릿 컨테이너가 필요합니다. 서블릿 컨테이너의 역할은 다음과 같습니다.  
  
1\. 네트워크 통신   
일반적으로 네트워크 통신을 위해서는 웹소켓을 만들어 통신을 해야하지만, 서블릿 컨테이너에서는 기본적으로 제공하여  웹서버와 손쉽게 통신을 할 수 있도록 해줍니다.

  
### Tomcat의 서비스 실행 구조

Tomcat이 어떻게 서비스를 실행 시키는지 알아보기위해 Tomcat을 초기화 할때 어떤 일들이 일어나는지 알아보도록 하겠습니다. 

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_tomcat/tomcat_init_process.png" style="height:300px;">
</div>

***1. Bootstrap***
클래스 로더 

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

---

---

### <마무리>
사실  WS와 WAS의 역할 및 구조에대해서 알지못해도 간단한 설정만으로 서버를 동작시키는데에는 전혀 무리가 없기에 어쩌면 그저 이론적인것들이 아닌가 생각하실수도 있으실것 같습니다. 하지만 이들은 여러분의 어플리케이션을 직접적으로 동작시키는 역할을 하고 있으므로 이를 잘 알지 못한다면 실제 서비스 환경에서 서버 에러 대응 및 튜닝 등 서버를 직접 만져야할때 어려움을 겪게 되실 수도 있으니 미리 잘 알아두셨으면 좋겠습니다.    
  
---
