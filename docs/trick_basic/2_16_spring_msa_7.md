---
layout: default
comments: true
title: 2.16 Spring MSA (7) -  트랜잭션
parent: 요령과 기본(Spring)
date: 2019.06.20
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

## 2.16 Spring MSA (7) -  트랜잭션
{: .no_toc }
일체형 아키텍쳐 에서는 왠만한 모든 처리들을 데이터베이스 트랜잭션에 의존하여 롤백처리를 지원해줄수 있습니다. 하지만 MSA와같은 분산환경에서는 서비스마다 다른 데이터베이스를 쓰는것이 일반적이라 일괄된 롤백처리가 어렵습니다. 이번챕터에서는 MSA 환경에서 트랜잭션처리를 하는 방법에대해서 알아보도록 하겠습니다.  

## Contents List
{: .no_toc .text-delta }

1. TOC
{:toc}

---
## 스프링 트랜잭션 

작성중 입니다...

## 보상 트랜잭션




---

## <마무리>



---

## 샘플 프로젝트 
{: .no_toc }

위 프로젝트는 다음 링크에서 확인하실수 있습니다.  
<https://github.com/taes-k/spring-example/tree/master/spring-msa>


---
