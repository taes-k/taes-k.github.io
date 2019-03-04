---
layout: default
title: spring security 적용하기-2
parent: spring
date: 2017.02.06
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

별도 로그인 페이지를 구축하여 적용해보기

1) `security-context.xml`
login page 는 리다이렉트가 일어나지 않도록 access 권한을 전체공개로 풀어준다.
<form login> 탭에서 로그인 기능들 이 설정됨

`login-page` : 로그인 페이지 설정  
`username-parameter` : 로그인페이지에서의 id name 값 설정   
`password-parameter` : 로그인페이지에서의 pw name 값 설정  
`login-processing-url` : 로그인페이지에서의 form action 값 설정  
`default-target-url` : 로그인 성공후 URL  
`authentication-failure-url` : 로그인 실패후 URL  
  
~~~c
	<http auto-config="true"> 
		<intercept-url pattern="/login" access="IS_AUTHENTICATED_ANONYMOUSLY" />
		<intercept-url pattern="/*" access="ROLE_USER" /> 
		
		<form-login login-page="/login" 
					username-parameter="id" 
					password-parameter="pw"		
					login-processing-url="/loginProcess"
					default-target-url="/login_success" 
					authentication-failure-url="/login"	
					always-use-default-target='true'
					/>
					
	</http> 
~~~


2)`login.jsp`
위에서 설정해준 설정으로 name 세팅하여 동작시켜준다.
~~~c
<form name="form1" method="post" action="loginProcess">
	<div class="login-inputBox"> 
		<div>
			<div style="margin-bottom: 7px;">아이디</div>
			<input name ="id" type="text"/>
		</div>
		<div>
			<div style="margin: 18px 0 7px;">비밀번호</div>
			<input  name ="pw" type="password" />
		</div> 
		<input type="submit" id="loginSubmit" value="로그인"/>
	</div>
</form>
~~~

