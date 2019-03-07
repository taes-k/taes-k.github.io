---
layout: default
title: Nginx SSL 설정
parent: server
date: 2019.03.02
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### SSL

일반적으로 HTTPS 에서 이용되는 프로토콜인 SSL(Secure Socket Layer)는 간단히 이야기해서, 웹서버와 브라우저 사이의 인증을 하는 기술이다.

SSL에 대한 자세한 설명은 <a href="/docs/server/ssl">SSL 이란?</a>  

이번 포스트에서는 Nginx 에서 SSL 설정하는 방법에 대해서 정리한다.  
  
nginx 환경설정 수정하기.
`vi /etc/nginx/sites-available/default`
```c
server{
listen 443; //https 포트
server_name www.mydomain.com;
ssl on;
ssl_certificate /파일경로/fullchain.pem; // 풀체인
ssl_certificate_key /파일경로/privkey.pem; // 인증키

location /{
	root path
      }
}
```

`listen 443` 을 통해 https 프로토콜로 들어온 request 에 대하여 ssl 키 인증을 거쳐 서비스가 구동되게 된다. 


