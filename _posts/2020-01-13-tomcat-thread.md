---
layout: post
comments: true
title: Tomcat 쓰레드 설정
tags: [server, tomcat]
---

### Tomcat 과 쓰레드
WAS의 종류중 하나인 Tomcat은 기본적으로 접근하는 request들에 대해 Thread를 할당하여 작업을 수행하도록 해줍니다.  
해당 작업을 위해서 Thread pool을 사용하여 쓰레드를 재사용 할 수 있게 해주고 max-thread, connection timeout 등의 설정을 사용자가 직접 설정 해 줄 수 있습니다.   

---

### Tomcat 쓰레드 설정
Tomcat8 기준, default 설정값

- maxThread = 200 (쓰레드풀 최대 쓰레드 갯수)
- minSpareThreads = 25 (쓰레드풀 초기 쓰레드 갯수)
  
위와 같은 값으로 설정되어 있으며 해당값은 tomcat의 Server.xml에서 조정이 가능합니다.  

```c
<Connector port="8080" address="localhost" maxThreads="250" maxHttpHeaderSize="8192"
emptySessionPath="true" protocol="HTTP/1.1" enableLookups="false" redirectPort="8181" 
acceptCount="100" connectionTimeout="20000" disableUploadTimeout="true" />
```

Tomcat8 Doc(https://tomcat.apache.org/tomcat-8.0-doc/config/executor.html)
