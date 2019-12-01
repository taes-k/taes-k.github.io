---
layout: post
comments: true
title: Tomcat 메모리 설정하기
tags: [server]
---

### 구동환경

OS : Ubuntu 16.04  
JAVA : JAVA8
TOMCAT : Tomcat8

### Tomcat out of memory

tomcat으로 서비스를 하다가 out of memory 오류가 나는경우가 있을것이다.
자주 발생하는 에러로는  
- `Java.lang.OutOfMemoryError: Java heap Space` Heap size error  
- `Java.lang.OutOfMemoryError: Permgen space` PermGen space error 

이런 메모리 오류의경우 여러가지 원인이 있을수 있으나 메모리공간의 부족으로 일어나는 에러로써, '일단은' 메모리 사이즈를 늘려주는 방법으로 문제를 해결 해 줄 수있다.   
실제로 에러가 났을때는, 원인파악이 가장 중요할것이다. 


### Tomcat 메모리 설정 
`tomcat 설치 폴더/bin/catalina.sh` 파일에 설정을 추가한다.  
```c
JAVA_OPTS="$JAVA_OPTS -Xms512m -Xms512m -XX:MaxPermSize=128m"
```
여기서,  
`Xms` heap size 최소값  
`Xmx` heap size  최대값   
`MaxPermSize` permanent size    

위설정의 변경으로 사이즈를 원하는대로 설정해 줄 수 있다.



  


