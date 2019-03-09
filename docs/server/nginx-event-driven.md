---
layout: default
comments: true
title: Nginx와 Apache의 구동방식 비교
parent: server
date: 2019.03.08
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### Nginx vs Apache

웹개발을하는데 있어서 반드시 필요한 웹서버.  
그중에서도 모두들 들어봤을 Nginx 와 Apache를 동작방식에 근거하여 비교해보려한다. 

### Apache

Apache 국내는 물론 전세계적으로 아직까지 부동의 1위를 지키고 있는 웹서버이다. 웹서버의 표준을 잡았다고 해도 과언이 아닐것이다. 웹개발의 공식이라고 불리우는 "APM"중 A가 Apache 이다.  

- Apache 병렬처리 동작방식
기본적으로 Apache는 Process 기반 방식으로 클라이언트 요청들을 처리한다.  
요청에따라 프로세스를 생성하는 `MPM(Multi Process Module)-prefork` 방식과 프로세스와 쓰레드를 병행해서 사용하는 `MPM-worker` 두가지 방법이 있다.

> ![nginx-event-driven]({{ site.images }}/nginx-event-driven/nginx-event-driven_1.png)    
  

위 MPM 방식에따른 병렬처리는 클라이언트의 요청이 늘어날수록 프로세스가 계속해서 늘어나는 문제점을 가지고 있어 대량의 메모리가 필요하며 Context switching 발생으로 인한 부하가 걸릴 수 있다는 단점이 있다.  
`MPM-worker`방식이 `MPM-prework`방식에 비해 고속처리 및 더 대량의 처리를 하는것은 아니지만, process 수가 적게사용되어 메모리를 조금은 덜 소비한다. 

### Nginx 

웹에 발전에따라 웹서버에 트래픽이 점점더 몰리게되고 Apache 병렬처리 방식인 MPM \의 단점으로인해 사람들은 대안책을 찾기시작했다. Nginx는 2004년 러시아의 개발자에 의해 탄생한 웹서버로, 가벼우면서도 강력한 프로그램을 모토로 apache사용률을 절반이상으로 따라왔다. 

- Nginx 병렬처리 동작방식
Nginx는 event-driven 즉, 이벤트 구동방식으로 클라이언트 요청 병렬처리를 진행한다. 이는 입출력 이벤트를 감지해 처리가 필요할때 시그널을 통해 처리한다.  
기본적으로 싱글 프로세스 기반으로 이벤트를 받는 reactor와 이벤트를 실제 처리를 위한 worker로 전달하기위한 handler등으로 구성되어 처리단위로 동작하게된다. 

> ![nginx-event-driven]({{ site.images }}/nginx-event-driven/nginx-event-driven_2.png)

위 방식은 싱글 프로세스/스레드로써 Context switching 에 의한 오버헤드가 발생하지않아 빠른 처리가 가능하다. 또한 비동기처리로 인해 적은 메모리로 운영이 가능하다.  
다만, 긴 많은 I/O처리가 필요한 작업의 경우 시스템 큐에 요청이 쌓이게되어 성능이 저하 될 수 있다. 

따라서, 복잡한 처리나 대용량 데이터처리가 필요한 서비스 등에서는 적합하지 않을수 있다. 


### 마무리

현재 Apache에서도 nginx에 대응하기위하여 MPM Event-driven 구조로 2.4버전 업데이트를 진행하기도 한다고 한다. 만들고자 하는 서비스에따라 유리한 방식이 따로 있으니 선택은 본인의 몫.


