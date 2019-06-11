---
layout: default
comments: true
title: 1.6 Spring build tools
parent: 요령과 기본(Spring)
date: 2019.05.31
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

## 1.6 Spring build tools
{: .no_toc }
 이번챕터에서는 어쩌면 그냥 모르고 사용하고있었던 빌드 툴 (Maven, Gradle)에 대해서 알아보도록 하겠습니다. 저는 개인적으로 'maven은 xml, gradle은 gradle로써 dependency를 관리한다'라고만 알고 있었는데요,  build tool의 역할과 종류별 특징 및 차이점들을 알아보도록 하겠습니다.  
  
## Contents List
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Build tool

 스프링 부트로 프로젝트를 생성 해보셨다면 아시겠지만, 프로젝트 이름과 함께 가장 먼저 설정 하는것중 하나가 Type을 설정하는 옵션이 있습니다.  여기서 설정하는 type이 바로 build tool을 설정하는 옵션입니다.  

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_build_tools/spring_boot_start_build_tool_option.png" style="height:400px; border:1px solid #d0d0d0;">
</div>   
    
스프링에서는 maven과 gradle 두가지를 지원하고있으며, 프로젝트를 생성했을때 시각적으로 보이는 차이는 maven은 pom.xml , gradle은 build.gradle과 setting.gradle 이 생성된다는 차이가 있습니다.   
  
자 이제 build tool의 역할에대해서 알아보도록 하겠습니다.  먼저 이름에서 알 수 있듯이 build tool은 'build'작업을 도와주는 도구입니다. 'build'란 단순히 코드를 컴파일해주는 작업 뿐만아니라 테스팅, 검사, 배포까지의 일련의 작업들을 통틀어 빌드라고 할 수 있습니다. 

- 소스코드 컴파일
- jar파일 등의 컴포넌트들 패키징
- 테스팅
- JAVA Doc 생성
- 배포
- 라이브러리 자동 추가 및 관리
- 라이브러리 버전 자동 동기화  

위와같은 빌드툴들이 제공하는 기능들이 있는데, Spring에서 IDE에서 자체적으로 빌드를 하는것이아닌 외부의 Build tool을 이용하여 빌드를 진행하는 이유는 프로젝트가 IDE에 종속시키는것이 아니라 외부의 Build tool에 종속시켜 공통적인 빌드 환경을 제공해주기 위함입니다.   
  
이 뒤에서부터는 Maven과 Gradle 의 차이를 중심으로 빌드툴에대해 알아보도록 하겠습니다.

---

## <1.6.1 Maven>
## Maven

Maven은 2004년에 출시된 Apache사의 build tool 로써,  기존의 built tool이었던 Ant의 단점들을 보완한 tool입니다. pom.xml 파일을 통해 정형화된 빌드 시스템으로써 전체적인 프로젝트를 관리해줍니다.   
  
Maven은 플러그인을 구동해주는 프레임워크로써, 모든 작업은 플러그인에서 수행하게 됩니다. 이에대한 이해를 돕기위해 Maven의 구조도를 확인해보도록 하겠습니다.  

<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_build_tools/maven_structure.png" style="height:300px; border:1px solid #d0d0d0;">
</div>   
  
Maven 위와같은 구조도로써 build를 진행하게 되는데  Plugin 내에서도 Goal의 세부 작업단위로써 life-cycle을 만들어 실행하게됩니다. 간략한 빌드과정은 다음과 같습니다.  
 
Celan (이전빌드 파일 삭제 ) -> init -> compile -> test-compile(테스트코드 컴파일) -> test -> package -> integration-test -> verify(적합검사) ->install(패키지 로컬에 설치) -> deploy -> site(사이트문서생성)  
    
Maven의 상세한 내용은 다음 링크에서 확인하실수 있습니다. <https://maven.apache.org/plugins/index.html>
  
## pom.xml Example

```c
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>start.spring</groupId>
    <artifactId>spring-start</artifactId>
    <version>0.0.1</version>
    <packaging>jar</packaging>

    <name>FITT_OAUTH</name>
    <description>FITT_OAUTH</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-rest</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```
## <1.6.2 Gradle>
## Gradle

Gradle은 2012년에 출시된 Gradle사의 build tool 로써, 후발주자인 만큼 Maven의 장점과 그전의 Ant의 장점을 살린 build tool 입니다.  Gradle은 JVM기반의 빌드도구로써, xml이 아닌 Groovy 기반으로 설정파일을 사용합니다.  

Gradle은 Task기반으로 작업을 수행하며, Task는 종속성에 따라 다음 작업할 Task를 연결하여 작업을 진행합니다. 이에대한 Gradle 공식 문서에는 다음과같은 다이어그램으로 설명하고 있습니다.   
  
<div style="text-align:center;">
<img src="https://taes-k.github.io/assets/images/trick_basic/spring_build_tools/gradle_structure.png" style="height:300px; border:1px solid #d0d0d0;">
</div>   

Gradle도 Maven과 비슷한 과정으로 빌드가 진행됩니다.  
clean -> check -> test -> assemble(package) -> complile -> upload(deploy)
  
추상적인 동작 과정에서는 Maven과 큰 차이를 보기는 어렵지만, Gradle에서 소개하는 바로는 작업의 입/출력을 추적해 필요한 작업만 실행하고 빌드과정에서 캐시를 활용하여 Maven과 비교하여 Performance적인 측면에서 최대 100배까지 차이가 날 수 있다고 명시해두고 있습니다. <https://gradle.org/maven-vs-gradle/>

Gradle의 상세한 내용은 다음 링크에서 확인하실수 있습니다.  <https://docs.gradle.org/current/userguide/build_lifecycle.html#build_lifecycle>


## build.gradle Example  

```c
plugins {
    id 'org.springframework.boot' version '2.1.5.RELEASE'
    id 'java'
}

apply plugin: 'io.spring.dependency-management'

group = 'start.spring'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

---

## <마무리>
이번챕터에서는 Build tool에 대해 알아보고 Maven과 Gradle의 차이에대해 알아보았습니다. Maven과 Gradle이 제공하는 기능 들은 거의 비슷하나 사용성이나 성능에있어서는 Gradle이 비교적 좋은 성능을 가지고 있다고 합니다.  
  
개인적인 의견으로써 말씀드리자면, 'Maven을 안쓸 필요도 없고 Gradle을 쓸필요도 없다' 라고 말씀드리고 싶습니다. 기존의 프로젝트에서 Maven을 사용하셨으면 그대로 maven을 사용하셔도 좋고, 빌드시간을 조금이라도 줄여보고자 하신다면 Gradle의 도입을 고려해보시는것도 좋을것 같습니다. 만약 새로운 프로젝트를 진행하시려고 하신다면 Gradle의 도입을 긍정적으로 검토해보셔도 좋을것 같습니다.

--- 

## 참조 문서
{: .no_toc }
Spring 5.1.6 release docs  
<https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux>


---
