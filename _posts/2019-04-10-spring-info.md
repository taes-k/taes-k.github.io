---
layout: post
comments: true
title: Spring 이란?
tags: [spring]
---

### Spring
Java 개발에 있어서, 개발자에게 어플리케이션 레벨에만 집중할 수 있도록 해주는 경량 어플리케이션 프레임워크이다. 이때 경량이라함에 있어서 의문을 가질수도 있겠으나, Java 환경에 있어서는 기존의 EJB등의 프레임워크에 비해 많이 단순화가 되었다.

POJO(Plain Old Java Object) 기반의 구성으로, 특정 규약에 종속되거나 환경에 종속없이 일반적인 자바코드를 이용하여 개발이 가능하다.

### Spring 3대 특징
- IOC (Inversion Of Controll) : 제어역행
컨트롤의 제어권이 개발자가 아니라 프레임워크에 있어, 프레임워크의 필요에 따라 개발자의 코드를 호출함.
  
- DI (Dependency Injection) : 의존성 주입
각 계층의 서비스들간에 의존성이 존재할 경우 프레임워크가 서로 연결시켜줌.  
개발자가 객체를 연결시켜주기위한 별도의 설정 없이도 스프링에서 연결을 시켜줄 수 있다.  

- AOP (Aspect Oriented Programming) : 관점지향 프로그래밍
여러 모듈에서 공통적으로 사용하는 기능의 경우 해당기능을 분리하여 관리할수있다. 주로 어노테이션을 통해 관리하며 트랜잭션이나 로깅등에서 사용됨.

### MVC
Model , View, Controller 세개의 영역으로 분리 개발해 결합도를 낮추고 분업 및 유지보수를 용이하게 할수있는 소프트웨어 디자인 패턴  
- Model : Data 처리
- View : User Interface  처리
- Controller : Business 로직 처리

