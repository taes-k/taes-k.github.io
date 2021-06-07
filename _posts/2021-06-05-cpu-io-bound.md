---
layout: post
comments: true
title: CPU Bound vs I/O Bound
tags: [cpubouond, iobound, process]
---

### CPU bound, IO bound

개발을 함에 있어서 수행하는 로직이 `CPU bound 작업` 혹은 `I/O bound 작업` 이라는 말을 많이 들어보셨을 겁니다.  
일반적으로 연산이 많이 필요한 로직은 `CPU bound`, 로컬 파일 시스템 혹은 네트워크 통신이 많은 로직은 `I/O bound`라고 이야기를 하셨을텐데요 이번 포스팅에서 이 둘에 대해서 조금 더 자세하게 다루어 보도록 하겠습니다.  

우선, OS 관련 수업을 들어보신분이라면 `CPU burst`, `I/O burst`에 대해 들어보셨을텐데요 `Burst`라는 단어의 뜻은 다음과 같이 정의되고 있습니다.

> 버스트  
> 어떤 특정된 기준(criterion)에 따라 한 단위로서 취급되는 연속된 신호(signal) 또는 데이터의 모임. 어떤 현상이 짧은 시간에 집중적으로 일어나는 현상. 또는 주기억 장치의 내용을 캐시 기억 장치에 블록 단위로 한꺼번에 전송하는 것.  
> [네이버 지식백과](https://terms.naver.com/entry.naver?docId=1590886&cid=50373&categoryId=50373)


즉 `CPU burst`는 프로세스 내에서 `CPU 명령`작업이 연속된 작업을 의미하며 `IO burst`는 로컬 혹은 네트워크등의 `I/O wait` 작업이 연속되는것을 의미합니다.  

아래 예시 그림을 보면 조금 더 이해에 도움이 될 수 있습니다.

![1]({{ site.images | relative_url }}/posts/2021-06-05-cpu-io-bound/1.png)   

프로세스는 수행과정중에 `CPU burst`, `I/O burst` 가 계속 변경되면서 수행되며 프로세스 작업의 종류에 따라 `CPU burst`가 일어나는 시간이 달라집니다. 이때 일반적으로 `CPU burst`가 많이 일어나는 작업을 `CPU bound process` 라고 부르게 됩니다.

---
### CPU Bound process

그렇다면 어떤 작업들이 `CPU`를 많이 사용하게 될까요?  
당연하게도 `CPU`의 주 목적에 부합하는 `연산`처리가 많은 프로세스에서 `CPU burst`가 많이 일어나게 될 것 입니다. 예시로, 연산이 많은 작업은 아래와 같은 작업들이 존재합니다. ß

- 데이터 마이닝
- 이미지 프로세싱
- 암호화폐 마이닝
- ...

---

### 성능향상을 위한 방법

위에서 알아봤듯이 프로세스마다 성능에 영향을 미치는 원인이 다르기때문에, 성능향상 방법을 고민하기에 앞서 작업한 프로젝트가 `CPU bound process` 혹은 `IO bound process`인지 판별하는것이 중요합니다.  

- CPU bound process

먼저, `CPU bound proces` 는 말 그대로 CPU의 성능에 영향을 받는 작업이기 때문에 CPU의 성능을 높여(높은 클럭의 CPU 사용) 작업의 성능을 향상시킬 수 있습니다.  

또한, 단순히 클럭만 높이는 방법이 아닌 프로세서가 추가된 멀티코어 프로세서를 이용해 작업을 CPU 병렬적으로 처리해 성능을 더욱이 향상 시킬 수 있습니다.   

특히나 일반적으로 CPU연산이 많이 발생하는 머신러닝 작업에서 `Many-core processor`인 `GPU`를 사용하는 이유도 병렬연산를 통해 성능을 향상시키려는 같은 이유 입니다.  

- IO bound process

로컬 저장소에 대한 `I/O wait`가 많은 작업의경우 하드웨어 교체를 통해 `(HDD -> SSD)` 성능을 향상시켜 볼 수 있을 것 입니다. 그보다 좋은것은, 로컬 저장소보다 빠른 `memory`를 활용 하는 것 입니다. 

다만, `memory`용량 에는 한계가 있고 네트워크에 대한 `I/O wait` 성능개선에는 큰 도움이 안될 수 있다는점이 존재합니다.  

외부 `I/O`의 성능을 향상시는것은 어쩌면 처리할수 없는 부분이기 때문에, `Non-blocking I/O`를 통해 쓰로풋을 향상시키는 방법도 있을 것 입니다.









