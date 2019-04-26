---
layout: default
comments: true
title: Spring Singleton
parent: spring
date: 2019.04.26
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### 구동환경
- Spring boot 2.1.3

### 싱글톤
애플리케이션이 실행될때 최초에 1회만 메모리를 할당하여 사용하는 디자인 패턴이다. (단 하나의 객체만 만들어서 사용하는 객체)  

### 싱글톤을 사용하는이유?
- 메모리 낭비 방지
- 공통된 객체의 재생성 방지 (하나만 사용되어야 하는 객체)

### 싱글톤  예제

```c
public void main() {
    //직접 가져온 오브젝트
    TestService testService = new TestService();
    TestDao testDao1 = testService.getDao();
    TestDao testDao2 = testService.getDao();

    System.out.println("testDao1 : "+ testDao1);
    System.out.println("testDao2 : "+ testDao2);


    //스프링 컨텍스트로부터 가져온 오브젝트
    ApplicationContext context = new AnnotationConfigApplicationContext(TestConfigService.class);
    TestDao testConfigDao1 = context.getBean("getDao",TestDao.class);
    TestDao testConfigDao2 = context.getBean("getDao",TestDao.class);

    System.out.println("testConfigDao1 : "+ testConfigDao1);
    System.out.println("testConfigDao2 : "+ testConfigDao2);


    //직접만든 싱글톤 오브젝트
    TestService testService2 = new TestService();
    SingleTestDao singleDao1 = testService.getSingleDao();
    SingleTestDao singleDao2 = testService2.getSingleDao();
    
    System.out.println("singleDao1 : "+ singleDao1);
    System.out.println("singleDao2 : "+ singleDao2);
    System.out.println("---------------");
}
```

직접 등록하는 서비스  
```c
public class TestService {
    public TestDao getDao() {
        return new TestDao();
    }

    public SingleTestDao getSingleDao() {
        SingleTestDao singleDao = SingleTestDao.getInstance();
        return singleDao;
    }
}

```

Spring context 등록 서비스
```c
@Configuration
public class TestConfigService {

@Bean
public TestDao getDao() {
    return new TestDao();
    }
}
```

싱글톤 구현 Dao
```c
public class SingleTestDao {

    private static SingleTestDao instance;
    private SingleTestDao() {}

    public static synchronized SingleTestDao getInstance(){
        if(instance==null){
            instance = new SingleTestDao();
        }
        return instance;
    }
}

```

결과  
![1]({{ site.images | relative_url }}/spring-singleton-obj/1.png)    
  
위 결과를 보게되면 
- 1. 직접 가져다쓴 객체는 당연히 호출때마다 새로운 객체를 만든다.
- 2. Spring Context로 등록하게되면 싱글톤 객체로 생성된다.
- 3. 싱글톤으로 생성된 객체는 다른 어떤 객체에서 호출해도 하나의 같은 객체를 반환해준다.  

### 싱글톤의 한계
- private 생성자를 갖음으로 상속할수 없다.
- 전역상태를 만들수 있어 너무 많이 공유시킬경우 인스턴스들간의 결함도가 높아진다. 
- 멀티쓰레드의 접근이 가능함으로 동시성의 문제가 생길 수 있음.

### 스프링의 싱글톤
스프링의 주 목적은 자바 서버를 위해 개발된 프레임워크이기때문에, 많은 트래픽의 요청을 처리하기에 유리하도록 설계되었다. 그렇기에, 매번 클라이언트에서 요청이들어올때마다 객체들을 만들어서 사용하게 된다면 엄청난 부하가 걸릴것이다. 따라서 서블릿 클래스당 하나의 오브젝트만 만들어두고 여러 스레드에서 하나의 오브젝트를 공유해 동시에 사용하게 된다. 이렇게 한개의 오브젝트만 만들어서 사용하는것이 싱글톤 패턴의 원리이다.   

자바만으로도 물론 싱글톤을 구현할 수 있지만 static 메소드와 private 생성자를 사용해야하는것과 다르게 스프링 싱글톤 레지스트리를 통해 일반 클래스를 싱글톤으로 활용 할 수 있다. 어노테이션 설정만으로 IoC방식의 컨테이너에 제어권을 넘겨줌으로써 손쉽게 사용할 수 있다.
