---
layout: post
comments: true
title: JAVA 예외처리
tags: [java]
---

### Exception
코드를 작성시 불가피하게 더이상 코드를 실행시킬수 없을때가 있다. 이때,  JVM에서는 문제가 있음을 알려야하는데 그 방법으로 Exception을 발생시킨다.   
자바에서는 2가지 종류의 Exception이 존재한다. `NullpointerException`이나, `IllegalArgumentException` 등 `RuntimeException`을 상속하는 `<Unchecked exception>`과  `RuntimeException`을 상속하지않는 나머지 `<Checked exception>`으로 구분된다.     
  
 `<Unchecked exception>`의 경우 주로 프로그램의 오류가 있을때 발생하도록 의도된 것으로 미리 코드에서 조건등을 통해 회피할수있는 오류들이라 catch나  throw를 사용하지 않아도 된다.  
 `<Checked exception>`의 경우 개발자가 예상하지 못한 오류들이 발생 할 수 있으므로 그에 대한 처리를 직접 하게끔 설계되어 catch나 throw 를 통해 반드시 예외처리를 해 주어야 한다.   

### Exception 처리
- 예외복구
예외 상황 발생시 문제를 해결해 정상상태로 돌려놓기. (재시도를 통한 방법 등)

- 예외회피  
예외의 throw를 통해 상위 메소드에서 예외처리를 할 수 있도록 한다.

- 예외전환
의미가 분명한 예외로써 재가공하여 throw를 해 상위 메소드에서 처리 할 수 있도록 하거나 런타임에러로 포장해 throw 하도록 한다.

### Exception 생성
로직에 있어서 예외를 생성해주어야할때, 비즈니스 로직에 맞추어 명시적인 예외명을 설정해주는것이 좋다.  
계정 관련 처리에를 예를들어보자면 Spring security 에서는 없는 ID일 경우에는 `UsernameNotFoundException`, 패스워드가 틀릴경우에는  `BadCredentialsException`의 Exception을 발생시켜주게된다.  
0, -1, -99 같은 상수값 코드를 통해 예외처리를 할경우 명확하게 관리를 하지 않을경우 혼란이 생길 수 있으니 에러코드 체계관리를 잘 하면서 사용해야 한다.  

### 마무리
- 예외를 잡는다면 반드시 처리를 해 주어야한다. 아무런 조치가 없거나 의미없는 throw 는 위험하다.
- 복구할수 없는 예외는 런타임예외로 전환시키는 것이 좋다.
- 로직을 담기위한 예외는 `checked exception`으로 만든다.
