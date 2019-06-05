---
layout: default
comments: true
title: (요령과 기본) 2.3 Spring 프로젝트 시작하기 (4) - Testing
parent: 요령과 기본
date: 2019.05.30
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### 2.4 Spring 프로젝트 시작하기 (4) - Testing
{: .no_toc }
사실 테스트라는 과정이 없어서는 안되는 과정이지만 필수적이지는 않기 때문에 '계륵' 이라는 단어가 잘 어울린다고 할 수 있을것 같습니다. 특히나 저 같은경우에는 매일같이 잦은 변동사항속에서 빠르게 출시가 가능한 제품을 만들어 내야했던 스타트업 초기에는 테스트코드를 만들고, 테스트를 진행하는 과정이 굉장히 사치스럽게 느껴지기도 했습니다. 하지만 사실 테스트툴을 사용하지 않더라도 웹개발을 할때는 웹페이지를 직접 켜서 테스트를 진행한다던가 API개발을 할때는 curl을 통해 직접 결과값을 받아보며 테스트를 하는 등 훨씬 더 비효율적인 방법으로 테스트를 하기도 했었는데요, 이번 챕터에서는 스프링 테스트를 이용하여 간편하고 빠르게 TDD를 하는 예제 프로젝트를 진행해 보도록 하겠습니다.
  

## Contents List
{: .no_toc .text-delta }

1. TOC
{:toc}

---

### Spring test

Spring에서는 개발자들의 TDD를 지향하며 많은 편리한 사항들을 지원해주고 있습니다. 이에대한 자세한 설명은 이전 챕터에서 다루었기에 링크를통해 대체하도록 하겠습니다. [1.7 Spring testing](https://taes-k.github.io/docs/trick_basic/1_7_spring_testing/)   
  
 ---

### 실전 Testing

테스트 프로젝트는 바로 이전 챕터에서 진행했던 [2.2 Spring 프로젝트 시작하기 (3) - JPA](https://taes-k.github.io/docs/trick_basic/2_2_spring_start_3_jpa/) 프로젝트로 진행을 하도록 하겠습니다. 이전 프로젝트에서는 JPA를 통해 데이터가 제대로 불러와졌는지 확인해보기 위해 MVC를 모두 구성하여 전달한 데이터가 웹뷰에 잘 나타나는지를 통해 정상작동을 확인했습니다. 아마 이와같은 테스트는 데이터 단 만이라도 MVC가 통합개발이 완료된 상태에서나 가능 할 것입니다.   
  
물론 Repository나 Service에서 로그를 찍어 확인하는 방법도 있겠지만 이러한 방법은 '로그 출력을 위한 코드를 작성 > 빌드 > 직접 결과 확인'이라는 단계를 거쳐 테스트를 진행을 할텐데 테스트 자동화 툴과 비교하면 오히려 개발자들의 생산성을 더 떨어뜨린다고 할수 있을것 같습니다.  
  
그렇다면 Spring-test를 이용해 test를 하는 예제를 진행해 보도록 하겠습니다.    
  
***1. Repository 유닛테스트***  
```java
// NewsDaoUnitTest.java

@RunWith(SpringRunner.class)
@DataJpaTest
@AutoConfigureTestDatabase(replace = Replace.NONE)
public class JpaUnitTest {

    @Autowired private NewsDao newsDao;

    @Test
    public void findAllTest() {
        List<News> newsList = newsDao.findAll();
        assertThat(newsList)
            .isNotEmpty()
            .hasSize(3);
    }
}
```
  
다음과 같은 테스트 코드만으로 이전 프로젝트에서 사용했던 `newsDao.findAll()`JPA 메서드를 통해 데이터를 잘 가지고 왔는지 자동테스트 툴을 통해 확인이 가능합니다. 

테스트 성공시 출력화면  
<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_start_4_testing/testing_success.png" style="height:250px; border:1px solid #d0d0d0;">
</div>   
  
테스트 실패시 출력화면  
<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_start_4_testing/testing_fail.png" style="height:250px; border:1px solid #d0d0d0;">
</div>   
  
테스팅은 설정해둔 테스트케이스와 비교하여 성공/실패를 결정해주기 때문에 개발자가 생각하지 않던 결과가 나오면 바로 확인이 가능합니다.   
  
***2. server-side 통합테스트***

통합테스트를 위해 News 데이터를 불러오는 컨트롤러를 하나더 만들고 테스트를 진행 해 보도록 하겠습니다.  

```java

@RestController
public class NewsController {

    @Autowired
    MainNewsServiceImpl mainNewsService;

    @RequestMapping("/news")
    public Map<String,Object> index(Model model) {
        Map<String,Object> result = new HashMap<>();

        result.put("code", 200);
        result.put("result", mainNewsService.getNews());

        return result;
    }
}
```  

```java
@RunWith(SpringRunner.class)
@WebMvcTest(NewsController.class)
public class NewsMvcTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void getNewsTest() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders.get("/news"))
                .andExpect(status().isOk());

    }

}
```
  
@WebMvcTest를 통해 MVC 통합테스트 코드를 작성했습니다. 하지만 위와같이 테스트를 진행하게 되면 아마 다음과같은 오류로 테스트가 실패할것입니다. 
  
> Field mainNewsService in start.mvc.spring.controller.NewsController required a bean of type 'start.mvc.spring.service.MainNewsServiceImpl' that could not be found.
  
@WebMvcTest 어노테이션는 @Controller, @RestController 등의 컴포넌트들만 Bean으로 등록시켜 테스트를 진행하기때문에 `MainNewsServiceImpl` service가 등록되지 않아 생기는 문제입니다. 따라서 @Service, @Repository 등을 연결하기위해서는 별도로 등록을 해주셔야 하는데 이때 @MockBean을 이용해 등록이 가능합니다.  
  
```java
@RunWith(SpringRunner.class)
@WebMvcTest(NewsController.class)
@Transactional
public class NewsMvcTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private MainNewsServiceImpl mainNewsServiceImpl;

    @Test
    public void getNewsTest() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders.get("/news"))
            .andExpect(status().isOk());

    }
}
```
  
다음과 같은 테스트코드로 테스트가 가능합니다.  

***3. client-side 통합테스트***

위에서는 서버측에서 mock object를 통해 Controller를 실행시켰다면, 클라이언트측에서 직접적인 RestCall을 하는것처럼 테스트도 가능합니다.  
  
```java
// NewsIntegrationTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)
public class NewsIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    public void getNewsTest() throws Exception {
        ResponseEntity<News> response = restTemplate.getForEntity("/news", News.class);
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody()).isNotNull();
    }
}
```  
  
TestRestTemplate을 통해 컨트롤러를 클라이언트에서 호출하는것처럼 테스트를 하여 전체적인 통합테스트를 진행할 수 있습니다.  다만, testresttemplate을 사용하기위해 다른 포트를 사용하게되어 데이터 조작시 transaction이 설정 되지 않기에 주의해야합니다.  

---

### TDD

이번챕터에서는 이전에 만들어둔 프로젝트를 테스트를 해보았으나 사실 TDD는 테스트 주도개발로써, 테스트케이스를 먼저 만들고 실제 개발을 진행하여 개발 과정에서 설정해둔 테스트 케이스에 맞는지 확인해가며 개발해나가는 개발 방법론입니다.   
따라서 이번 예제프로젝트와는 조금 맞지 않으나 테스트환경은 같기때문에 미리 소개를 드렸습니다. 다음 섹션부터는 TDD를 통해 예제 프로젝트를 생성해보도록 하겠습니다. 

---

### <마무리>

스프링에서 단위테스트와 통합테스트 예제를 진행해 보았습니다. 어떻게보면 간단히 처리할수있으나, 어찌보면 여간 귀찮은 일 일지도  모릅니다. 하지만 분명한건 장기적으로 보았을때는 테스팅이 오류를 줄여주고 생산성을 향상시킬수 있다는 것입니다.  
물론 모든 개발사항들에대해서 유닛테스트부터 통합테스트 과정을 거치기에는 현실적으로 어려움이 있습니다. 하지만 중요한 개발사항들에대해서만 유닛테스트를 거치고 일반적인 개발사항들에 대해서는 통합테스트만 실행시킨다는 등의 개인 혹은 회사만의 테스팅 원칙을 세워 진행하게되면 더욱이 효과적으로 테스팅을 사용 하실수 있게 될것입니다.  
  
---
### 참고자료 

[Spring 5.1.6 release docs](
https://docs.spring.io/spring/docs/current/spring-framework-reference/testing.html#testing)  
[Spring Junit test](https://taes-k.github.io/docs/spring/spring-junit-test/)  
[1.7 Spring testing](https://taes-k.github.io/docs/trick_basic/1_7_spring_testing/)  
  
--- 

### 샘플 프로젝트 
{: .no_toc }

위 프로젝트는 다음 링크에서 확인하실수 있습니다.  
<https://github.com/taes-k/spring-example/tree/master/spring-mvc-start-testing>

---
