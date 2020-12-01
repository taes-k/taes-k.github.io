---
layout: post
comments: true
title: 에러노트 - non-static variable this cannot be referenced from a static context
tags: [db, database, migration, column, vertica]
---

### 에러 메세지

`non-static variable this cannot be referenced from a static context`

우선 해당 에러메세지를 통해 `static context`에서 `non-static` varaible을 참조하여 발생한 에러로 파악됩니다.  

---

### 실행 환경

- Intellij IDE Ultimate 2019.04
- Intellij Lombok plugin 0.33
- Project java SDK 11 => 에러발생
- Project java SDK 8 => 정상수행

---

### 에러 코드

```java
@UtilityClass
public class UtilClazz
{
  public static void method()
  {
     InnerClazz obj = new InnerClazz(); 
  }

  private class InnerClazz
  {

  }
}
```

---

### 에러 원인 분석

에러 상황
- `reviewee` 프로젝트에서는 정상적으로 수행되었으나, `reviewer`의 프로젝트에서는 컴파일 오류가 발생함
- `reviewee` 프로젝트는 `JAVA SDK 8` 으로 세팅 되어 있었고, `reviewer`의 프로젝트는 `JAVA SDK 11` 으로 세팅되어 있었음
- IDE 상 에서는 별다른 에러 확인이 되지 않음

원인 분석
- Lombok `@UtilityClass`에 이슈가 있을것이라 판단
- JAVA SDK 8 의 경우 method, inner class 모두 static 붙여줌
- JAVA SDK 11 의 경우 method 에만 static 붙여줌

JAVA 8
```java
// case 1 -> 정상 수행
@UtilityClass
public class UtilClazz
{
  public static void method()
  {
     InnerClazz obj = new InnerClazz(); 
  }
  private class InnerClazz
  {
  }
}

// case 2-> 정상 수행
@UtilityClass
public class UtilClazz
{
  public void method()
  {
     InnerClazz obj = new InnerClazz(); 
  }
  private class InnerClazz
  {
  }
}
```

JAVA11
```java
// case 1 -> Error:(45, 49) java: non-static variable this cannot be referenced from a static context
@UtilityClass
public class UtilClazz
{
  public static void method()
  {
     InnerClazz obj = new InnerClazz(); 
  }
  private class InnerClazz
  {
  }
}

// case 2->  Error:(45, 49) java: non-static variable this cannot be referenced from a static context
@UtilityClass
public class UtilClazz
{
  public void method()
  {
     InnerClazz obj = new InnerClazz(); 
  }
  private class InnerClazz
  {
  }
}

// case 3 -> 정상 수행
@UtilityClass
public class UtilClazz
{
  public static void method()
  {
     InnerClazz obj = new InnerClazz(); 
  }
  private static class InnerClazz
  {
  }
}
```

---

### 해결방법

`@UtilityClass` 사용시 Inner class에 `static`을 명시적으로 붙여줘야 합니다.  

저희 팀 에서는 `@UtiliyClass` 사용시 static 선언시 sonar에 걸리는 이슈 등의 이유도 포함해 롬복 어노테이션 사용시 IDE 상에서 java 기본문법이 숨겨져서 `@UtiliyClass`대신에 직접 UtilityClass를 만들어 사용하는쪽으로 규칙을 잡고 수행하기로 해결하였습니다.

```java
public class UtilClazz
{

    private UtilClazz()
    {
        throw new IllegalStateException("Utility class");
    }
    public static void method()
    {
        ...
        InnerClazz obj = new InnerClazz(); 
        ...
    }

    private static class InnerClazz
    {
        ...
    }
}
```