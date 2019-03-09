---
layout: default
comments: true
title: Tomcat admin manager 사용하기
parent: server
date: 2019.03.8
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### Tomcat admin
먼저, tomcat admin에서 어떤기능들을 사용 할 수 있는지 살펴보자.  
admin manage 페이지는, tomcat 설치시 tomcat-admin을 함께 설치해야 사용 가능하며 `웹서버주소/manager/html`으로 접속 가능하다.  
  
tomcat 계정 설정이 안되어있다면 다음 설정 파일에서 계정을 만들어 준뒤에 사용이 가능하다.  

`tomcat 설치 루트/conf/tomcat-user.xml`

```
  <role rolename="manager-gui"/>
  <user username="admin" password="admin" roles="manager-gui"/>
```

### Tomcat admin page

> ![tomcat-admin page]({{ site.images }}/docs/tomcat-admin-start/tomcat-admin-start_1.png)

#### 확인 가능한 내용
- Applications (page 별 동작상태 확인)
- Deploy (배포 설정)
- Diagnostics (memory 문제 확인)
- Server Information (서버정보 확인/ 탐캣버전, JVM 버전, OS 정보 등)


### Tomcat status page

> ![tomcat-admin page]({{ site.images }}/docs/tomcat-admin-start/tomcat-admin-start_2.png)

#### 확인 가능한 내용
- JVM (메모리 상태 확인)
- http (IP 확인)



