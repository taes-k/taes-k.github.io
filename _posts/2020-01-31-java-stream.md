---
layout: post
comments: true
title: 자바 스트림
tags: [java]
---

(해당 포스팅은 자바 프로그래밍에는 익숙하지만, java8부터 지원하는 `스트림`에 대해서는 미숙한 독자분들을 대상으로 합니다. 만약 `람다식`에 대해서 미숙하신 상태라면 [자바 람다식](https://taes-k.github.io/2020/01/21/java-lambda-expression/) 포스팅을 선행하여 읽고 오시기를 추천드립니다. )

---

### Stream 개요

java8 에서 `람다식`과 함께 추가된 Stream은 '데이터의 흐름'을 다룰수 있는 기능으로, 기존의 배열/컬렉션을 다룰때 반복문을 이용해 각각의 요소들을 처리 하던것을 선언형 코드를 통해서 간결하고 직관적으로 처리 할 수 있습니다. 또한 별도의 멀티스레드 구현 없이도 간단하게 병렬처리를 통해 컬렉션을 다룰 수 있습니다. 

다음은 Stream을 알아보기 위한 예제 코드 입니다.  

```java
List<String> numbers = Arrays.asList("One", "two", "three", "FOur", "fiVe", "SIx", "sEveN");

for(String number: numbers){
    number = number.toUpperCase();
    if(number.contains("O")){
        System.out.println(number);
    }
}
```

위의 코드는 Stream을 사용하여 다음과 같이 구현 할 수 있습니다.

```java
List<String> numbers = Arrays.asList("One", "two", "three", "FOur", "fiVe", "SIx", "sEveN");

numbers.stream()
    .map(String::toUpperCase)
    .filter(s -> s.contains("O"))
    .forEach(System.out::println);
```

---

### Stream 사용하기

#### Stream 변환
가장 먼저, Stream을 생성하기 위해서는 Stream으로 사용할 data source가 있어야합니다. 일반적으로 배열, 컬렉션 등으로부터 Stream을 생성하게 됩니다.  

- 배열 -> Stream 
```java
String[] stringArray = {"one", "two", "three", "four"};

// Array -> stream 생성
Stream<String> stringStream = Arrays.stream(stringArray);
```

- 컬렉션 -> Stream
```java
List<String> stringList = Arrays.asList("one", "two", "three", "four");

// List -> stream 생성
Stream<String> stringStream = stringList.stream(); 
```


#### Stream 주요 연산

Stream의 연산은 두가지 종류가 있습니다. 결과로 다른 Stream을 반환하는 `중간연산`과 중간연산들을 합쳐서 최종적으로 한번에 처리해주는 `최종연산` 두가지 연산입니다.  
`중간연산`은 선언한다고 작업이 바로 수행되지 않으며 `최종연산`이 선언될때 여러개의 `중간연산`이 차례로 lazy하게 수행하게 됩니다.

*중간연산*

- Filter  
스트림 데이터에서 특정 연산에 불일치하는 데이터들을 필터링하여 제외 시킬수 있습니다.
```java
// "O" 문자가 포함된 데이터만 필터링
stringStream.filter(s -> s.contains("O"));
```

- Map  
스트림 데이터를 특정 연산을 통해 변환 할 수 있습니다.
```java
// 데이터를 대문자로 변경
stringStream.map(String::toUpperCase);
```

- Sorted  
스트림 데이터를 정렬 할 수 있습니다.
```java
// 데이터를 내림차순으로 정렬
stringStream.sorted((s1,s2)->(s2-s1)));
```

- Limit  
스트림 데이터 갯수를 제한 할 수 있습니다.
```java
// 데이터를 5개까지 제한 
stringStream.limit(5);
```

- Distinct  
스트림 데이터를 중복 제거 할 수 있습니다.
```java
// 데이터 중복제거
stringStream.distinct();
```

- mapToXX  
스트림 데이터 타입과 함께 데이터를 변경 할 수 있습니다.
```java
// String 데이터 -> length로 변환
stringStream.mapToInt(String::length);
```


*최종연산*

- 집계연산(count, min, max, sum, average)  
일반적입 집계연산 결과를 구할수 있습니다.   
```java
// 데이터 갯수
stringStream.count();
```
```java
// 데이터 최소값
stringStream.min();
```
```java
// 데이터 최대값
stringStream.max();
```
```java
// 데이터 합계
stringStream.sum();
```
```java
// 데이터 평균값
stringStream.average();
```

- reduce  
스트림 최종 데이터를 정의한 연산으로 누적 처리 할 수 있습니다.
```java
// 모든 String 데이터 concat
stringStream.reduce((String s1,String s2)->(s1+s2));
```

- forEach  
스트림 최종 데이터를 순환하여 처리 할 수 있습니다.
```java
// 모든 데이터 순환 출력
stringStream.forEach(System.out::println);
```

- collect  
스트림 최종 데이터를 모아서 객체로 만들수 있습니다.
```java
// 데이터 List 객체화
List<String> result = stringStream.collect(Collectors.toList());
```
```java
// 모든 String 데이터 concat
String result = stringStream.collect(Collectors.joining());
```

- XXMatch  
원하는 데이터가 Stream에 있는지 확인하고자 할 수 있습니다.  
`allMatch`, `anyMatch`, `noneMatch` 메서드를 통해 다양한 조건에서 데이터 존재여부를 확인할 수 있으며 결과는 boolean 으로 return 됩니다.  

```java
// 모든 데이터 만족시 true
Boolean result = stringStream.allMatch(s -> s.length>5)
```

```java
// 적어도 하나의 데이터 만족시 true
Boolean result = stringStream.anyMatch(s -> s.length>5)
```

```java
// 모든 데이터 불만족시 true
Boolean result = stringStream.noneMatch(s -> s.length>5)
```

- findXX  
스트림 최종 데이터 내부에서 특정 데이터를 반환 할 수 있습니다.

```java 
// 첫번째 데이터 반환
String result = stringStream.findFirst();
```
 
```java 
// 임의의 데이터 반환
String result = stringStream.findAny();
```
---

### Stream의 내부반복
Stream에서 위와같은 연산들을 제공 할 수 있는 가장 큰 이유는 사용자 단에서 반복문을 통해서 각 요소들을 직접 탐색하여 처리했던 `외부반복` 방식과는 다르게, 내부적으로 요소들을 반복 탐색하는 `내부반복`을 통해서 개발자의 별도 처리 없이 간단하고 직관적으로 표현이 가능합니다.  

`외부반복`을 사용할시 `병렬처리`를 쓰레드별 요소 탐색 범위를 직접 지정해주고 각 처리결과를 병합 하는 등의 개발자의 별도 처리가 필요한데 `내부반복`을 사용함으로서 이러한 번거로운 개발 과정 없이 내부적으로 모두 처리가 가능해집니다.   

---

### Stream 병렬처리

위에서 설명드렸다시피, Stream 사용시 매우 간단하게 병렬처리를 통해 작업효율을 높일 수 있습니다. 처음 보여드렸던 예제를 다시 사용하여 병렬처리 예제를 알아보도록 하겠습니다.   

코드)
```java
List<String> numbers = Arrays.asList("One", "two", "three", "FOur", "fiVe", "SIx", "sEveN");

numbers.stream()
    .map(String::toUpperCase)
    .filter(s -> s.contains("O"))
    .forEach(System.out::println);
```
결과)
> ONE  
> TWO  
> FOUR

위의 결과를 보시면 List에 담겨있던 순서대로 순차 출력이 되신것을 확인 하실수 있습니다.  
위의 코드를 병렬처리 하게 되면 내부 로직에 따라 컬렉션을 병렬처리하여 각 쓰레드별로 다른 범위의 데이터를 처리하게 됩니다.   

코드)
```java
List<String> numbers = Arrays.asList("One", "two", "three", "FOur", "fiVe", "SIx", "sEveN");

numbers.stream().parallel()
    .map(String::toUpperCase)
    .filter(s -> s.contains("O"))
    .forEach(System.out::println);
```
결과)
> FOUR  
> ONE  
> TWO  

정말 간단하게 `.parallel()`을 추가해 줌으로서 병렬처리 된것을 확인 하실수 있습니다. 이는 위에서도 언급했다시피 Stream은 `내부반복자`를 통해 개발자의 개입없이 내부적으로 반복처리를 제공하기때문에 개발자 입장에서는 간단하게 병렬처리가 가능 할 수 있습니다.  

---

### Stream 사용시 주의사항

위처럼 좋은 것들만 제공해주는 Stream 같지만, 주의해서 사용 하셔야 할것들이 있습니다. 

- 무한 Stream 생성

```java
Stream.iterate(0, i -> (i+1) % 9)
      .distinct()
      .limit(10)
      .forEach(System.out::println);
```
위코드는 얼핏보기에는 문제는 없어보이지만 iterate 과정중에 0~8의 결과를 만들어내도 distinct를 통해 반복값은 모두 제거하면서 9개의 데이터를 계속유지하는 반면 limit(10)의 조건에 의해 데이터가 10개가 생길때까지 반복을 하면서 무한 연산 작업을 진행하게 될겁니다.  

위와같이 기존의 일반적인 반복 코드보다 디버깅 하기에는 가독성이 좋지않아 무한 Stream 작업이 생성되지 않도록 주의가 필요 합니다.

- Stream의 재사용

Stream은 재사용 할 수 없습니다.  
```java
Stream stream = numbers.stream();

stream.map(String::toUpperCase)
    .filter(s -> s.contains("O"))
    .forEach(System.out::println);

stream.map(String::toUpperCase)
    .filter(s -> s.contains("I"))
    .forEach(System.out::println);
```

위와같이 동일한 Stream 데이터셋을 재사용하여 다른연산을 실행하고 싶을수도 있습니다. 하지만 Stream은 한번의 `최종연산`이 끝나게 되면 재사용이 불가능합니다. 위의 코드는 Stream을 재사용 하기 때문에 정상작동 하지 않습니다.

- Stream의 소비  

Stream은 `최종연산`이 일어날때 이전에 정의해 두었던 `중간연산`들이 한번에 적용되어 소비되어집니다. 만약 `최종연산`없이 `중간연산`만 정의해두었다면 해당 Stream은 아무 일도 일어나지 않게 됩니다.  

```java
numbers.stream().parallel()
    .map(String::toUpperCase)
    .filter(s -> s.contains("O"))
```

위의 코드는 무언가 연산이 일어날것 같지만, `최종연산`이 없기때문에 실제로는 아무일도 일어나지 않습니다.

- 자원 과소비 주의

위에서 알아본 Stream들에 의하면 좋은점만 있는것 같은데 모든 외부 반복이 일어나는 코드들 (for문, while문)을 Stream.forEach() 구문으로 바꾸면 좋을까요?  

실제 for-loop 구문을 Stream forEach구문으로 바꾸어 비교하는 성능 테스트를 진행해보면 연속적인 조회 상황에서는 Stream을 사용하면 오히려 더 좋지 않은 결과가 나오며 CPU 사용량 또한 2-30% 더 사용한다는 실험 결과가 있습니다. 무조건적으로 Stream 사용이 좋다기 보다는 적절한 상황에서의 사용이 중요합니다.  

관련된 내용은 아래 URL으로 참조 드립니다.   
https://jaxenter.com/java-performance-tutorial-how-fast-are-the-java-8-streams-118830.html

---

### 마무리

위에서 알아본 Stream의 특징에 대해서 정리해보면 다음과 같습니다.  

- Stream에서 연산된 결과는 실제 데이터 소스를 변경하지 않습니다.
- Stream을 한번 사용하게되면, 닫히게 되어 재사용이 불가능합니다. (일회용)
- `중간연산`의 조합을 통해 복잡한 연산 과정이 필요한 데이터도 편하게 얻어 낼 수 있습니다.
- '내부반복'을 통하여 간단하게 병렬연산 처리가 가능합니다.

---