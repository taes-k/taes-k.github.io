---
layout: post
comments: true
title: Spring JUnit 테스트
tags: [spring]
---

### 구동환경
- Spring boot 2.1.3

---

### JUnit
자바 단위테스트를 위한 프레임워크로써 따로 테스트 케이스를 작성하여 번거롭게 디버깅 하지 않아도 되며 실제 개발소스를 통해 테스트가 가능하다.  이번 포스팅에서는 Spring boot 환경에서 JUnit에 대하여 알아보도록 하겠다.   

Spring boot를 생성하게되면 기본적으로 `/src/test/java` 폴더와 함께 `SpringJunitTestApplicationTests.java`라는 파일이 함께 생성되어 있을것이다.  

![1]({{ site.images | relative_url }}/posts/2019-05-01-spring-junit-test/1.png)      

```c
// SpringJunitTestApplicationTests.java

package test.junit.spring;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringJunitTestApplicationTests {

@Test
public void contextLoads() {
}

}
```  

Springboot에는 기본적으로 JUnit test가 내장되어있음을 확인 할 수 있다.  

---

### JUnit 어노테이션과 단정문
**[ 어노테이션 ]**  

*@Test* 
```c
@Test
public void test() { 

}
```
테스트 대상 메소드임을 선언하는 어노테이션으로 테스트하고자 하는 모든 메소드들은 @Test 어노테이션을 붙여야한다.  @Test 에는 `@Test(timeout=1000)` 과 같이 timout 시간을 설정 할 수 있고, `@Test(expected=RuntimeException.class)` 예상결과를  런타임에러로 설정하여 테스트를 진행 할 수도 있다.  
  
*@BeforeClass, @AfterClass*
```c
@BeforeClass
public static void testBeforeClass() throws Exception {

}
@AfterClass
public static void testAeforeClass() throws Exception {

}
```
어노테이션 이름에서도 알수있다시피 테스트클래스가 실행하기보다 먼저 혹은 이후에 1회 실행되도록 선언해주는 어노테이션이다.  
  
*@Before, @After*
```c
@Before
public void testBefore() throws Exception {

}
@After
public void testAefore() throws Exception {

}
```
테스트 메소드들이 실행하기보다 먼저 혹은 이후에 실행되도록 선언해주는 어노테이션이다.


**[ 단정문 ]**  

*assertTrue(A)*
A가 True 인지 확인.

*assertNotNull(A)*
A가 NotNull인지 확인.

*assertSame(a,b)*
A,B이 같은 객체인지 확인.

*assertEquals(A,B)*
A,B값이 같은지 확인.

*assertArrayEquals(A,B)*  
배열 A,B가 값이 같은지 확인.    

---

### JUnit 테스트 예제
사칙연산 결과를 반환하는 프로그램 예제를 통해 JUnit 테스트 예제를 진행해보도록 한다.  
이 예제는 [예제 Github](https://github.com/taes-k/spring-example/tree/master/spring-junit-test)에서 확인 할 수 있다.   

먼저 실제 서비스에서 사용하는 Calc 클래스를 작성해 @Service로 등록시킨다.  
```c
// Calc.java

import org.springframework.stereotype.Service;

@Service
public class Calc {

    public int sum(int a, int b) {

        return a+b;
    }

    public int sub(int a, int b) {

        return a-b;
    }

    public int mul(int a, int b) {

        return a*b;
    }

    public double div(int a, int b) {

        return (double)a/b;
    }
}
```

테스트코드 작성  
```c
// SpringJunitTestApplicationTests.java

package test.junit.spring;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringJunitTestApplicationTests {

@Test
public void contextLoads() {
}

}
@Test
public void calcTest(){

    //Calc sum(a,b) Test 3+4 = 8
    int sum = calc.sum(3, 4);
    assertEquals(8,sum);

    //Calc div(a,b) Test 3-4 = -1
    int sub = calc.sub(3, 4);
    assertEquals(-1,sub);

    //Calc multiply(a,b) Test 3*4 = 12
    int mul = calc.mul(3, 4);
    assertEquals(12,mul);

    //Calc multiply(a,b) Test 3/4 = 12
    double div = calc.div(3, 4);
    assertEquals(0.75,div,0.01);

}
```  

`Run > JUnit test`  

![2]({{ site.images | relative_url }}/posts/2019-05-01-spring-junit-test/2.png)      
assertEquals를 통해 미리 설정해둔 테스트케이스의 값이 같지 않을때 예상값과 다르다는 Failure를 내게 된다.

![3]({{ site.images | relative_url }}/posts/2019-05-01-spring-junit-test/3.png)      
테스트값이 모두 성공일때는 다음과 같이 실패내역이 출력되지 않는다.    

```c
// SpringJunitTestApplicationTests.java

@Test
public void calcTest(){
    calc.div(3,0);
}
```
만약, 에러나 예외상황의 결과가 있을시 테스트결과에서 바로 확인이 가능하다.   
![4]({{ site.images | relative_url }}/posts/2019-05-01-spring-junit-test/4.png)      

테스트시에는 예상하는 예외상황의 결과도 테스트해봐야하는데  이를위해 `@Test(expected=RuntimeException.class)` 를 통해 예상 Exception 테스트케이스를 작성할 수 있다.

```c
// SpringJunitTestApplicationTests.java

@Test(expected=RuntimeException.class)
    public void exceptionTest() {
    //Calc expected divide by zero exception
    int div2 = calc.div(3, 0);
}
```
![5]({{ site.images | relative_url }}/posts/2019-05-01-spring-junit-test/5.png)     
예상했던대로 예외가 정상적으로 발생했기때문에 성공한것으로 확인되었다.  

---

### 마무리
JUnit을 통해 실제 서비스와 분리된 환경에서, 매우 간편하게 실제 서비스 코드를 가지고 테스트를 해볼수 있다.   
단순히 TDD를 해야지를 위해서 사용하기 보다는 테스트케이스를 정리해가며 개발을 함으로써 내가 생각했던것들과 고려하지 못한것들을 정리해가며 개발을 할 수 있기에 TDD의 본질을 생각하며 JUnit을 사용 할 수 있으면 좋겠다.
