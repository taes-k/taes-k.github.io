---
layout: default
comments: true
title: (요령과 기본) 2.5 Spring RestAPI 서버
parent: 요령과 기본
date: 2019.06.05
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### 2.5 Spring RestAPI 서버
{: .no_toc }
Spring 프레임워크는 Spring MVC 그대로 Restful을 위한 서비스를  제공하고 있습니다.  이번챕터에서는 TDD로, Restful한 api 서비스를 만들어보고 스프링에서 Restful한 서비스를 위해 지원해주는 사항들을 알아보도록 하겠습니다.
  

## Contents List
{: .no_toc .text-delta }

1. TOC
{:toc}

---

### REST

먼저 REST에대해서 알아보도록 하겠습니다. REST API 혹은 RESTful, 이제는 REST 그자체로 불리는 REST는 'Representational State Transfer'의 약자로써 상태를 표현하면서 전송하는 아키텍쳐를 의미합니다. REST는 자원(Resource), 메서드, 메시지 3가지의 요소로 구성되며 기존의 HTTP 프로토콜 그대로를 활용 할 수 있습니다.  
  
***REST 특징***
1. 무상태성 (Stateless) : 서버는 들어오는 request들을 각각 별개의 요청으로 처리합니다. 세션과 같은상태를 따로 서버에 저장 하지 않고 들어오는 요청들을 단일의 메세지로 처리합니다.  
2. 인터페이스 일관성 (Uniform Interface) :  HTTP 표준에 따른 어떤 플랫폼, 언어에 관계없이 사용가능합니다.  
3. 캐시처리가능 (Cacheable) : 기존 HTTP에서 사용하던 E-Tag 캐시를 그대로 사용 가능합니다.   
4. 자체표현구조 (Self-descriptiveness) : API Resource 즉 URI 만으로 API 기능을 이해 가능합니다.   
5. 서버-클라이언트구조 (Server-client) : 서버와 클라이언트가 완전히 분리된 구조로써 의존성이 줄어듭니다.  
6. 계층화 (Layerd system) : 클라이언트는 REST API Server만을 호출하고 서버는 다중계층으로 이루어질수 있습니다. 클라이언트는 서버측의 계층구조를 따로 확인할수 없으며 암호화,로드밸런싱,프록시등을 사용해 확장이 가능합니다.  
  
  
  ***REST 주요규칙***
  URI는 정보의 자원을 표현합니다 (ex : POST http://taes-k.com/users)  
  
  1\. Method로서 자원의 처리방법을 정의합니다.  
  |Method|내용|
  |:--:|:--:|
  |POST|Create|
  |GET|Select|
  |PUT|Update|
  |DELETE|Delete|
  
  2\. URI에 정보의 자원(Resource)들을 표현해야합니다.  
 users 명사 복수형으로 사용하며 세부 하위 내용들에 대해서는 /이후에 붙여나감으로써 정보에대한 내용을 확장시켜나갈수 있습니다.  
 
 - 예제 1 ) GET https://taes-k.com/users/1 : 1번 user 내용 조회
 - 예제 2 ) POST https://taes-k.com/users/2 : 2번 user 생성
  
  ***REST 목적***
  사실 REST를 사용함으로써 특별하게 성능적으로 좋아지는 효과가 있지는 않습니다. 다만 URI만으로도 이해하기 쉬운 API를 제공해 주고 일관적이고 호환성을 높이기 위함에 있습니다.   
  현재 개발되고 있는 API의 대부분은 RESTFul 하게 개발되어지고 있기때문에 REST를 사용하면 안되는 특별한 이유가 없는한 REST규칙을 따라 개발해 나가는것을 추천 드립니다.  
  
 ---

### Spring Restful API 서비스 만들기 

자 이제 직접 Spring에서 REST API 서비스를 만들어 보도록 하겠습니다.   
  
***테스트케이스_1 작성***
먼저 제가 구상하고 있는 서비스의 플로우는 다음과 같습니다.  
1\. 뉴스 입력(저장) -> 생성된 뉴스 데이터 불러오기(조회)  
2\. 뉴스 데이터 불러오기(조회) -> 뉴스 데이터 수정(수정) -> 변경된 뉴스 데이터 불러오기(조회)  
  
도메인으로 보자면 뉴스 데이터를 생성, 수정, 조회하는 API를 만들고자하는데 위와같은 테스트 케이스를 통해 TDD를 진행하려고 합니다. 이를 위해 먼저 테스트 케이스를 작성하겠습니다.  
  
```java
// NewsApiTest.java

@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
@Transactional
public class NewsApiTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void insertNewsTest() throws Exception {
        //뉴스 생성
        mockMvc.perform(MockMvcRequestBuilders.post("/news/{id}",4))
            .andExpect(status().isOk())
            .andExpect(content().string("새로운 4번 뉴스"))
            .andDo(print());

        //뉴스 불러오기
        mockMvc.perform(MockMvcRequestBuilders.get("/news/{id}",4))
            .andExpect(status().isOk())
            .andExpect(content().string("조회된 4번 뉴스"))
            .andDo(print());

    }

    @Test
    public void updateNewsTest() throws Exception {
        //뉴스 불러오기
        mockMvc.perform(MockMvcRequestBuilders.get("/news/{id}",1))
            .andExpect(status().isOk())
            .andExpect(content().string("조회된 1번 뉴스"))
            .andDo(print());
        //뉴스 수정
        mockMvc.perform(MockMvcRequestBuilders.put("/news/{id}",1))
            .andExpect(status().isOk())
            .andExpect(content().string("수정된 1번 뉴스"))
            .andDo(print());
        //뉴스 불러오기
        mockMvc.perform(MockMvcRequestBuilders.get("/news/{id}",1))
            .andExpect(status().isOk())
            .andExpect(content().string("조회된 1번 뉴스"))
            .andDo(print());

    }

}

```

우선은, 위와같은 뉴스생성, 뉴스 수정 테스트 케이스를 만들어 보았습니다. 이제 위 테스트케이스를 만족시키기위한 컨트롤러를 만들어 보겠습니다.  
  
  ***컨트롤러 작성***
```c
// NewsController.java

@RestController
public class NewsController {

    @RequestMapping(value="/news/{id}", method=RequestMethod.POST)
    public String insertNews(@PathVariable ("id") int newsId) {
        String result = "새로운 "+newsId+"번 뉴스";

        return result;
    }

    @RequestMapping(value="/news/{id}", method=RequestMethod.PUT)
    public String updateNews(@PathVariable ("id") int newsId) {
        String result = "수정된 "+newsId+"번 뉴스";

        return result;
    }

    @RequestMapping(value="/news/{id}", method=RequestMethod.GET)
    public String getNews(@PathVariable ("id") int newsId) {
        String result = "조회된 "+newsId+"번 뉴스";

        return result;
    }
}
```

테스트 성공 출력화면   
  
<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_rest_api/test1_success.png" style="height:250px; border:1px solid #d0d0d0;">
</div>   
  
위의 테스트케이스를 통해 `/news`의 GET,PUT,POST 모든 Request mapping이 잘 이루어지도록 Controller가 잘 작성되었음을 확인 할 수 있습니다.   
  
사실 위와같이 Controller를 작성한것만으로 벌써 RestApi 서비스를 완성한것이나 다름없습니다. 지금 현재상태에서 실제 서버에 배포를 한다면 원하는 데이터를 잘 응답 해 줄것입니다.  
  
  
***테스트케이스_2 작성***
자 이제 테스트케이스_1이 잘 작동했으니 다음 테스트케이스로 수정해보겠습니다. 이번 케이스에서는 String이 아닌 NEWS Entity를 직접 받아올수 있게끔 해보겠습니다.  `News` 엔티티는 이전 프로젝트를 기반으로 하겠습니다.

```java
// NewsApiTest.java

@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
@Transactional
public class NewsApiTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper mapper;

    @Test
    public void insertNewsTest() throws Exception {
        //뉴스 생성
        News insertNews = new News();
        insertNews.setId(4);
        insertNews.setTitle("새로운 4번 뉴스");
        insertNews.setContents("새로운 뉴스 컨텐츠");
        insertNews.setDate(new Date());
        insertNews.setAuthor("Taes-k");
        insertNews.setType("sports");

        mockMvc.perform(MockMvcRequestBuilders.post("/news/{id}",4)
        .contentType(MediaType.APPLICATION_JSON)
        .content(mapper.writeValueAsString(insertNews)))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.id").value(4))
        .andDo(print());

        //뉴스 불러오기
        mockMvc.perform(MockMvcRequestBuilders.get("/news/{id}",4))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.title").value("조회된 4번 뉴스"))
        .andDo(print());

    }

    @Test
    public void updateNewsTest() throws Exception {
        //뉴스 불러오기
        mockMvc.perform(MockMvcRequestBuilders.get("/news/{id}",1))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.title").value("조회된 1번 뉴스"))
        .andDo(print());

        //뉴스 수정
        News updateNews = new News();
        updateNews.setId(1);
        updateNews.setTitle("수정된 1번 뉴스");
        updateNews.setContents("수정된 뉴스 컨텐츠");
        updateNews.setDate(new Date());
        updateNews.setAuthor("Taes-k");
        updateNews.setType("sports");

        mockMvc.perform(MockMvcRequestBuilders.put("/news/{id}",1)
        .contentType(MediaType.APPLICATION_JSON)
        .content(mapper.writeValueAsString(updateNews)))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.id").value(1))
        .andDo(print());

        //뉴스 불러오기
        mockMvc.perform(MockMvcRequestBuilders.get("/news/{id}",1))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.title").value("조회된 1번 뉴스"))
        .andDo(print());

    }
}
```

***컨트롤러 작성***
```java
// NewsController.java

@RestController
public class NewsController {

    @RequestMapping(value="/news/{id}", method=RequestMethod.POST)
    public News insertNews( @PathVariable ("id") int newsId,
                            @RequestBody News insertNews
    ) {

        return insertNews;
    }

    @RequestMapping(value="/news/{id}", method=RequestMethod.PUT)
    public News updateNews( @PathVariable ("id") int newsId,
                            @RequestBody News updateNews) {

        return updateNews;
    }

    @RequestMapping(value="/news/{id}", method=RequestMethod.GET)
    public News getNews( @PathVariable ("id") int newsId) {
        News result = new News();
        result.setTitle("조회된 "+newsId+"번 뉴스");

        return result;
    }
}
```

테스트 성공 출력화면   

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_rest_api/test2_success.png" style="height:250px; border:1px solid #d0d0d0;">
</div>   
  
자 이제 실제 News Entity 연결 까지 성공했습니다. 이제 Service와 Repository까지 작성하여 실제 Database와의 연동을 진행해보도록 하겠습니다.   
    
***테스트케이스_3 작성***
```java
// NewsApiTest.java

@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
@Transactional
public class NewsApiTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper mapper;

    @Test
    public void insertNewsTest() throws Exception {
        //뉴스 생성
        News insertNews = new News();
        insertNews.setTitle("새로운 뉴스");
        insertNews.setContents("새로운 뉴스 컨텐츠");
        insertNews.setDate(new Date());
        insertNews.setAuthor("Taes-k");
        insertNews.setType("sports");

        MvcResult insertResult = mockMvc.perform(MockMvcRequestBuilders.post("/news")
        .contentType(MediaType.APPLICATION_JSON)
        .content(mapper.writeValueAsString(insertNews)))
        .andExpect(status().isOk())
        .andDo(print())
        .andReturn();

        String contents = insertResult.getResponse().getContentAsString();
        News insertResultNews = mapper.readValue(contents, News.class);

        //뉴스 불러오기
        mockMvc.perform(MockMvcRequestBuilders.get("/news/{id}",insertResultNews.getId()))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.title").value("새로운 뉴스"))
        .andDo(print());
    }

    @Test
    public void updateNewsTest() throws Exception {
        //뉴스 불러오기
        mockMvc.perform(MockMvcRequestBuilders.get("/news/{id}",1))
        .andExpect(status().isOk())
        .andDo(print());

        //뉴스 수정
        News updateNews = new News();
        updateNews.setTitle("수정된 1번 뉴스");
        updateNews.setContents("수정된 뉴스 컨텐츠");
        updateNews.setDate(new Date());
        updateNews.setAuthor("Taes-k");
        updateNews.setType("sports");

        mockMvc.perform(MockMvcRequestBuilders.put("/news/{id}",1)
        .contentType(MediaType.APPLICATION_JSON)
        .content(mapper.writeValueAsString(updateNews)))
        .andExpect(status().isOk())
        .andDo(print());

        //뉴스 불러오기
        mockMvc.perform(MockMvcRequestBuilders.get("/news/{id}",1))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.title").value("수정된 1번 뉴스"))
        .andDo(print());
    }
}

```

***컨트롤러 작성***

```java
// NewsController.java

@RestController
public class NewsController {

    @Autowired
    MainNewsServiceImpl mainNewsService;

    @RequestMapping(value="/news", method=RequestMethod.POST)
        public News insertNews(@RequestBody News insertNews) {

        return mainNewsService.insertNews(insertNews);
    }

    @RequestMapping(value="/news/{id}", method=RequestMethod.PUT)
    public News updateNews( @PathVariable ("id") int newsId,
                            @RequestBody News updateNews) {

        return mainNewsService.updateNews(newsId,updateNews);
    }

    @RequestMapping(value="/news/{id}", method=RequestMethod.GET)
    public Optional<News> getNews( @PathVariable ("id") int newsId) {

        return mainNewsService.getNews(newsId);
    }
}

```

***Service 작성***

```java
@Service
public class MainNewsServiceImpl implements NewsService {

    @Autowired
    NewsDao newsDao;

    @Override
    public News insertNews(News news) {

        return newsDao.save(news);
    }

    @Override
    public News updateNews(int id, News news) {

        news.setId(id);

        return newsDao.save(news);
    }

    @Override
    public Optional<News> getNews(int newsId) {

        return newsDao.findById(newsId);
    }
}
```

***테스트 결과 확인***

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_rest_api/test_success.png" style="height:250px; border:1px solid #d0d0d0;">
</div>   
  
***실제 데이터 결과***

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_rest_api/test_result.png" style="height:250px; border:1px solid #d0d0d0;">
</div>   


---



### <마무리>

지난챕터에서 미리 말씀드렸던것 처럼 TDD를 활용하여 Spring Rest API 서비스를 만들어보았습니다. 간단한 api와 간단한 테스트였기 때문에 어려움없이 진행 되었지만, TDD가 익숙하시지 않으신 분이라면 조금 어려우실수도 있겠으나 최대한 간단하게 진행했던 예제 프로젝트이기 때문에 위의 과정을 잘 이해하신다면 TDD를 이해하시고 앞으로의 프로젝트에서 적용하시는데 큰 도움이 되실것입니다.  
  
특히나 RestAPI서비스를 만들면서 Spring에서 지원하고있는 RestController와 JPA를 사용하여 굉장히 효율적으로 만들수 있었는데요 본문상에는 생략한 파일들이 많아 따라하기 어려우신 분들은 하단 샘플 프로젝트를 확인해주시면 감사드리겠습니다.

--- 

### 샘플 프로젝트 
{: .no_toc }

위 프로젝트는 다음 링크에서 확인하실수 있습니다.  
<https://github.com/taes-k/spring-example/tree/master/spring-rest-api>

---
