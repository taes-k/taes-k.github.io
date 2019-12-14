---
layout: post
comments: true
title: Spring Security (1) - 로그인 세션
tags: [요령과 기본, Spring, Spring-security]
date: 2019.06.12
---

### 2.6 Spring Security (1) - 로그인 세션
{: .no_toc }
Spring 에서는 보안과 인증, Access control을 할수 있는 Spring Security 프레임워크를 제공합니다. 이번 챕터에서는 Spring Security를 알아보기전, 고전적인 세션을 통한 로그인 관리에 대해서 알아보도록 하겠습니다.  

### Contents List
{: .no_toc .text-delta }

1. TOC
{:toc}

---

### 2.6.1) 세션과 쿠키

이미 많이 알고계시겠지만 Spring Security에서도 사용되는 세션과 쿠기이기때문에 먼저 개념을 정리하고 가보도록 하겠습니다.  세션과 쿠키는 HTTP 프로토콜에서 상태유지를 목적으로 사용되는 것으로, HTTP의 특징인 'Stateless'를 보완하기위해 사용됩니다.  
  
---
  
#### 쿠키(Cookie)  
위의 설명에서 쿠키는 '상태유지'를 목적으로 사용되는것이라고 했는데, 쿠키는 '상태'를 클라이언트의 로컬에 저장해 참조하는 방식으로 동작하고 브라우저를 통해 확인이 가능합니다.   

![1]({{ site.images | relative_url }}/posts/trick_basic/2019-06-12-spring-security-1/1.png) 
    
쿠키는 이름, 값, 도메인, 경로, 기한의 데이터를 가지며 클라이언트가 따로 설정하지 않아도 도메인과 경로에 따라 브라우저에서 Request시 기한내의 쿠키를 Header에 세팅하여 서버로 전송해 '상태'를 함께 전송 할수있게 해줍니다.   

![2]({{ site.images | relative_url }}/posts/trick_basic/2019-06-12-spring-security-1/2.png) 

---
    
#### 세션(Session)

세션 역시 '상태유지'를 목적으로 사용됩니다. 다만 쿠키와 다른점은, '상태'를 서버에 저장해 참조하게됩니다. 하지만 HTTP의 특성상 세션만으로는 클라이언트를 구별하고 상태를 유지할수 없기때문에 쿠키와 함께 사용되게 됩니다.  

![3]({{ site.images | relative_url }}/posts/trick_basic/2019-06-12-spring-security-1/3.png) 
   
쿠키로써 SESSIONID별로 클라이언트를 구별해 서버측에서 key:value 구조로 세션을 저장하고 참조 합니다.  

---

### 2.6.2) 로그인 서비스 구현

이제 위에서 알아본 세션과 쿠키를 통해 로그인서비스를 구현해보도록 하겠습니다. 이번 챕터는 따로 예제프로젝트를 제공하지않고 개념적으로 컨트롤러를 구현하면서 설명드리도록 하겠습니다.  
  
---

####  Controller 구현
  
```java
// LoginController.java

@Controller
public class LoginController.java {

    @Autowired
    LoginService loginService;

    @RequestMapping(value="/loginProcess")
    public String loginProcess(HttpSession session,
                             @RequestParam(value="id") String id, 
                             @RequestParam(value="pw") String pw) {
                            
        if(loginService.loginCheck(id, pw)){ // id,pw검사를 통해 True,false를 return
            session.setAttribute("loginCheck",true);
            session.setAttribute("id",id);
            return "index";
        }else{
            return "login";
        }
    }
    
    @RequestMapping(value="/logoutProcess")
    public String logoutProcess(HttpSession session) {
                            
        session.setAttribute("loginCheck",null);
        session.setAttribute("id",null);
        
        return "index";
    }
    
    @RequestMapping(value="/needLogin")
    public String needLoginPage(HttpSession session) {
    
        //세션 검사를 통해 Access control
        if(session.getAttribute("loginCheck")!=null){
            return "needLogin";
        }else{
            return "login";
        }
    }
    
}

```
  
위의 `loginProcess`를 통해 로그인 정보를 session으로 등록시켜주고 `logoutProcess`에서 session 정보를 삭제함으로써 로그인 여부를 컨트롤 할 수 있습니다. 이 세션을 통해 `needLoginPage`에서 Access control 을 해주는 예제를 확인 하실수 있습니다.  

---

#### View 구현

```jsp
// index.jsp

<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html>
<html>
<head></head>
<body>

<c:choose>
    <c:when test="${sessionScope.loginCheck eq true}">
        ${sessionScope.id} 님이 로그인 되었습니다.  
        <button type="submit">로그아웃</button>
    </c:when>
    <c:otherwise>
        <form id="loginForm">
            <input name="id"/>
            <input name="pw"/>
            <button type="submit">로그인</button>
        </form>
    </c:otherwise>
</c:choose>

</body>
</html>

```

controller에서 설정해둔 session을 위와같은 jsp에서의 처리를 통해 view단에서 로그인 세션관리를 해줄수 있습니다.

---

### 마무리

위와같은 인증,보안 프로세스는 현재는 잘 사용하지않는 프로세스이기 때문에 실제 예제프로젝트를 작성하지 않고 개념적으로만 정리해보았습니다. 다만 위에서 사용한 쿠키와 세션의 개념은 Spring Security에서도 똑같이 사용되기 때문에 잘 이해해두시길 바랍니다. 다음 챕터에서는 Spring Security를 이용한 인증을 알아보도록 하겠습니다.

---
