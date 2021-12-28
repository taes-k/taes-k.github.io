---
layout: post
comments: true
title: 코틀린이 CheckedException을 없앤 이유
tags: [kotlin, checkedexcpetion]
---

### 코틀린에서의 CheckedException

코틀린에서는 자바와 달리 `CheckedException` 처리를 강제화하지 않습니다. (https://kotlinlang.org/docs/exceptions.html#checked-exceptions)   
따라서 Exception 을 처리할때 자바와 다음과 같은 차이점이 존재합니다.

```java
// java case1

public void doSomething(){
  try{
    service.somethingA()
  } catch(AException e){
    // exception process
  }
}

// java case2

public void doSomething() throws AException
  service.somethingA()
}
```

```kotlin 
// kotlin case1

fun doSomething(){
  service.somethingA()
}

// kotlin case2
@Throws(AException::class)
fun doSomething(){
  service.somethingA()
}
```

코틀린 코드에서 `CheckedException`을 일으키는 API를 사용하더라도 별도의 try-catch 혹은 throws 없이 (마치 `UncheckedException`을 사용하는것 처럼) 구현을 할 수 있습니다.  물론, 자바 혹은 타 언어와의 호환을 위해  `@Throws()` 어노테이션을 제공해 CheckedException 을 전파하여 사용 할 수 있도록 하고 있습니다.

---

### 코틀린에서는 왜 CheckedException을 취급하지 않을까?

그렇다면 코틀린에서는 왜 `CheckedException`을 사용하지 않을까에 대해서 찾아보니, 코틀린 개발진에서는 다양한 이유로 `CheckedException`의 불필요성에 대해 소개하고 있습니다.

- 많은 `CheckedException`은 무시처리 되고 있습니다.
- 소규모 프로그램에서는 개발자의 생산성과 코드품질을 향상시킬수 있으나, 일반적인 대규모 소프트웨어에서는 전혀 그렇지 않습니다.
- 낮은수준에서 발생하는 특정유형의 `CheckedException` 의 경우 (File I/O, Network, Database ...) 일반적인 응용프로그램에서는 알 필요 없거나 알고싶지 않아합니다. 만약 Exception을 캐치한다고 하더라도 적절하게 대응하기 힘든경우가 대부분이라 `RuntimeException`으로 rethrow처리하는 경우가 많습니다.
- `CheckedException`사용시 코드의 확장성에 이슈가 생길수 있습니다. 단일 CheckedException 사용은 훌륭하게 동작하는것으로 보이나 4~5개의 서로다른 `CheckedExcpetion`을 사용하는 하위 API를 호출하는 경우 Exception 체인이 기하급수적으로 증가할 수 있습니다.
- ...

이하 코틀린 개발진에서 공유하고자 하는 내용 아티클 원문은 아래 링크를 통해 확인하실수 있습니다.
- [https://radio-weblogs.com/0122027/stories/2003/04/01/JavasCheckedExceptionsWereAMistake.html](https://radio-weblogs.com/0122027/stories/2003/04/01/JavasCheckedExceptionsWereAMistake.html)
- [https://www.artima.com/articles/the-trouble-with-checked-exceptions](https://www.artima.com/articles/the-trouble-with-checked-exceptions)


---

### 코틀린에서 CheckedException을 사용해도 될까?

위 아티클이 쓰여진 시기를 보시면 아시겠지만 CheckedException에 대한 논란은 코틀린이 만들어지면서 생긴것이아니라 기존 자바 혹은 그이전부터 이야기가 나오던 오래된 논쟁거리라고 생각이 됩니다. 그렇기때문에 CheckedException을 사용하는것에 대해서는 당연히 개발자분들 각각이 생각이 다르실수 있으실거라 생각합니다.

다만, 코틀린을 사용하고 있다면 개발진의 개발의도를 따라주는것이 이 언어를 효과적으로 사용할 수 있는 방법이라고 생각하기에 무의미한 CheckedException 을 사용하지 않는것이 개발진의 의도를 따르는 방법이라고 생각합니다.

마지막으로 정리해보자면, 코틀린에서는 특별한 사유로 CheckedException을 사용하는 케이스를 제외하고는 (가령, 트랜잭션에서 Rollback 하지 않는 Exception 을 사용하기 위해서) UnCheckedException을 사용하는것이 개발진의 의도를 따라 코틀린을 사용하는 방법이 아닐까 생각해봅니다.