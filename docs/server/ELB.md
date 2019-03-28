---
layout: default
comments: true
title: ELB(Elastic Load Balancing)
parent: server
date: 2019.03.26
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### ELB

Elastic Load Blancing , 이름에서 알수있듯이 로드밸런싱의 주된 기능은 부하 분산이다.   
트래픽을 설정알고리즘을 통해 여러대의 서버로 분산시켜 하나의 서버에 트래픽이 몰리는 일을 방지시켜준다. 


### ELB 알고리즘
- RR (Round Robin)
- Hash-ip 
- Sticky session
등

### ELB 구조 
![1]({{site.images}}/ELB/elb1.png)

### 특징
- ELB의 스케일은 알아서 관리되며 장애가 일어난 서버를 감지하여 다른서버로 트래픽을 분산시켜준다.
- 사용자의 세션을 특정 인스턴스에 고정시켜줄수 있다.(hash-ip , stick session 등)
- 사용자는 ELB 에 우선 접속되기에, SSL을 적용시킬경우 ELB에 적용시켜 주어야한다.
