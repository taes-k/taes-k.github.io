---
layout: default
comments: true
title: Tomcat으로 다중 서비스 구동하기
parent: server
date: 2019.03.06
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

하나의 서버, 하나의 tomcat으로 여러개의 서비스를 운영하고싶다면?

기본적으로, 탐캣에는 여러개의 포트를 설정해서 서비스를 돌릴 수 있도록 버츄얼호스트 (virtual host)기능을 지원하고 있다.

tomcat server conf 설정파일 `conf/server.xml`을 열어보면

```c
  <Service name="Catalina">
   <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               URIEncoding="UTF-8"
               redirectPort="8443" />

   <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />

   <Engine name="Catalina" defaultHost="localhost">

     <Realm className="org.apache.catalina.realm.LockOutRealm">
       <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
              resourceName="UserDatabase"/>
     </Realm>

     <Host name="localhost"  appBase="webapps"
           unpackWARs="true" autoDeploy="true">
       <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
              prefix="localhost_access_log." suffix=".txt"
              pattern="%h %l %u %t &quot;%r&quot; %s %b" />
     </Host>
   </Engine>
 </Service>

```
다음과 같은 Service, Engine, Host가 설정되어 있을것이다.

여기서 다중 서비스를 운영하려면 두가지의 방법이 있다. 
> 1. 같은 포트로 도메인만 나누어 운영하기 
> 2. 다른 포트로 나누어 운영하기

### 1. 같은포트로 도메인만 나누어 운영하기
```c
  <Service name="Catalina">
   <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               URIEncoding="UTF-8"
               redirectPort="8443" />

   <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />

     <Engine name="Catalina" defaultHost="localhost">

     <Realm className="org.apache.catalina.realm.LockOutRealm">
       <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
              resourceName="UserDatabase"/>
     </Realm>

     <Host name="test1.domain.com"  appBase="webapps1"
           unpackWARs="true" autoDeploy="true">
       <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
              prefix="localhost_access_log." suffix=".txt"
              pattern="%h %l %u %t &quot;%r&quot; %s %b" />
     </Host>

     <Host name="test2.domain.com"  appBase="webapps2"
           unpackWARs="true" autoDeploy="true">
       <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
              prefix="localhost_access_log." suffix=".txt"
              pattern="%h %l %u %t &quot;%r&quot; %s %b" />
     </Host>

   </Engine>
 </Service>
```
호스트 설정을추가하여 도메인이름으로 appBase를 나누어 운영한다.


### 2.다른 포트로 나누어 운영하기
```c
  <Service name="Catalina1">
   <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               URIEncoding="UTF-8"
               redirectPort="8443" />

   <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />

     <Engine name="Catalina" defaultHost="localhost">

     <Realm className="org.apache.catalina.realm.LockOutRealm">
       <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
              resourceName="UserDatabase"/>
     </Realm>

     <Host name="localhost"  appBase="webapps"
           unpackWARs="true" autoDeploy="true">
       <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
              prefix="localhost_access_log." suffix=".txt"
              pattern="%h %l %u %t &quot;%r&quot; %s %b" />
       <Context docBase="TEST1n" path="" reloadable="true" sessionCookiePath="/"/>
     </Host>
   </Engine>
 </Service>



  <Service name="Catalina2">
   <Connector port="8081" protocol="HTTP/1.1"
               connectionTimeout="20000"
               URIEncoding="UTF-8"
               redirectPort="8443" />

   <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />

     <Engine name="Catalina" defaultHost="localhost">

     <Realm className="org.apache.catalina.realm.LockOutRealm">
       <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
              resourceName="UserDatabase"/>
     </Realm>

     <Host name="localhost"  appBase="webapps"
           unpackWARs="true" autoDeploy="true">
       <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
              prefix="localhost_access_log." suffix=".txt"
              pattern="%h %l %u %t &quot;%r&quot; %s %b" />
       <Context docBase="TEST2" path="" reloadable="true" sessionCookiePath="/"/>
     </Host>
   </Engine>
 </Service>

```
Service를 추가로 설정하여 다른포트로 구동되도록 하며 context docBase를 통해 웹서비스 폴더를 구분하여 서비스를 동작 시킨다.


