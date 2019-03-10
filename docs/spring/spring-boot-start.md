---
layout: default
comments: true
title: Spring boot 시작하기
parent: spring
date: 2019.03.09
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### Spring boot?

JAVA기반 웹 프로그래밍을 할때 가장 유명한 프레임워크를 꼽자면 단연 Spring이라 할수 있을것이다. Spring boot 라 하면 생소한 사람들도 있을수 있으나 스프링에서는 2014년 꽤나 최신 버전으로 스프링 프레임워크 프로젝트를 간편하게 세팅할 수 있도록 도와주는 서브 프로젝트인 Spring boot를 출시하였다.   
Spring boot를 사용하게되면 자체 내장 웹 애플리케이션서버(tomcat)를 통해 다른 웹 서버 설정 없이 서비스를 구동 할 수 있으며 프로젝트 시작시 설정내용들을 표준화된 스타터 설정을 통해 개발과 운영을 이전보다 쉽게 할수 있도록 해준다.  

이번포스팅에서는 Spring boot를 시작하는 분들을 위해 프로젝트 생성에 관한 포스팅을 진행하도록 하겠다. 


### Spring boot 프로젝트 시작하기 

> ![1]({{ site.images }}/docs/spring-boot-start/spring-boot-start_1.png)  
> 
> ![2]({{ site.images }}/docs/spring-boot-start/spring-boot-start_2.png)

먼저 새프로젝트를 생성해 Spring Boot - Spring Starter Project 선택 해 준다.

> ![3]({{ site.images }}/docs/spring-boot-start/spring-boot-start_3.png)

이제 기본 프로젝트 설정차례인데  
- `Name` : 프로젝트이름설정
- `Type`:  Maven / Gradle type  설정
- `packaging` : 배포 파일 설정 (Jar / War), View 단을 포함한 웹 서비스를 운영하기위함이라면 War 형식 packaging을 하여야함에 유의한다.

> ![4]({{ site.images }}/docs/spring-boot-start/spring-boot-start_4.png)

스프링 부트를 쓰는 이유중 하나인 자동 의존설 설정 창이다. 왼쪽의 설정 가능한 탭에서 선택만해주면 프로젝트가 생성될때 자동으로 dependency 들이 들어가서 설정되게 된다. 선택이 완료됬으면 Finish.

> ![5]({{ site.images }}/docs/spring-boot-start/spring-boot-start_5.png)

위 간편한 설정들만으로 프로젝트생성이 완료되었다.


> ![6]({{ site.images }}/docs/spring-boot-start/spring-boot-start_6.png)
  
pom.xml 을 확인하면 의존성 설정창에서 선택한대로 dependency들이 자동으로 설정되어 있는것을 확인 할 수 있다.  


### Spring boot jsp 웹서비스 시작하기

프로젝트가 생성 완료되었으니 웹 서비스를 실행 시켜보자.
먼저 스프링부트 프로젝트의 경우 앞서 설명한 것과 같이 내장탐캣을 통해 로컬및 서버에서의 탐캣 설치 없이 바로 구동할수 있다.  

> ![7]({{ site.images }}/docs/spring-boot-start/spring-boot-start_7.png)

Boot Dashboard를 통해 서비스 run 이 가능하다.
하지만, jsp view 단을 함께 서비스하기위해 몇가지 설정이 더 필요하다. 

> ![8]({{ site.images }}/docs/spring-boot-start/spring-boot-start_8.png)

`src/main/resources/application.properties` 이 파일은 이 프로젝트 안에서 spring 설정들을 정의해 주는 파일이다. 우리는 jsp view를 사용하기위해 prefix, suffix를 설정해주고 웹 포트를 지정해준다.

```c
spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
server.port=8080
```
  
> ![9]({{ site.images }}/docs/spring-boot-start/spring-boot-start_9.png)

다음은 view 단의 파일들이 들어갈 폴더를 지정해주어야 하는데, `src/main/webapp` 내에 `WEB-INF`폴더를 만들어 `css`, `js`, `images`, `jsp` 등 파일들을 저장해준다.  
테스트로 index.jsp를 하나 생성해 주었다.

> ![10]({{ site.images }}/docs/spring-boot-start/spring-boot-start_10.png)

이제 jsp 파일까지 만들었으니 controller 를 만들어 index 파일과 연결시켜주면 설정 완료.  

이제 Boot Dashboard에서 project를 실행시켜보자.

> ![11]({{ site.images }}/docs/spring-boot-start/spring-boot-start_11.png)

콘솔창에 Spring 표시가 나타며 정상적으로 동작이 완료되었다. 웹상에서 localhost:8080에 접속하여 정상작동 확인이 가능하다.

> ![12]({{ site.images }}/docs/spring-boot-start/spring-boot-start_12.png)

완료!

