---
layout: default
comments: true
title: (요령과 기본) 2.1 Spring 프로젝트 시작하기 (1) - MVC
parent: 요령과 기본
date: 2019.05.28
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

## 2. 당신은 Spring을 어떻게 사용하고 있는가?
{: .no_toc }
1장 에서는 Spring을 사용하는 이유에 대해서 알아보기 위해 Spring에서 제공해주는 기능들을 위주로 알아보았습니다. 그렇다면 이번에는 '나는 Spring을 어떻게 사용해 왔는가?'에 대한 질문을 드리도록 하겠습니다.  
  
이번에도 먼저 저의 답은 '필요한 기능들을 구글링 해가며 필요한 소스들을 찾아 적용시켜왔다'입니다. 물론 정답은 없지만, 스스로 채점을 해보자면 50점을 줄만한 답인것 같습니다. 막힐때 마다 필요한 기능들을 잘 찾아 적용시켰으나, 적용시키는 '요령'만 습득하고 그 작동원리인 '기본'까지는 습득하려 하지는 않았기때문입니다.   
    
이번장 에서는 실제로 Spring 웹 프로젝트를 만들어보면서 서비스 구축을 위한 '요령'과 함께 동작원리 '기본'까지 알아가보도록 하겠습니다.  

---

## 2.1 Spring 프로젝트 시작하기 (1) - MVC
{: .no_toc }
1.4 챕터에서 저희는 Spring MVC가 어떻게 작동하는지 알아보았습니다. 해당내용을 바탕으로 이번챕터에서는 직접 MVC 프로젝트를 만들어보도록 하겠습니다.  
[1.4 Spring MVC의 진짜 의미](https://taes-k.github.io/docs/trick_basic/1_4_about_spring_mvc/)  
  

## Contents List
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 프로젝트 시작하기  
  
우리는 2장에서 뉴스 큐레이션 웹사이트를 만들어 나가도록 하겠습니다. 뉴스는 몇가지 테마별로 구성되어있으며 로그인한 사용자는 뉴스를 스크랩하거나 좋아요 등록이 가능합니다. 이 프로젝트는 2장에서 실습할때 계속해서 확장시켜 나갈 예정입니다.   
  
해당 프로젝트의 플로우차트는 다음과 같습니다.  

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_start_1_mvc/mvc_start_flowchart.png" style="height:400px; border:1px solid #d0d0d0;">
</div>   

---

## MVC Project start

자 이제  MVC 프로젝트 만들기를 시작하도록 하겠습니다. 상세 구동환경은 다음과 같습니다.   
  
- 구동환경  
STS 3.9.5
Spring boot 2.1.5
JAVA 8
Gradle 3.x
  
- 디렉토리 구조  

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_start_1_mvc/mvc_start_directory.png" style="height:250px; border:1px solid #d0d0d0;">
</div>   

- MVC 기본 골조 세팅  
가장먼저, 사용자의 요청과 응답을 직접적으로 처리해주는 Controller부터 세팅을 하도록 하겠습니다.  

```java
// ViewController.java

@Controller
public class ViewController {

    @Autowired
    MainNewsServiceImpl mainNewsService;

    @RequestMapping("/")
    public String index(Model model) {

        model.addAttribute("newsData",mainNewsService.getNews());
        return "index";
    }
}
```
먼저 화면처리를 위한 ViewController를 만들어주어 index를 매핑해주었습니다. `/` url으로 접근한 클라이언트의  request는 디스패쳐서블릿으로 전달 되어진 후 @Controller와 @RequestMapping 어노테이션에 의해 url이 mapping되어 일련의 처리 과정을 거친 후 "index" String을 반환해 ViewResolver 설정에 따라 index view를 보여주게 될 것입니다.  
  
다음은, index 페이지에서 뉴스기사들을 불러오기위한 newsService를 만들어줍니다. 이때 서비스는 추후에 뉴스의 카테고리별로 확장될수 있기때문에 Interface와 구현체를 따로 나눠 서비스를 구현해주도록 하겠습니다.  
```java
// NewsService.java
public interface NewsService {

    public List<News> getNews();

}
```  

```java
// MainNewsService.java
@Service
public class MainNewsServiceImpl implements NewsService {

    @Override
    public List<News> getNews() {
        List mainNews = new ArrayList();

        News news1 = new News();
        news1.setTitle("중고차 매매 후 한 달 내 고장나면 보험 보상받는다");
        news1.setContents("중고차 구입 소비자의 피해를 구제하기 위한 중고차 보험이 다음 달부터 의무화되면서 구입 한 달 이내의 중고차 고장은 보험으로 보장을 받을 수 있게 된다.");
        news1.setDate("2019-05-27 13:30");
        news1.setAuthor("Taes");
        news1.setType("financial");

        News news2 = new News();
        news2.setTitle("하늘 참 신기하네");
        news2.setContents("오전 대구 달서구청 옥상 정원에서 시민들이 하늘에 펼쳐진 이색적인 구름을 관찰하고 있다. 파란 하늘에서 잘라낸 듯 넓게 펼쳐진 구름대와 차고 습한 대기 속을 비행하는 항공기가 남기는 가늘고 긴 구름 ‘비행운(오른쪽 가늘고 긴 구름)’이 겹쳐지며 진기한 풍경을 연출하고 있다. ");
        news2.setDate("2019-05-27 09:30");
        news2.setAuthor("Taes");
        news2.setType("social");

        News news3 = new News();
        news3.setTitle("커쇼 11연승, 다저스 4연승");
        news3.setContents("커쇼가 11연승에 성공했다. 벨린저는 19호 홈런과 함께 팀을 구해낸 두 개의 외야 어시스트를 기록했다. 오클랜드는 10연승을 질주. ");
        news3.setDate("2019-05-27 11:35");
        news3.setAuthor("Taes");
        news3.setType("sports");


        mainNews.add(news1);
        mainNews.add(news2);
        mainNews.add(news3);

        return mainNews;
    }
}
```
위와 같이 메인 뉴스 데이터를 제공하는 서비스까지 작성이 완료되었습니다. 현재는 샘플 데이터를 서비스부분에 담아두어 구조가 이상해보이지만, 추후 챕터에서 변경 해 나갈 예정입니다.   
  
다음으로는 데이터 객체인 Member VO 를 정의해보도록 하겠습니다.   
  
 ```java
public class News {
    String title;
    String contents;
    String date;
    String author;
    String type;

    public String getType() {
        return type;
    }
    public void setType(String type) {
        this.type = type;
    }
    public String getTitle() {
        return title;
    }
    public void setTitle(String title) {
        this.title = title;
    }
    public String getContents() {
        return contents;
    }
    public void setContents(String contents) {
        this.contents = contents;
    }
    public String getDate() {
        return date;
    }
    public void setDate(String date) {
        this.date = date;
    }
    public String getAuthor() {
        return author;
    }
    public void setAuthor(String author) {
        this.author = author;
    }

}
```  

News VO는 간단하게 제목(Title), 내용(Contents), 날짜(Date), 저자(Author), 타입(Type) 5가지의 값들로 정의하였습니다.  VO(Value Object)는 데이터를 Objeect 단위로 변경해서 관리하기 위한 클래스입니다. 사실 VO를 꼭 사용해야하는가에 대한 논쟁은 계속해서 있지만, 데이터를 객체화 시켜 전달해주는 목적보다도 개발자들에게 데이터를 명시적으로 '정의'해 줄수 있다는점에서 사용하는것이 좋다고 말씀드리고 싶습니다.  
  
다음은 view를 처리할 차례입니다. 이번 프로젝트에서는 jsp를 사용하여 view를 제공하려고하는데 이를 위해서는 간단한 사전 설정이 필요합니다.   

```c
//build.gradle

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    compile 'org.apache.tomcat.embed:tomcat-embed-jasper'
    compile 'javax.servlet:jstl:1.2'
}
```

```c
// application.properties

spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
```
jsp 사용을 위한 gradle 혹은 maven dependency 설정이 필요합니다.  또한 view의 위치 및 확장자명 설정을 위해 appliciation.properties에서 spring mvc view prefix, suffix을 해주셔야 컨트롤러에서 편리하게 view name만으로 return을 보내줄수 있습니다.   
  
자, 이제 클라이언트가 `/`를 요청할때 뉴스데이터를 포함시켜 index페이지를 넘겨주는 과정까지 완료했습니다.  이과정이 잘 이루어 졌는지 마지막으로 `index.jsp`를 작성하며 알아보도록 하겠습니다.  
  
```c
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>Trick&Basic News</title>
    </head>
    <body>
    
    <div class="header" style="width:100%;height:100px;line-height:100px;border-bottom:1px solid #d0d0d0">Trick&Basic News</div>
    
    <div class="news-container">
        <c:forEach var="el" items="${newsData}">
            <div class="news" style="margin-bottom:20px;">
                ${el.title}<br>
                ${el.type} / ${el.author} / ${el.date}<br>
                ${el.contents}
            </div>
        </c:forEach>
    </div>
    
    </body>
</html>
```

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_start_1_mvc/mvc_start_result.png" style="height:350px; border:1px solid #d0d0d0;">
</div>   

리스트로 넘겨진 'newsData'값을 출력하기위해 jstl forEach 문으로 처리하여 원하던 데이터가 view까지 잘 전달되어 출력된것을 할 수 있습니다. 



---

## <마무리>
이번 챕터는 간단한 MVC의 골조만 잡아 주었다고 할수 있겠습니다. 다음챕터에서는 JDBC를 이용해 데이터를 DB에 저장시켜두고 사용할수 있도록 해 보겠습니다.

--- 

## 샘플 프로젝트 
{: .no_toc }

위 프로젝트는 다음 링크에서 확인하실수 있습니다.  
<https://github.com/taes-k/spring-example/tree/master/spring-mvc-start>


---
