---
layout: default
comments: true
title: 2.11 Spring MSA (2) - 스프링 클라우드
parent: 요령과 기본(Spring)
date: 2019.06.12
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

## 2.11 Spring MSA (2) - 스프링 클라우드
{: .no_toc }
MSA의 확장에 맞추어 분산 시스템 개발은 필수가 되었습니다. 스프링은 이러한 발전에 맞추어 마이크로서비스 구축에 필요한 라이브러리들을 모아 '스프링 클라우드 프로젝트'를 제공하고 있습니다. 이번 챕터에서는 MSA 예제 프로젝트에서 사용하게될 스프링 클라우드 서비스에대해 먼저 알아보도록 하겠습니다.  

## Contents List
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Spring cloud config

마이크로 서비스의 확장으로 인해 관리해야 할 서버의 수는 수십, 수백개가 되었습니다. 갯수의 확장에 따라서 이를 관리해야하는 공통된 환경설정의 경우도 하나가 바뀌면 수십, 수백개가 함께 변화가 되어야 하는데 스프링 클라우드 컨피그는 클라우드상에 정의해둔 환경설정을 가져와 서비스에 적용시킬수 있도록 지원해주고 있습니다.   

<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_2/spring_cloud_config.png" style="height:300px; ">
</div>   
  
  

## Spring Eureka

서비스가 많이 늘어나게되면서 서비스들의 url 을 개발자가 직접 정적으로 지정하는것에 부담을 느끼고 잘 맞지 않다는것을 깨닫게 되었습니다. 이에따라 넷플릭스 OSS는 '동적등록'과 '동적탐색'을 위한 Eureka가 나오게되었고 스프링 클라우드에서도 이를 통합하여 지원 해주고 있습니다.  

<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_2/spring_cloud_eureka.png" style="height:300px; ">
</div>   

그렇다면 Eureka가 지원하는 '동적등록'과 '동적탐색'에 대해서 알아보도록 하겠습니다.   
먼저, '동적등록'이란 새로운 서비스가 시작될때 중앙 서비스 레지스트리에 등록하고 서비스가 제공되지 못하는 상태가 되었을때에는 제외시켜 가용한 서비스만을 등록시켜두는 것을 말합니다.     
'동적탐색'이란 사용자가 서비스 레지스트리에서 필요한 서비스를 찾아 호출할수 있게 하는것입니다. 이를통해 서비스들의 URL을 정적으로 지정해 주는 대신에 서비스레지스트리의 등록내용에 따라서 가용한 URL을 탐색할수 있게 됩니다.  


## Spring Zuul

서비스가 나누어지면서 다양한 api를 접근할때마다 다른 서버를 통해 접근해야하는데 그에따른 다양한 불편한 점들이 발생했습니다. 이를 해결하기 위해  Eureka와 마찬가지로 넷플릭스 OSS서 제공한 Zuul(주울)이 게이트웨이의 역할을 제공해 주고있습니다.  
  
그렇다면 Zuul 은 어떻게 게이트웨이의 역할을 하는지 알아보도록 하겠습니다. 

<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_2/spring_cloud_zuul.png" style="height:300px; ">
</div>   

위의 다이어그램을 보시면 이해하시겠지만 Zuul은 Eureka와 함께 사용되어, Eureka server에 등록되어진  서비스들을 탐색하여 마이크로서비스로 proxy를 통해 매칭시켜주게 됩니다. 특히 Zuul은 사전 필터, 라우팅 필터, 사후 필터, 에러 필터등 여러가지 필터를 제공하여 서비스들을 커스터마이징하여 적용 할 수 있도록 해줍니다. 
  
그렇다면 어떤경우에 Zuul을 사용하면 좋을까요?  

- 인증 및 보안처럼 마이크로서비스 전체에서 적용되어져야 하는것들을 게이트웨이에서 통합 진행할수 있습니다.  
- 실시간으로 마이크로 서비스들에 접근하는 데이터를 수집하고 모니터링이 가능합니다.  
- 서비스 요청자에따라서 서비스 인스턴스가 변화하는 동적 라우팅 설정이 가능합니다.  
- 부하 분산의 직접적 제어권을 가질수 있습니다.  

## Spring cloud stream

Spring은 MSA에서 더욱더 중요해진 Event-driven을 지원하는 Spring cloud stream을 제공하고있습니다. RabbitMQ, Kafka 드의 메시지 구현체들을 지원하며 꼭 하나의 시스템과 종속적이지 않게 햊줄수 있습니다. 또한 직관적인 어노테이션을 통해 간단하게 
구현을 할 수 있도록 해줍니다.  

<div style="text-align:center; margin:50px 0;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_msa_2/spring_cloud_stream.png" style="height:300px; ">
</div>   

## Spring cloud security

일체형 서비스에서의 인증은 세션데이터를 통해 진행됩니다. 하지만 마이크로 서비스에서는 각 서버마다의 세션이 유지되지 않기때문에 문제가 생기는데 이 문제는 위에서 알아본 게이트웨이를 통해 해결이 가능합니다. 게이트웨이 단일 진입점에서 인증을 먼저 확인후 인증이 완료되면 해당 마이크로서비스를 매칭 시켜주는 방식입니다. 하지만 이는 마이크로서비스 최종진입점 자체에서는 인증이 되지 않는다는 문제가 있습니다.  
  
이를 해결 하는 방법은 게이트웨이-마이크로서비스의 네트워크를 격리하여 게이트웨이에서만 진입하게하거나 릴레이 토큰방식으로, 게이트웨이에서 얻은 인증 토큰을 마이크로서비스까지 넘겨줘 토큰을 통해 마이크로서비스에서 재인증을 진행 할 수 있습니다. 스프링 클라우드 시큐리티는 이러한 릴레이토큰 방식으로 MSA에서의 보안을 지원하고 있습니다.  

---

## <마무리>

이번 챕터에서는 마이크로서비스를 지원하기 위한 프레임워크인 스프링 클라우드에 대해 알아보았습니다. 다음챕터에서는 스프링클라우드를 사용하여 MSA를 기반으로한 예제 프로젝트를 구축해보도록 하겠습니다.   

---
