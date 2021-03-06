---
layout: post
comments: true
title: 다시세우는 테스트코드 작성 전략
tags: [spring, junit]
---

### 개요  
사실 해당내용에 대해서는 정확한 답변을 드리기가 어렵습니다.  
사실 모든 개발내용에 대해 테스트를 진행 할 수 있다면 더할나위없이 좋지만, 그에대한 트레이드 오프는 분명하기때문에 실무에서는 모든 내용에대해 테스트를 진행한다는것은 쉬운일이 아닐것입니다.  
그럼에도 불구하고 개발을 함에 있어서 반드시 테스트를 적용해야하는데, 내가 어떤 목적으로, 어떤 내용들을, 어떤범위까지 테스트를 진행할지에 대해서 정해두지 않는다면 개발하는 내내 테스트에대한 두려움을 안게될 것 입니다.   
   
이번 포스팅에서는 '다시세우는 테스트코드 작성 전략'이라는 제목으로 테스트의 목적과 테스트 전략을 세워보도록 하겠습니다.  

### 테스트의 목적  
리팩토링 : 테스트를 잘 할 수 있는 코드를 작성하면 좋은 코드를 만들수 있습니다.  
품질보장 : 코드 추가, 변경에 대해 개발자의 품질에대한 불확실성을 줄여줄수 있습니다.    
  
### 테스트 주의사항   
테스트 코드작성을 목적으로 테스트 하지 않는다.  
코드 커버리지를 높이기위한 테스트를 하지 않는다.  
테스트 목적을 단순하게 잡습니다.  

> 1\. 내가 생각한 결과값이 나온다.  
> 2\. 내가 생각한 잘못된 값이 나온다.  
> 3\. 내가 생각한 에러가 발생한다.  

위의 세가지를 목적으로 테스트를 진행한다.  

### 다시세우는 테스트 전략  

**유닛테스트 작성**
- 서비스 로직 (서비스 메서드)를 기준으로 유닛테스트를 진행한다.  
- 유닛테스트 과정중에서는 실제데이터에 대한 검증을 진행하지 않는다.  
  - (DB 연결이 필요한 메서드는 Mockito를 활용하여 가상데이터로 처리한다.)  
- 하나의 테스트케이스에는 하나의 로직을 검증한다. 
  - (실제 객체도 최대한 SRP에 따라 나누어져야 한다.)  
  - 해당 로직은 테스트를 해야하는데, 다른 로직 매서드 내에 포함되어있다? > 리팩토링 대상. 메서드 분리  
- 테스트를 위한 코드를 만들지 않는다. 
  - 만약 테스트를 진행하는데에 문제가 있다? > 서비스 코드에 문제가 있을 가능성이 높다. 
  - -> 리팩토링 타이밍!  
- 트랜잭션이 있는경우, 트랜잭션 테스트는 별도로 진행한다.  
  -(mock을 활용하여 에러 사항에대한 테스트 진행)  
- 불필요한 검증 구문을 작성하지 않는다.  
  * 단순 CRUD 목적의 메서드의 경우, 유닛테스트를 따로 작성하지 않고 통합테스트 단계에서 검증한다.  

**통합테스트 작성**
- 통합테스트에 너무 많은 시간을 할애하지 않는다.  
  - 통합테스트를 잘 작성할수록 고려못한 많은 케이스들에대한 검증을 할수있다. 
  - 하지만 그만큼 테스트를 작성하는 시간과 실행하는데 오래걸리고 많은 리소스가 소요된다.  
  - 민감 사항에대한 검증은 최대한 단위테스트에서 검증하고 통합테스트환경에서는 전체적인 성공적인 동작에대한 호출로써 통합환경을 검증한다.  
- 가능하다면 mock 사용 없이 데이터단 까지 검증을 진행한다.  
- 호출에 필요한 파라미터 > 결과값에 대한 정의 및 검증을 위한 테스트를 진행한다.  
- 실제 데이터를 기반으로 검증한다.  
- 불변데이터로 검증할수 있는 테스트 코드를 작성한다. (시간에 따라, DB 데이터에 따라 실패할수도 있는 파라미터 사용 지양)    

### 좋은 테스트의 조건 (FIRST)   
> **F**ast (빠르다)  
> **I**ndependent (독립적이다) : 다른테스트에 의존적이지 않는다.  
> **R**epeatable (반복가능하다) : 언제든지 원하는 결과를 얻을수 있어야 한다.  
> **S**elf Validating (자가검증할수 있도록) : 테스트결과를 개발자가아닌 시스템으로 검증해야한다.   
> **T**imely (적시에) : 코드를 새로 작성할때, 변경할때가 테스트하기 가장 좋은 시기이다. 옛날 코드를 테스트하는것은 시간 낭비가 될 수도 있다.  

### 무엇을 테스트 할것인가? (Right-BICEP)  
> **Right** (올바른 결과를 얻는지에 대한 테스트)  
> **B** (경계조건에 대한 테스트) : 특수문자, 잘못된 양식 데이터, 수치적 오버플로우를 일으키는 계산, null값, 예상 기대값을 훨씬 벗어나는 값, 중복값, 시간이 맞지않는 데이터  
> **I** (역관계를 이용 할 수 있는 테스트) : 곱 ↔ 나눗셈, 더하기 ↔ 빼기, 긍정 ↔ 부정 등  
> **C** (다른로직으로 검증 할 수 있는 테스트) : 곱셈 ← 덧셈, mod ← 나눗셈 등  
> **E** (에러에 대한 테스트) : 강제 오류를 발생시킬수 있는 테스트 / 메모리에러, 디스크 에러, time 매칭 에러, 네트워크에러 등  
> **P** (성능에 대한 테스트) : 프로파일링을 통한 기대 동작 속도 테스트  


### 결국은 리팩토링
테스트의 목적은 결국은 리팩토링입니다.  

------

### TDD 기본 프로세스
실패하는 테스트 작성 > 통과하는 개발 코드 작성 > 리팩토링 > 실패하는 테스트 작성 (반복)   

> Rule 1. 실패하는 테스트 코드를 작성 하기 전에는 개발 코드를 작성하지 않는다.  
> Rule 2. 실패하는 테스트 코드를 한번에 두개이상 작성하지 않는다.  
> Rule 3. '현재' 실패한 테스트에 대해서만 개발코드를 작성한다. (추가적인 사항에 대해서 개발 X)  

### TDD의 이점
- 객체지향적인 코드를 자연스럽게 만들어낼 수 있습니다. (테스트 유닛별 메서드 생성 함으로써)  
- 전체 비용 감소 (언제나 품질검사된 개발 사항이 적용)  
- 디버깅 용이  
- 유지보수 용이  


### 참고 URL & 문헌
- 자바와 Junit을 활용한 실용주의 단위테스트 - 길벗출판사  
- 테스트를 작성하자. 너무 많이는 말고. 통합 위주로(Write tests. Not too many. Mostly integration.)  
- Unit Testing Best Practices: JUnit Reference Guide  