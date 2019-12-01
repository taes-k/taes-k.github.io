---
layout: post
comments: true
title: 객체지향 SOLID
tags: [java]
---

### 객체지향 설계 원칙 : SOLID
좋은 객체지향의 설계를 위해서는 다음 5가지의 원칙을 따르는것이 좋다. 이는 로버트 마틴이 정리한 원칙으로,  왜 이러한 원칙들이 세워졌는가에 대한 의미를 알아두도록 하자.  

**SRP(The Single Responsibility Priniciple)**
[단일 책임 원칙]  
- 모든클래스는 하나의 책임을 가진다. 
- 클래스의 목적은 하나여야한다.  


**OCP(The Open Closed Principle)**    
[개방 폐쇄 원칙]  
- 확장에는 개방, 수정에는 폐쇄 되어야 한다.
- 서비스의 확장 및 변경이 있는경우 있어서 부모클래스의 변경없이 인터페이스를 제공받아 수정 할 수 있도록 설계해야한다.  

**LSP(The Liskov Substitution Principle)**    
[리스코프 치환 원칙]  
- 자식클래스는 부모클래스를 교체할 수 있다.
```c
Parent a = new Parent();
Child b = (child) a;
``` 

**ISP(The Interface Segregation Principle)**   
[인터페이스 분리 원칙]  
- 클라이언트(다른 오브젝트를 사용하는 오브젝트)가 자신이 이용하지 않는 메서드에 의존하지 않아야한다.
- 사용하지않는 함수를 implement하지 않아야 한다.  

**DIP(The Dependency Inversion Principle)**  
[의존관계 역전 원칙]   
- 상위 클래스는 하위클래스에 의존되어서는 안된다.  



