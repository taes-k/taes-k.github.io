---
layout: post
comments: true
title: java.util.date를 사용하지 말아야하는 이유
tags: [java]
---

### java.util.date 를 사용하지 말아야 하는 이유  

- time zone 이 없음  
- format이 없음  
- 캘린더 시스템이 없음  
- 1900년 후 몇년은 두자리수로 표현됨, 1900년 이전 처리와 함께 별도의 처리과정 사용됨  
- 월의 비직관적인 표시 (0 - 1월, 11- 12월)  
- mutable한 객체로서, thread-safe 하지 않다.  
- sql datetime과 연계시 별도의 java.sql.date를 사용한다.  

---

### 대체 api  

JAVA8부터 제공되는 java.time api사용


---

### 참고자료  

https://programminghints.com/2017/05/still-using-java-util-date-dont/

