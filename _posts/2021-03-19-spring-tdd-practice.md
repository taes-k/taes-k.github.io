---
layout: post
comments: true
title: TDD Spring 실무에서 적용하기
tags: [spring, tdd]
---

### TDD

`테스트 코드`, `TDD`를 적용하는것에 대해서는 이전에도 몇번 포스팅을 진행했습니다.  
(https://taes-k.github.io/2019/09/27/spring-junit-testing-strategy/)

사실 우리 모두 `테스트 코드`가 잘 짜여있으면 좋다는것을 알지만 실무에서는 적용하기 어렵다고들 이야기도 많이 하는데요, 이번 포스팅에서는 Spring 실무에서 `TDD`를 최대한 쉽고 빠르고 자연스럽게 활용 할 수 있는 방법을 예제를 통해 공유드리려고합니다.

--- 

### 테스트코드를 어떻게 짜야할까?

`TDD`는 실제 개발코드보다도 테스트 코드를 먼저 작성하게 되는데, `'코드가 먼저 작성된 상태에서 테스트코드를 작성하는것도 어려운데, 테스트코드를 먼저만드는건 훨씬 어렵다.'`라고 느끼시는 분들이 많이 계실겁니다.

하지만, 개발된 코드를 테스트하는것이 훨씬 어렵습니다.  
우리가 실무에서 개발된 코드에 대해 테스트 코드를 짜려면 `어떤코드들에 대해서 테스트코드를 짜야하지?`, `Spring Bean에 등록된 객체를 어떻게 테스트하지?`, `목객체를 생성하는 범위는 어떻게 정해야하지?`, `DB 연결은 어떻게하지?` ... 등 수많은 고려사항들을 만나고 이로인해 많은 고통을 받으셨을것이라 생각이 듭니다.

하지만 몇가지 원칙과 함께 테스트코드를 먼저 짜게된다면 단순하게 테스트코드를 만들고, 수정해나갈 수 있게됩니다.

- 유닛테스트는 DB 연결을 하지 않는다.
- 주요 로직에 대해서 테스트코드를 작성한다. (주로 public method가 될 것)

이를 통해 테스트코드를 작성하다보면 자연스럽게 `'테스트'`는 메서드를 테스트하는 것이 아니라 `'기능'`을 검증 하는것이 될 것입니다.


---

### Spring TDD 실무 예제

실무 예제를 위해 우리는 이번 작업이슈에서 `카탈로그 생성` 기능을 만든다고 가정하겠습니다.


**1\. 개발작업의 목적을 생각하고, 필요한 '기능'을 중심으로 Test 코드를 작성합니다.**

 '카탈로그 생성' 기능을 만든다고 가정했을때, 다음과같은 유닛테스트를 만들 수 있을것입니다.  

 ```java
@Test
void createCtlg()
{
    CtlgService ctlgService = new CtlgService();
    Ctlg ctlg = Ctlg.builder()
        .name("삼성 냉장고")
        .maker("삼성")
        .brand("비스포크");
    ctlgService.createCtlg(ctlg);
    Long createdCtlgId = ctlg.getId();
 
    Assertions.assertNotNull(createdCtlgId);
}
 ```
실제코드는 없기때문에 빨간줄이 잔뜩 그여 있는 테스트코드들이 완성 될 것입니다.

![1]({{ site.images | relative_url }}/posts/2021-03-19-spring-tdd-practice/1.png)

2\. 테스트코드의 에러를 없애기위해 실제 코드를 수정을 합니다.  

다음단계는 테스트코드에서 발생하는 에러들을 없애주는것 입니다.  
IDE 에서 표시해주고있는 에러들을 해결하기 위해, IDE의 기능들을 적극 활용합니다.  

*(클래스가 없어? → 클래스 만들어)*

![2]({{ site.images | relative_url }}/posts/2021-03-19-spring-tdd-practice/2.png)

![3]({{ site.images | relative_url }}/posts/2021-03-19-spring-tdd-practice/3.png)

*(메서드가 없어? → 메서드 만들어)*


![4]({{ site.images | relative_url }}/posts/2021-03-19-spring-tdd-practice/4.png)

![5]({{ site.images | relative_url }}/posts/2021-03-19-spring-tdd-practice/5.png)


(필드가 없어? → 필드 만들어)

![6]({{ site.images | relative_url }}/posts/2021-03-19-spring-tdd-practice/6.png)

자, 이렇게 IDE 의 도움으로 손쉽게 ***빨강***없는 테스트코드가 완성되었습니다. (짝짝짝)


![7]({{ site.images | relative_url }}/posts/2021-03-19-spring-tdd-practice/7.png)


3\. 테스트코드의 성공시키기 위해 실제 코드를 수정을 합니다.  

다음단계는 테스트코드를 성공시키는 것 입니다.
2단계에서 테스트코드의 컴파일에러는 없앴지만 위테스트를 실행시키면 기대했던 Assertion을 통과하지 못해 에러가 발생합니다.

![8]({{ site.images | relative_url }}/posts/2021-03-19-spring-tdd-practice/8.png)

현재 ctlgService.createCtlg() 메서드에는 로직이 존재하지 않기때문에 해당 테스트는 당연히 실패할수밖에 없습니다.
그렇다면 로직을 추가해보겠습니다.

우리가 기대하는 바는, 전달 받은 Ctlg 데이터를 DB에 등록했을때 자동으로 생성된 카탈로그의 ID를 얻어 올 수 있어야합니다.


![9]({{ site.images | relative_url }}/posts/2021-03-19-spring-tdd-practice/9.png)

테스트를 성공시키기 위해 위와같이 ctlg 를 받아서, id를 세팅해주는 로직을 추가했습니다.

![10]({{ site.images | relative_url }}/posts/2021-03-19-spring-tdd-practice/10.png)

간단하게 테스트를 통과 했습니다.

4\. 실제 서비스를 성공시키기 위해 필요한 로직을 구현합니다.  

> 사실 (1~3) 까지의 과정은 실제 구현을 하기 전, 테스트를 세팅하는 일 이었습니다.  
> TDD에 익숙하지 않을때 TDD를 한다고 할때 10중 9는 아래와 같은 반응일 것 입니다.
> 
> - 테스트코드를 언제 다 작성해?
> - 구현하기전에 테스트 코드를 작성하는게 말이 쉽지
> - 하지만 위 과정들을 통해 테스트 코드 세팅이 정말 간단하게 완성되었습니다.
> 
> 테스트 코드는 오히려 코드가 완성 된 후에 작성하려할때 더 복잡하고 어려워 질 수 밖에 없습니다.  
> 테스트 코드 작성이 실제 코드작성의 과정이 될 때 TDD의 힘이 크게 발휘 될 수 있습니다.

createCtlg 메서드에서 전달받은 Ctlg에 setId를 해 줌으로써 간단하게 해결을 했으나, 사실 실제로 서비스에 필요한 로직은 전혀 구현되지 않은상태입니다.

해당 서비스의 최종 목적은 전달받은 데이터를 통해 DB에 저장하는 것이기 때문에 CtlgService를 Bean으로 등록하고 Repository로 연결해 DB에 저장하는 로직을 구현합니다.

![11]({{ site.images | relative_url }}/posts/2021-03-19-spring-tdd-practice/11.png)

5\. 테스트 코드의 오류를 수정합니다.  

실제 로직을 넣어 Service 코드를 구현했으나, related problem이 발생하고 있습니다.  
오류를 따라가보면 위에서 작성해두었던 Test코드에서 오류가 발생하고 있음을 확인하실수 있습니다.

![12]({{ site.images | relative_url }}/posts/2021-03-19-spring-tdd-practice/12.png)


CtlgService 생성자에서 Repository 주입이 필요하게 되어 테스트에서 선언한 생성자코드에서 오류가 발생하고 있습니다.

여기서부터는 Test코드 작성에 어느정도 숙련도가 필요합니다.

해당 유닛테스트 코드에서는 DB 연결은 제외하는것을 규칙으로 잡았기때문에 CtlgRepository를 Mock 객체로써 사용해주도록 합니다.

*(실제 DB 기능 까지의 테스트는 통합테스트를 통해 진행합니다)*

![13]({{ site.images | relative_url }}/posts/2021-03-19-spring-tdd-practice/13.png)


CtlgRepository Mock객체를 생성해주고,   
CtlgService 객체를 생성할 때 Mock 객체를 주입 해 줌으로써 실제 DB 를 타지않더라도 우리가 원하는 동작을 할 수 있도록 지정해 줄 수 있게되었습니다.  

*(여기서 스프링 DI시 생성자 주입을 사용해야하는 이유가 보여집니다.)*

```java
...

@Mock
 CtlgRepository ctlgRepository = Mockito.mock(CtlgRepository.class);
 
 ...
 
 CtlgService ctlgService = new CtlgService(ctlgRepository);
 
 
 ...
 Mockito.doAnswer(i -> {
         Ctlg arg0 = i.getArgument(0);
         arg0.setId(1L);
         return null;
}).when(ctlgRepository).save(ctlg);
 
 
 ...
```


![14]({{ site.images | relative_url }}/posts/2021-03-19-spring-tdd-practice/14.png)



6\. 리팩터링  

위의 과정을통해 테스트코드와 실제 코드의 구현을 자연스럽게 통합하여 성공적으로 완성했습니다.  

이과정까지 끝났으면 이제 어느정도 신뢰성있는 테스트 코드가 구축되었다고 판단하고 4~5의 과정을 반복하면서 리팩터링을 진행하면됩니다.  

리팩터링을 하는 과정에서 당연하게도 private method들이 생기게 될 텐데  
해당 private method들을 별도의 테스트를 만들려고 하지 않아도 작성해둔 테스트에서 모두 함께 체크 될 것입니다.

다만, 코드내에 분기가 생기거나 경계영역 데이터에대한 검사가 필요한경우에는 테스트 케이스들을 추가해 테스트를 강화시켜주시면 되겠습니다.



