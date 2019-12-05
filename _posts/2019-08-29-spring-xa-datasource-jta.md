---
layout: post
comments: true
title: XA datasource, JTA
tags: [spring]
---

### XA

2PC(2 phase commit) 분산 트랜잭션 처리를 위한 X-OPEN 표준이며 하나 이상의 데이터베이스간에 2PC 트랜잭션이 보자오디어야할때 XA Datasource가 사용됩니다.  
  
![1]({{ site.images | relative_url }}/posts/2019-08-29-spring-xa-datasource-jta/1.jpg)   
  
글로벌 트랜잭션(분산 트랜잭션)을 적용하기위해서는 반드시 2PC를 해야하고 JTA를 사용할때 XA가 사용됩니다.

### JTA  
JTA (Java Transaction API)는 XA 리소스 간의 분산트랜잭션을 처리하는 JAVA API입니다.