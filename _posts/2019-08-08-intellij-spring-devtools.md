---
layout: post
comments: true
title: Intellij Spring devtools 적용 이슈
tags: [itellij, spring, devtools]
---

# Devtools 개요

기능 1. 템플릿(thymeleaf, freemaker ... )에 대한 cache property default들을 자동 비활성화 처리 

기능 2. Trigger 파일들에 대한 update 발생시 자동 시작

기능 3. 소스 변화시 application 이 브라우저 liveReload



# 발생 이슈

IntelliJ에서 devtools  적용 안되는 현상.



# Devtools 적용방법

- Dependency 추가
```
// build.gradle
compile 'org.springframework.boot:spring-boot-devtools'
```


- Run > EditConfiguration : Running Application Update Policies 필드값 변경
```
On 'Update' action = Update trigger file
On frame deactivation = Update resources
```

![1]({{ site.images | relative_url }}/posts/2019-08-08-intellij-spring-devtools/1.png)  



# 정리

IntelliJ Ultimate에서 제공해주는 Update Policies 정책으로 인해 업데이트가 바로 적용 안되었음.

확인 결과 Jetbrain에서 Spring devtools를 위해 만든 옵션으로, 다른 설정없이 해당 설정만을 통해 devtools기능을 모두 사용 가능하게 한것. (Jetbrain 문서참조)

application 옵션 설정을 통해 trigger 파일들을 따로 설정 가능함. (스프링 문서참조)

따로 liveReload 확장프로그램 설치 불필요함.



Jet brain 블로그 참조

https://blog.jetbrains.com/idea/2018/04/spring-and-spring-boot-in-intellij-idea-2018-1/

Spring 공식 문서 참조

https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html#using-boot-devtools-restart-triggerfile

IntelliJ에서 Spring Dev