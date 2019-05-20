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

### Distpatcher 서블릿




--- 

### 마무리


--- 

### 참조 문서
{: .no_toc }
Spring 5.1.6 release docs  
<https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-introduction>


---
