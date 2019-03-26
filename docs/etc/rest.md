---
layout: default
comments: true
title: Rest
parent: etc
date: 2019.03.26
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### REST
Representational State Transfer의 약자로, WWW와 같은 분산하이퍼 미디어 시스템에서 사용되는 아키텍쳐이다.  
REST 의 원칙을 잘 지킨서비스를두고 `RESTful`이라 칭하기도 한다.


### REST의 3대 구성요소
- Resource  `http://taes-k.com/users`
  
- Method     `HTTP POST`
  
- Message   `{user:taeseong}`


### REST의 특성
1. 유니폼 인터페이스
HTTP 표준에 따른 어떤 플랫폼, 언어에 관계없이 사용가능하다. 
2. 무상태성 (Stateless)
세션과 같은상태를 따로 서버에 저장 하지 않고 들어오는 요청만을 메세지로 처리한다.
3. Cacheable
기존 HTTP에서 사용하던  E-Tag 캐시를 그대로 사용 가능하다.
4. 자체표현구조 (Self-descriptiveness)
API Resource 즉 URI 만으로 API 기능을 이해 가능하다.
5. Client-server
클라이언트가 로그인/세션을 관리하여 서버와 사용자간의 의존성이 줄어든다.



### REST의 주요 규칙
URI는 정보의 자원을 표현한다.
`POST http://taes-k.com/users`

[METHOD]
- POST : Create
- GET : Select
- PUT : Update
- DELETE : Delete

위 Method로써, 자원 처리 방법을 정의한다.

[RESOURCE]
`users`
명사 복수형으로 사용하며 세부 하위 내용들에 대해서는 `/`이후에 붙여나감으로써 정보에대한 내용을 확장시켜나간다.  
`GET https://taes-k.com/users/1`

추가적인 규칙으로는
- 소문자 사용
- "_" 언더바 대신 "-" 하이픈 사용
- 파일 확장자 사용 X
등이 있다.




