---
layout: default
title: spring security 적용하기-1
parent: spring
date: 2017.02.06
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:10px; color:#ddd">{{ page.date }} </div>

session 으로만 관리하던 프로젝트를 좀더 스프링화 시키기위해 spring security 를 적용 하고자 하다가 공부하며 정리 해 본다.

본 프로젝트에 바로 적용하고자 하였으나 설정을 바꾸기가 좀 그래서 새로운프로젝트에 설정해보며 알아보는걸로 시작.

1) pom.xml

스프링 시큐리티를 사용하기위해 pom.xml 에 디펜던시 추가

~~~c
    <dependency> 
    	<groupId>org.springframework.security</groupId> 
    	<artifactId>spring-security-web</artifactId> 
    	<version>{org.springframework-version}</version> 
    </dependency> 
    <dependency> 
    	<groupId>org.springframework.security</groupId> 
    	<artifactId>spring-security-config</artifactId> 
    	<version>{org.springframework-version}</version> 
    </dependency>
    
    <!-- CGLib --> 
    <dependency> 
    	<groupId>cglib</groupId>
    	<artifactId>cglib</artifactId> 
    	<version>3.1</version> 
    	<type>jar</type> 
    	<scope>compile</scope> 
    </dependency>
~~~



		


2) web.xml

security-context  불러올 위치 설정 및 시큐리티 필터 설정을 걸어준다.
이때 필터네임 springSecurityFilterChain 은 스프링 default 값으로 정해져있다.

~~~c
    <context-param> 
    	<param-name>contextConfigLocation</param-name> 
    	<param-value> /WEB-INF/spring/security-context.xml </param-value> 
    </context-param>
    
    <!-- spring security --> 
    <filter> 
    	<filter-name>springSecurityFilterChain</filter-name> 
    	<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class> 
    </filter> 
    <filter-mapping> 
    	<filter-name>springSecurityFilterChain</filter-name> 	
            <url-pattern>/*</url-pattern> 
    </filter-mapping>
~~~ 







3) security-context.xml

실제 security 설정을 해주는 xml을 생성해 설정해준다.

`<http>` : 접근 권한 설정 부분
`<authentication-manager>`  :  접근 권한 부여 부분
~~~c
    <?xml version="1.0" encoding="UTF-8"?> 
    <beans:beans xmlns="http://www.springframework.org/schema/security"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:beans="http://www.springframework.org/schema/beans"
    xmlns:sec="http://www.springframework.org/schema/security"
    xsi:schemaLocation="
          http://www.springframework.org/schema/security
          http://www.springframework.org/schema/security/spring-security-3.1.xsd
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans-3.2.xsd">
    
    <http auto-config="true"> 
    	<intercept-url pattern="/test" access="ROLE_USER" /> 
    </http> 
    	
    <authentication-manager> 
    	<authentication-provider> 
    		<user-service> 
    			<user name="guest" password="guest" authorities="ROLE_USER"/> 
    		</user-service> 
    	</authentication-provider> 
    </authentication-manager> 
    
    /beans:beans
~~~



기본적 로그인 구조 완료.

