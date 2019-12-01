---
layout: post
comments: true
title: SSL과 HTTPS
tags: [server]
---

### SSL

HTTPS 에서 이용되는 프로토콜인 SSL(Secure Socket Layer)는 간단히 이야기해서, 웹서버와 브라우저(클라이언트) 사이의 보안을 위하여 CA라 불리는 제3인증을 통해 보증  한다.

### HTTPS 통신
일반적인 통신인 HTTP에보안이 추가된 +  over SSL 의 의미의 약자로써 HTTPS는 일반 HTTP의 보안적 단점인 평문(Plain Text)통신을 보완하기 위해 생겨났다. HTTP over SSL 의 약자처럼, 일반 HTTP 통신의 요청과 응답이 SSL 보안계층에의해 네트워크로 보내지기 전 암호화되어 전송되어진다.   

![https-security]({{ site.images }}/posts/2019-03-04-server-ssl/https-security.png)


### SSL 인증서

기본적으로 공인된 SSL 인증서를 발급받기 위해서는 공인된 CA로부터 인증서를 발급받아야 한다. 많이들 사용하는  CA 리스트는 아래와 같다.

- Let's encrypt (인증서 무료발급 가능, 기존에는 와일드카드 서브도메인 인증서 발급이 불가능 하였으나 최근 업데이트된것으로 확인.)
- Comodo
- Symantec
- StarSSL

등여러 기관들이 있는데 이들 기관마다 인증기간, 종류에따라 금액이 다르다. 기본적으로 인증서를 발급받는방법은 비슷하게 진행되는데 이후 포스트에서 Let's encrypt 인증서를 발급받아 무료로 HTTPS를 적용하는 예제를 포스팅 하겠다. 

인증서를 발급받게 되면 
- cert (인증서+공개키)
- chain (인증서 발급자)
- fullchain (인증서+인증서 발급자) --option
- privkey (개인키)
3~4가지 인증서가 받아지게 된다.
  


### SSL 인증 동작방식
기본적으로 SSL 인증절차를 받는이유를 다시한번 상기해보자.
> 클라이언트가 접속한 서버가 신뢰 할 수 있는 서버임을 보장하기위함  

![ssl-work]({{ site.images }}/posts/2019-03-04-server-ssl/ssl-work.png)

CA 확인을 통해 인증받은 서버임을 확인하고, 서버와의 일회용 세션키 생성으로 서버-클라이언트간의 통신을 암호화한다.









