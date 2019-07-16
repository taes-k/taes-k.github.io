---
layout: default
comments: true
title: JAVA Class loading 과정
parent: java
date: 2019.07.16
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### 개요
이번 포스팅에서는 자바에서는 파일을 어떻게 동작을 시키는지를 알아보도록 하겠습니다. 
  
---

### 실행 과정
1\. Compile  
  
보통 JAVA로 코딩을 하게되면, `.java` 파일형식으로 작성을 하게되는데 해당 파일은 컴퓨터에서 바로 실행가능한 파일이 아닙니다. 실행간으하게 만들기위해 컴파일 과정을 거치게 되는데, 이 과정으로 통해서 `.java`파일은 `.class`파일로 컴파일 되게 됩니다.
  
2\. Load  
  
이제 실행시킬수 있는 `.class`파일이 만들어 졌기때문에 해당 파일을 메모리상에 올려 실행시켜야 합니다. 해당과정을 Class loading이라 하며, 이 과정을 클래스 로더(Class Loader)가 주체적으로 수행하게 됩니다.  클래스로더 여러개로 구성되어 있어 유저가 작성한 클래스뿐만 아니라 목적에 따라 부트스트랩 클래스, 확장클래스등의 로드를 함께 수행합니다.  실제적인 로더의 동작 순서는 다음과 같습니다.  
  
BootstrapClassLoader -> Extensions Class Loader -> System Class Loader -> User-Defined Class Loader  
  
3\. Link  
  
링크과정에서는 Verification, prepare, resolve하는 과정을 진행합니다.  
- Verification(확인) 과정을 통해 로드된 클래스의 바이트구조가 올바른지 검사해 보안을 확인합니다.  
- Prepare(준비) 과정을 통해 클래스 및 인터페이스의 static 필드를 생성하고 기본값으로 초기화 합니다.  
- resolve(해석) 과정을 통해 인스턴스들의 실제 주소값에대한 심볼릭 링크를 설정해줍니다.  
  
4\. Initialization   
Link단계에서 기본값으로 초기화된 static 필드들에 대해 정의된 값을 지정해줍니다.   

5\. Run main
main 을 찾아 main 스레드를 실행 시켜줍니다.  
  
### 클래스 로더의 특징  
1\. 계층적 구조   
위에서 클래스 로더의 실행순서를 보면 알수있다시피, bootstrap으로 부터 시작되는 클래스로더들이 계층적으로 구성되어있음을 알수 있습니다.   
  
2\. unload 불가  
한번 class loader에 의해 로드된 클래스는 해제될수 없습니다.  
  
### 로드타임 로딩 & 런타임 로딩

일반적으로 클래스를 로딩하는 로드타임에는 해당 class및 import로 주입되어있는 클래스들을 로드하게됩니다.  
```java
import java.xxx.xxx;

public class Example{
    public static main(String[] args){
        //something code;
    }
}
```  
  
예를들어 위와같은 class를 로드할때는 import되어있는 java.xxx.xx를 함께 로드학되는데 타겟 클래스를 로드하는 시점에서 함께 일어나는 로드들을 로드타임 로딩이라고 합니다.  
  
그렇다면 런타임 로딩은 언제일어나게 될까요?  
이름에서 알수 있다시피 런타임시 로딩이 일어나게 됩니다.  

```java
import java.xxx.xxx;

public class Example{
    public static main(String[] args){
    //something code;
        Class dynamicClass = Class.forName("className");
        Object obj = dynamicClass.getInstance();
    }
}
```  

위와같은 코드를 작성했을때, main 이 실행되면서 Class.forName() 호출을 통해 새로운 클래스를 참조하게 됩니다. 현재 main 함수를 동작시키는 중에  class를 로드하게되는데, 이것을 런타임 로딩이라고 합니다.  
  
---
### 마무리
사실 이번 내용은 어쩌면 몰라도 그만인 내용들일수 있지만, 실제 작성한 java 코드들이 jvm내에 어떻게 로드되고 실행되는지에 대한 과정들을 알고 코딩을 할 수 있기를 바랍니다.  
