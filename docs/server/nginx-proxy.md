---
layout: default
title: Nginx proxy 설정
parent: server
date: 2019.03.02
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---
`부제 : Nginx 도메인 + 프록시설정으로 마이크로서비스 운영하기`

### Nginx

경량 웹서버 소프트웨어로, 아파치와 비교가 많이되며 사용됨.  
Nginx의 활용중 하나인 프록시 서버로써의 Nginx 활용법을 정리해두고자 한다.

![nginx-proxy]({{ site.images | relative_url }}/nginx-proxy/nginx-proxy.png)

위의 모식도와 같이 여러개의 서비스 + 도메인을 하나의 물리적 서버로 운용하고자 할때 Nginx 프록시 설정으로 포트를 나누어 운영이 가능하다.

nginx 환경설정 
`/etc/nginx/sites-available/default`

```c
server{
listen 80;
server_name user.api.kr;

location / {
        proxy_pass      http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
	}

}

server{
listen 80;
server_name work.api.kr;

location / {
        proxy_pass      http://localhost:8081;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
	}

}

server{
listen 80;
server_name news.api.kr;

location / {
        proxy_pass      http://localhost:8082;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
	}

}
```

위와 같이 80port 로 들어오는 request들을 `server_name` (도메인네임)으로 판별하여 서버내 설정해둔 다른 포트로 접속할수 있도록 프록시서버 역할을 해주게 된다.  
   
위와 같은 방식으로 여러개의 서비스를 간단하게 운용할수 있다.

 

