---
layout: post
comments: true
title: Java stream에서 객체상태 변경
tags: [java, stream]
---

### Stream 객체상태 변경하기 

이른바 모던자바라 불리는 `Java8`이상 쓰시는 분들이라면 잘 사용하고 계실텐데요,  `Stream`에서 요소를 다른 객체로 변환하기 위해서 `map` 중간연산자를 잘 사용하고 계실것이라 생각합니다.

```java
List<String> nameList = list.stream()
    .map(User::getName)
    .collect(Collectors.toList());
```

`map` 중간연산자를 통해 요소를 다른 요소로 변환 할 수도 있지만, 객체의 상태를 변경하여 받은 요소 그대로를 return 시킬 수도 있습니다.

아래 코드예제를 통해 좀더 자세히 알아보도록 하겠습니다.

```java
// User.java

@Data
public class User {
    private String name;
    private Integer age;
    private boolean isAdult;

    public User(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public void setAdultWithRule(Function<Integer, Boolean> rule) {
        this.isAdult = rule.apply(age);
    }
}
```

```java
// UserTest.java

public class UserTest {

    @Test
    void streamTest1() {
        List<User> list = Arrays.asList(
                new User("userA", 15)
                , new User("userB", 16)
                , new User("userC", 17)
                , new User("userD", 18)
                , new User("userE", 19)
                , new User("userF", 20)
                , new User("userG", 21)
                , new User("userH", 22)
                , new User("userI", 23)
        );

        Function<Integer, Boolean> koreanAdultRule = age -> (age > 19);
        Function<Integer, Boolean> usaAdultRule = age -> (age > 18);

        List<User> adultUsers = list.stream()
                .map(user -> {
                    user.setAdultWithRule(koreanAdultRule);
                    return user;
                })
                .filter(User::isAdult)
                .collect(Collectors.toList());

        adultUsers.forEach(System.out::println);
    }
}
```

```java
// result - streamTest1

User(name=userF, age=20, isAdult=true)
User(name=userG, age=21, isAdult=true)
User(name=userH, age=22, isAdult=true)
User(name=userI, age=23, isAdult=true)
```

위 테스트 코드 수행 결과는 원하던대로 잘 수행되지만 `IntelliJ`에서는 아래와 같은 `warning`메세지가 나타나게 됩니다.

![1]({{ site.images | relative_url }}/posts/2021-08-01-java-stream-peek/1.png)   

요소의 변경없이 그대로 return 되는 경우 `map` 대신에 `peek` 중간연산자를 사용하라는 메세지 인데, 가이드대로 변경하는 경우 다음과 같은 코드로 치환이 가능합니다.

```java
List<User> adultUsers = list.stream()
    .peek(user -> user.setAdultWithRule(koreanAdultRule))
    .filter(User::isAdult)
    .collect(Collectors.toList());
```

---

### Stream 에서 객체 변환시 주의점, 혹은 사용하면 안되는 이유

위 예제에서와 같이 `Stream`에서 원본 객체에 변화를 가하는 작업에 대해서는 논란이 많습니다.  

- https://stackoverflow.com/questions/33635717/in-java-streams-is-peek-really-only-for-debugging
- https://stackoverflow.com/questions/44370676/java-8-peek-vs-map
- https://stackoverflow.com/questions/51648176/map-vs-peek-intellij-suggestions

일반적으로 `Stream` 내부에서 객체 변경작업을 수행시, 작업자는 `Stream` 모든 요소에 대해 작업이 수행 될 것을 기대하지만 특정상황에서는 작업이 호출되지 않을 수 있습니다.

`Stream.java` `peek()` 메서드에는 다음과같은 주석내용이 첨부되어있습니다.

> In cases where the stream implementation is able to optimize away the production of some or all the elements (such as with short-circuiting operations like {@code findFirst}, or in the example described in {@link #count}), the action will not be invoked for those elements.


- findFirst(), findAny()와 같은 단락 작업
- count

위와같은 최종연산 스트림작업에서는 연상이 최적화되어 모든 요소들에 대해 작업이 호출되지 않을 수 있고, 더욱이 병렬환경에서는 수행되는 요소가 무작위가 될 수 있기때문에 의도치 않은 결과를 나타낼 가능성이 존재하기도 합니다.

가장 중요한 점은 `API`의 설계 의도에 맞게 사용을 해야한다는 것 인데, 그렇지 않으면 지금은 결과가 동일하게 나올수도 있지만 추후 업데이트에서 어떻게 변경될지 모른다는 것 입니다.  

실제로 `java8` -> `java9` 업데이트가 되면서 `count`연산에 대한 최적화가 변경되면서 `peek`, `map`을 수행하지않는 변경이 일어났습니다.

위와같은 이슈들 때문에 '`Stream` 내부에서 원본 객체를 변환하는 작업을 해서는 안된다' 라고 주장이 존재한다는것을 알고서 작업에 참고를 하시면 좋을것 같습니다.

```java
list.foreach(user -> user.setAdultWithRule(koreanAdultRule))
List<User> adultUsers = list.stream()
    .filter(User::isAdult)
    .collect(Collectors.toList());
```

다음은 `Sonar`에서 `Major CodeSmell`으로 안내하는 내용입니다.

```
"Stream.peek" should be used with caution

According to its JavaDocs, the intermediate Stream operation java.util.Stream.peek() “exists mainly to support debugging” purposes.
A key difference with other intermediate Stream operations is that the Stream implementation is free to skip calls to peek() for optimization purpose. This can lead to peek() being unexpectedly called only for some or none of the elements in the Stream.

As a consequence, relying on peek() without careful consideration can lead to error-prone code.
This rule raises an issue for each use of peek() to be sure that it is challenged and validated by the team to be meant for production debugging/logging purposes.
```