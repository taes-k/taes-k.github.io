---
layout: default
comments: true
title: (요령과 기본) 1.2 Spring은 어떻게 동작하는가? fff
parent: (요령과 기본) 1.2 Spring은 어떻게 동작하는가?
date: 2019.05.11
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### 1.2 Spring은 어떻게 동작하는가?
앞서서 스프링을 사용하는 이유에대해서 알아보았는데, 과연 스프링은 어떻게 동작하길래 그러한 특징들을 가지는것인지 알아보도록 하겠습니다.  

---

### 1.2.1 IoC Container
### IoC와 DI
스프링프레임워크를 말할때 IoC와 DI를 주요 핵심 개념으로 이야기합니다.  
먼저 IoC(Inversion of Control)는 서비스 제어의 흐름을 역전시켰다는 뜻으로, 개발자가 아닌 프레임워크가 흐름을 제어하는 주체가되어 필요할때 코드를 호출하며 사용하게됩니다.   

다음은 DI(Dependency Injection)를 알아보도록 합시다. 하나의 객체가 다른객체를 참조할때 객체간의 의존성을 갖는다고 말을합니다.  
gftrgdegerg
