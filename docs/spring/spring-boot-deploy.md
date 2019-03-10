---
layout: default
comments: true
title: Spring boot 배포하기 
parent: spring
date: 2019.03.09
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### Spring boot deploy

지난번 글에서 Spring boot로 프로젝트 만드는법을 알아보았다. 이번 포스팅에서는 Spring boot 에서 오나성시킨 프로젝트를 서버에 배포시키는 과정에대해서 포스팅 하려 한다.

### War 파일 export하기

Spring boot project의 경우 그냥 export를 하게되면 빌드파일이 빠져서 war파일이 제대로 생성되지 않는다. 따라서 별도의과을 통해 war파일을 추출해야 함을 기억하자. 참고로 지난 포스팅에 기술하였지만, war파일로 추출해내려면  
 `pom.xml`
```c
<packaging>war</packaing>
```
으로 설정되어 있어야한다.

![1]({{ site.images }}/spring-boot-deploy/spring-boot-deploy.png)

이후 프로젝트 우클릭 > run as > maven build 에서  
`Goals` : package 입력후 Run 하게되면 Console 창에 실행되는것을 확인하고 프로젝트 폴더> target 폴더 내로 들어가보면 `war파일`이 생성되어 있을것이다.  

해당 war 파일을 ftp를 통해 서버로 옮긴후 따로 서버 Tomcat을 통해서 실행시키는 것이 아닌 다음 명령문을 통해 배포단계에서도 마찬가지로 자체 Tomcat을 사용하여 실행 시킬수 있다.  

`java -jar war파일 경로/파일.war &`



