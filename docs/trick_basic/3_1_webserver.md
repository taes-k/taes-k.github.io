---
layout: default
comments: true
title: (요령과 기본) 3.1 Web Server와 Web Application Server
parent: 요령과 기본
date: 2019.05.24
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

## 3.1 Web Server와 Web Application Server
{: .no_toc }
자바기반으로 웹 서비스를 개발하셨다면 배포를 위해 서버를 구성해 보셨다면 대부분 Web server와 WAS를 사용해 배포를 하셨을겁니다. 이번 챕터에서는 Web server와 WAS에 대해서 알아보도록 하겠습니다.

## Contents List
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## WS (Web Server)

웹서버는 클라언트와의 네트워크 통신을 위한 소프트웨어 입니다. 클라이언트로 부터 들어오는 HTTP 요청을 처리하며 정적 서버 콘텐츠를 제공하는것을 담당합니다. 여기서 정적 콘텐츠란 HTML, CSS, JS , 이미지를 의미하며 HTTP 프로토콜을 통해 클라이언트의 웹 브라우저에 제공됩니다.  
  
WS가 제공하는 기능들은 다음과 같습니다.  
- HTTP Request 처리
- 정적 컨텐츠 관리
- 컨텐츠 압축
- HTTPS 지원
- 웹서버 메모리 캐시

***WS의 종류***
- Apache web server
- Nginx
- IIS (Microsoft)
- Google web server

---

## WAS (Web Application Server)

웹 애플리케이션 서버는 서버단에서 웹 애플리케이션을 동작 할 수 있도록 지원해주는 소프트웨어 미들웨어로써,  동적 서버 콘텐츠를 처리에 최적화 되어있어 정적 콘텐츠를 주로 제공하는 '웹서버'와는 구별되어집니다.  
  
WAS가 기본적으로 제공하는 기능들은 다음과 같습니다.  
- 어플리케이션 실행환경 제공  
- 데이터베이스 접속 기능 제공
- 여러 트랜잭션 관리
- 비즈니스 로직 수행

***WAS의 종류***  
- Apache Tomcat 
- WebLogic
- Jetty
- JBoss

---

## WS와 WAS

현재는 WS 컴파일 모듈을 지원하면서 몇몇의 언어로 작성된 웹 어플리케이션의 동적 처리가 가능하기도 하고 WAS에서도 자체적 WS의 기능을 내장이 되어 있습니다. 따라서 단순히 기능만으로 두 서버 시스템을 구분 해서 사용 하기보다는 본인이 사용하려는 서버 시스템에서 어떤 목적으로 WS와 WAS를 도입하려는지, 또 어떻게 동작하는지를 파악하고 목적에 맞게끔 사용하시면 좋을것 같습니다.  
  
사실 위의 내용만으로는 WS와 WAS의 구조 및 정확한 역할을 알기 어려웠을것 같은데요 다음챕터에서 실제 Spring에서 구현한 웹어플리케이션과 Tomcat, Nginx를 바탕으로 실제 웹서버에서 WS와 WAS의 역할과 구동 원리를 알아보도록 하겠습니다.

---

## 웹 서버 구조

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/webserver/web_server_diagram.png" style="height:300px;">
</div>

---

## <마무리>
사실  WS와 WAS의 역할 및 구조에대해서 알지못해도 간단한 설정만으로 서버를 동작시키는데에는 전혀 무리가 없기에 어쩌면 그저 이론적인것들이 아닌가 생각하실수도 있으실것 같습니다. 하지만 이들은 여러분의 어플리케이션을 직접적으로 동작시키는 역할을 하고 있으므로 이를 잘 알지 못한다면 실제 서비스 환경에서 서버 에러 대응 및 튜닝 등 서버를 직접 만져야할때 어려움을 겪게 되실 수도 있으니 미리 잘 알아두셨으면 좋겠습니다.    
  
---
