---
layout: default
comments: true
title: (요령과 기본) 1.7 Spring Testing
parent: 요령과 기본
date: 2019.06.04
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

## 1.7 Spring Testing
{: .no_toc }
'테스트'는 소프트웨어 개발의 필수 요소중 하나이며, 'TDD(테스트주도개발)'는 벌써 10년 전부터 소프트웨어 개발자들에게 트렌드 로 떠올랐던것이 이제는 필수가 되었습니다.    
이번챕터에서는 우리가 해왔던 테스트를 정말 개발의 일부분으로써 필요한 과정으로써의 테스트로 만들어보고 스프링에서 테스트를 위해 지원해주는 사항들을 알아보도록 하겠습니다.
  
## Contents List
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## <1.7.1 Unit Test>
## Unit Test

'유닛테스트'는 '단위테스트'라고도 불리며, 애플리케이션 내 일정 모듈 혹은 개별적인 코드가 예상대로 작동하는지 테스트하는 과정을 의미합니다. 유닛테스트는 일반적으로 설정을 위한 런타임 인프라가 없기때문에 매우 빠르게 실행됩니다. 스프링에서는 Mock Object를 통해 개발한 코드를 독립적으로 테스트 할 수 있습니다.

---

## Test Annotation

Spring boot를 사용하신다면, 프로젝트를 만드실때부터 test클래스가 생성되어있는것을 보셨을것입니다.  
  
```java
// SpringMvcStartApplicationTests.java

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringMvcStartApplicationTests {

    @Test
    public void contextLoads() {
    }

}
```
  
위 클래스의 어노테이션을 보시면 `@SpringBootTest`가 설정되어 있는것이 보이실텐데, 스프링에서는 어노테이션으로 테스트클래스를 선언 해 줄수 있습니다.  단, 어노테이션마다 테스트의 목적이 달라져 테스트 목적에 맞는 어노테이션을 사용해 주셔야 합니다. 다음은 Spring-boot-test 에서 사용하는 어노테이션들 입니다.  
  
|어노테이션|스캔범위|내용|
|:--:|:--:|:--:|
|@SpringBootTest|전체 Bean|통합테스트|
|@WebMVCTest|MVC Bean|MVC 단위테스트|
|@DataJpaTest|JPA Bean|JPA 단위테스트|
|@RestClientTest|관련 Bean|클라이언트 입장 RestAPI 단위테스트|
|@JsonTest|관련 Bean|Json serialize 단위테스트|
  
위 정리내용 외에도 몇몇 지원하는 어노테이션들이 있습니다. 위에서 확인 가능하다시피 설정하는 어노테이션에따라서 스캔하는 Bean들이 달라지기때문에 테스트환경이 달라지고, 수행시간또한 달라 질 수 있습니다. 이외에도 `@ContextConfiguration` 어노테이션을 통해 개발자가 직접적으로 테스트시 참조할 bean context를 지정 해 줄수도 있습니다.   
  
---

## DAO UnitTest

이제 실제 DAO 단위테스트를 예제소스를 보도록 하겠습니다..  

```java
@RunWith(SpringRunner.class)
// specifies the Spring configuration to load for this test fixture
@ContextConfiguration("repository-config.xml")
@Transactional
public class HibernateTitleRepositoryTests {

    // this instance will be dependency injected by type
    @Autowired
    private HibernateTitleRepository titleRepository;

    @Test
    public void findById() {
        Title title = titleRepository.findById(new Long(10));
        assertNotNull(title);
    }
}
```

위 테스트 내용은 JPA에서 ID로써 사용자를 찾는지 확인하는 예제입니다. 테스트 소스를 확인해보시면 `@ContextConfiguration("repository-config.xml")` repository-config.xml 의 컨텍스트만 참조하여 테스트를 진행하고있으며, repository 테스트를 진행하기때문에 테스트 결과값이 실제 DB에 반영되지 않도록 `@Transactional`을 사용하고 있는것을 확인 할 수 있습니다.  

---

## <1.7.2 Integration Test>
## Integration Test  

애플리케이션을 직접 배포하지 않고도 원하는 모듈들이 잘 연결되었는지, 혹은 전체 애플리케이션이 잘 수행되는지 테스트하는 과정이 필요합니다. 그때 수행하는 과정이 바로 '통합테스트'입니다. 이 통합테스트 과정에서 스프링 IoC 컨테이너 컨텍스트가 잘 연결되었는지, JDBC,  ORM 등을 통해 의 데이터 베이스와의 연결이 잘 되었는지 등 전체적인 연결에대한 테스트가 가능합니다.  특히 Spring 컨테이너와의 통합 테스트에 필요한 클래스들이 `org.springframework.test` dependency로 관리되어 서버 환경에 관계없이 테스트를 진행 할 수 있습니다.  

---

## Spring 지원사항

스프링에서 통합테스트를 할때, 지원해주는 사항들은 다음과 같습니다.

- To manage Spring IoC container caching between tests.   
테스트간에 Spring IoC container 컨텍스트의 캐싱을 제공합니다. 예시를 들어보자면, JPA 매핑 파일이 50-100개 정도 있는 애플리케이션을 테스트를 한다고 해보면 로드하는데에만 10-20초가 걸릴수 있게되는데 이러한 로드가 매 테스트마다 진행되게 되면 생산성이 저하될수 있어 캐싱을 지원합니다.   
    
- To provide Dependency Injection of test fixture instances.  
테스트 내에서 Application Context를 로드하여 DI를 통해 테스트 클래스의 인스턴스를 선택적으로 구성 할 수 있습니다.  
  
> To provide transaction management appropriate to integration testing.  
테스트 내에서의 데이터베이스 액세스 내용을 트랜잭션을 통해 롤백시킬수 있습니다. 테스트내에서 데이터 조작이 실제 데이터 조작으로 이어지게 되는 문제를 차단시킬수 있도록 지원해줍니다.  
  
> To supply Spring-specific base classes that assist developers in writing integration tests.  
통합테스트 작성의 편리성을 위해 Application Context와 JdbcTemplate 자체 추상 클래스들을 지원합니다.  

---

## Server-side 통합테스트

Spring MVC에서 통합테스트는 대부분 Controller단위로 이루어지게 됩니다. Controller - Service - Repository의 일련의 연결에 있어서 통합테스트를 하는 과정인데 이때 스프링 MVC 테스트에서는 Mock servlet API기반으로 테스트를 실행하게됩니다. 자세한 사항은 예제와 함께 알아보도록 하겠습니다.  

```java
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@SpringJUnitWebConfig(locations = "test-servlet-context.xml")
class ExampleTests {

    private MockMvc mockMvc;

    @BeforeEach
        void setup(WebApplicationContext wac) {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
    }

    @Test
    void getAccount() throws Exception {
        this.mockMvc.perform(get("/accounts/1")
        .accept(MediaType.parseMediaType("application/json;charset=UTF-8")))
        .andExpect(status().isOk())
        .andExpect(content().contentType("application/json"))
        .andExpect(jsonPath("$.name").value("Lee"));
    }
}
```

위 예제에서는 MockMvc 가상 객체를 생성해, 어플리케이션 컨텍스트를 주입해 Controller에 Request mapping 되어있는 "GET /accounts/1"을 요청하여 "status = 200", "name = Lee" 값을 확인하는 예제입니다. 스프링은 이와 같은 방법으로 서버 측면에서 Controller 통합 테스트를 할수있는 환경을 Mock 으로써 지원해줍니다.

---

## Client-side 통합테스트

위에서는 서버측면에서 Controller를 실행시켜 테스트를 진행했다면, 클라이언트 측면에서 통합테스트도 진행 할 수 있습니다. 서버 측면에서 테스트에서는 Mock 서블릿을 이용해 테스트를 진행하지만 사용자 측면에서의 테스트에서는 Servlet Container를 사용해 실제 서버에서 사용하는것 처럼 테스트를 진행해 볼 수 있습니다.

```java

@SpringBootTest()
class ExampleTests {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    void getAccount() throws Exception {
    
        ResponseEntity<Account> response = restTemplate.getForEntity("/accounts/1", Account.class);
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody()).isNotNull();
    }
}
```

---

## <마무리>
스프링에서는 개발자들의 TDD를 지향하며 그를 지원하기 위해 편리한 테스트 환경을 제공해주고 있습니다. 위 기능들을 잘 사용하여 개발을 하는데 있어서 테스트가 하나의 불편한 작업이 아닌 편리하고 도움이 되는 작업이 될수 있다면 좋겠습니다.  

--- 

## 참조 문서
{: .no_toc }
Spring 5.1.6 release docs  
<https://docs.spring.io/spring/docs/current/spring-framework-reference/testing.html#testcontext-fixture-di>


---
