---
layout: post
comments: true
title: Jenkins - 내 PR을 자동으로 코드리뷰 받게하자! 
tags: [jenkins, sonarqube, checkstyle, ci]
--- 

### CI

들어가기 앞서서, `CI`에대해 짧게나마 설명을 드리고 가도록 하겠습니다.  
`Continuous Integration`약어인 `CI`는 일반적으로 `지속적 통합`이라는 용어로 번역되는데, 이용어의 유래는 위키백과에 잘 설명되어있어 내용을 발췌합니다. 

> 소프트웨어 개발에서 각 소프트웨어 개발자가 작업한 변경점을 프로젝트의 원래 소스 코드에 자주, 빠르게 통합하는 것이다.
> 
> 개발자들이 저장소에 코드를 제출하려면, 먼저 자신이 코드를 받았던 때부터 현재까지 저장소 코드의 변경 내용을 자신의 코드에 반영되도록 자신의 코드를 업데이트한 후 자신의 코드를 제출해야 한다. 저장소에 변경된 내용이 많을수록, 개발자들이 자신의 작업 내용을 제출하기 전에 해야 할 일이 많아진다.
>
> 언젠가는 저장소가 개발자들의 베이스라인과는 너무 많이 달라지게 되는 "통합의 지옥" 이라 불리는 상황에 빠지게 된다. 이 경우, 작업하는 시간보다 작업 내용을 통합하는데 걸리는 시간이 더 걸리게 되어, 최악의 경우 개발자들이 자신들의 변경 내용들을 취소하고 작업들을 완전히 처음부터 다시하는 것이 나을 수도 있다.
> 
> 지속적인 통합은 초기에 그리고 자주 통합해서 "통합의 지옥"의 함정을 피하는 것을 내포하고 있다.  
> 지속적인 통합은 재작업을 줄여서 비용과 시간을 줄이는데 초점이 맞추어져 있다.
> 
> https://ko.wikipedia.org/wiki/%EC%A7%80%EC%86%8D%EC%A0%81_%ED%86%B5%ED%95%A9

이제는 대부분의 현업 개발자 분들이라면 이미 사내 시스템에 `CI,CD`가 구성이 되어 있으실거라 생각합니다.  

특히나 CI에 정적분석 툴을 연동하여 일명 `'냄새나는 코드'`들을 찾아내고 계실텐데, 이걸 확인하기 위해서 별도의 페이지에서 확인하는것이 여간 귀찮을 일이 아닐수 없습니다.

이번 포스팅에서는 Jenkins를 활용하여 PR을 올릴때마다 Sonarqube, checkstyle 에서 나온 이슈들을 코드리뷰 comment로 받아볼수 있도록 `통합`하는 과정을 소개합니다.

해당 예제에서 사용된 환경들은 다음과 같습니다.

- Bitbucket Server 6.x
- Jenkins
- Sonarqube
- Checkstyle
- Spring 

---

### 시나리오 

파이프라인을 구축하기 앞어서, 제가 구성하고자 하는 시나리오를 설명드리겠습니다.  

1. 우리팀에서 사용중인 Git-flow에 따라 ([참조](https://taes-k.github.io/2020/01/07/clean-git-flow/)),  `feature` 브랜치에서 작업한 커밋들을 `work` 브랜치로 `PR`을 요청할때 

2. 해당 소스의 `build`가 성공하는지 확인하고, 만약 실패한다면 `PR Merge`를 막을 수 있어야 한다.

3. `build`가 성공한다면 팀 코드 컨벤션 체크를 하고 `PR`에 자동으로 `Comment`를 달아주어 동료 리뷰어들이 컨벤션을 리뷰하는일이 없도록 한다.

4. 정적분석툴을 사용하여 나온 `'악취 코드'`들을 `PR`에 자동으로 `Comment`를 달아주어 별도의 사이트로 이동하지 않고 코드리뷰 페이지에서 바로 확인 할수 있게한다.

---

### Jenkins pipeline

위의 시나리오를 충족시키기 위해 Jenkins에서 CI를 구성해보도록 하겠습니다.

#### 1\.Jenkins Job 설정

새로운 CI를 구성하기 위해 젠킨스에서 작업을 처리하는 단위인 Job을 생성해줍니다.  
작업하시는분의 스타일에 따라서 여러가지 방법이 있지만, 이 포스팅에서는 `파이프라인`을 사용하여 Jenkins CI를 구축해 보도록하겠습니다. (Jenkins 설치과정 및 초기 세팅과정은 생략합니다.)

![1]({{ site.images | relative_url }}/posts/2020-06-22-jenkins-auto-pr-comment/1.png)  

시나리오상 Bitbucket에서 `PR`을 요청할때 CI가 동작해야 하기 때문에, 가장먼저 `Build Trigger`를 설정해 주셔야 합니다. 이 또한 다양한 방법으로 구성 할 수 있으나 가장 범용적으로 사용하기 좋은 `Generic Webhook Trigger`를 이용하여 설정해 보겠습니다.  

![2]({{ site.images | relative_url }}/posts/2020-06-22-jenkins-auto-pr-comment/2.png)  

우선은 다른 설정 필요없이 이 Job에 연결 할 수 있는 token만 설정을 해주도록 하겠습니다.


#### 2\.Bitbucket Hook 설정

Jenkins에서 Trigger token을 설정해주었기때문에 이제 bitbucket에서 webhook을 연결 해 줄 수 있습니다.

![3]({{ site.images | relative_url }}/posts/2020-06-22-jenkins-auto-pr-comment/3.png)  

pull-request가 OPEN 되거나 변경될때 CI쪽에서 캐치를 할 수 있도록 hook을 설정해줍니다.  
`URL`은 Jenkins에서 설정한 Project token으로 연결하여줍니다. 

#### 3\.Jenkins 파라미터 설정

Jenkins에서 구성한 `Generic Webhook`에서는 `Post content parameters`를 설정 할 수 있습니다. Bitbucket에서 hook을 통해 들어오는 데이터를 파싱하여 CI에서 사용할 parameter로 사용 할 수있는데 우리에게 필요한 파라미터들을 다음과 같습니다.  

- BRANCH
- PROJECT_KEY
- REPO_SLUG
- PULL_REQUEST_ID

위 파라미터를 얻기위해서는 Bitbucket hook에서 제공하는 payload를 매칭시켜주어야 합니다. 
([Bitbucket hook payload](https://confluence.atlassian.com/bitbucketserver063/event-payload-972354401.html?utm_campaign=in-app-help&utm_medium=in-app-help&locale=ko_KR%2Cko&utm_source=stash#Eventpayload-pullrequest))


![4]({{ site.images | relative_url }}/posts/2020-06-22-jenkins-auto-pr-comment/4.png)  

![5]({{ site.images | relative_url }}/posts/2020-06-22-jenkins-auto-pr-comment/5.png)  

#### 4\.Jenkins 파이프라인 설정

이제 실제 CI가 구동될 Pipeline script를 작성해보도록 하겠습니다.  
파이프라인은 시나리오를 만족시키기 위해 다음과 같은 stage들로 나누어 구성 할 예정입니다. 

> clone -> build -> 컨벤션 검사 -> 정적분석 -> write comment



- `stage('clone')`  

가장먼저, 저장소로 부터 코드를 불러와야합니다.  
이때 위에서 설정한 `BRANCH` 파라미터를 사용하셔서 `PR Branch`를 지정해서 clone 해야 하는 점을 주의하셔야 합니다.

```
stage ('clone') {
    git branch: '$BRANCH', credentialsId: '{jenkins-crendntial}', url: '{bitbucket-url}'
}
```

- `stage('build')`

다음 스테이지는 PR 코드를 `빌드`해보고, 빌드에 실패시 merge를 `block`해 줄수 있어야 합니다. 쉘 스크립트를 구성하기전에 수도코드로 나타내보자면 다음과 같습니다.

```
try
{
    build;
    comment("[BUILD] SUCCESS"); 
}
catch
{
    comment("[BUILD] FAILED");
    make bloking task("build failed");
    throw exception;
}
```

빌드에대한 결과는 간단하게 comment를 달아주는 방식으로 처리하였습니다. 다만, 빌드가 실패할시 merge를 막아줘야 하는데 어느정도 유연함을 주기 위하여 빌드 실패시 comment와 함께 `task`를 생성하여 blocking 하도록 설정해 줍니다.  

(Bitbucket에서 [Merge checks - No incomplete tasks] 를 enable로 설정하셔야 합니다.)

```
stage ('build'){
    withCredentials([usernameColonPassword(credentialsId: 'jenkins-credential', variable: 'USER')]) {
        sh '''
        {
            ./gradlew clean build -x test \
            && curl -X POST -u "$USER" {bitbucket-url}/rest/api/1.0/projects/$PROJECT_KEY/repos/$REPO_SLUG/pull-requests/$PULL_REQUEST_ID/comments -d '{"text":"[BUILD] SUCCESS"}' -H 'Content-Type: application/json';
        }||\
        {
            COMMENT_ID=$(curl -X POST -u "$USER" {bitbucket-url}/rest/api/1.0/projects/$PROJECT_KEY/repos/$REPO_SLUG/pull-requests/$PULL_REQUEST_ID/comments -d '{"text":"[BUILD] FAILED"}' -H 'Content-Type: application/json' | python3 -c 'import sys, json; print(json.load(sys.stdin)["id"])'); 
            curl -X POST -u "$USER" -d '{"anchor":{"id":'${COMMENT_ID}',"type":"COMMENT"},"text":"BUILD FAILED"}' {bitbucket-url}/rest/api/1.0/tasks -H 'Content-Type: application/json';
            exit 1;
        }
        '''
    }
} 
```

현재 저희가 사용하는 BitbucketServer 버전에서는 `blocking-comment`api를 제공하지 않아 최신버전의 bitbucket을 사용하신다면 `task` api 대신 `blocking-comment`api를 사용하시는것을 추천드립니다.

- `stage('checkstyle')`

다음 스테이지는 `컨벤션 검사`입니다. 컨벤션 검사를위해 `checkstyle`을 사용했습니다.  
`checkstyle`은 컨벤션 검사 이외에도 버그검사가 가능하지만, 버그 및 code-smell 검사를 위해 별도의 정적분석 툴을 사용할 예정이기때문에 이번 스테이지에서는 `컨벤션 검사`만을 진행합니다.  

`checkstyle`을 사용하기위해서는 미리 팀에서 사용하는 컨벤션에 따라서 `checkstyle rule`을 지정해 줘야 합니다. 저희팀에서 사용하는 기번 컨벤션 룰은 다음과 같습니다.  

> - Indent : 4 space
> - Line length : 120
> - Brace style : new-line
> - If statemnet : new-line, force-brace in multi-line
> - For statemnet : forced-brace
> - try statement : new-line

checkstyle 사용을 위해 Spring project에 환경설정이 필요합니다.

```
// build.gradle

plugins {
    id 'checkstyle'
}
checkstyle {
    ignoreFailures true
    toolVersion '8.33'
    configFile file("config/checkstyle/checkstyle.xml")
}
checkstyleMain {
    source ='src/main/java'
}
checkstyleTest {
    source ='src/test/java'
}
```

```xml
// config/checkstyle/checkstyl.xml

<?xml version="1.0"?>
<!DOCTYPE module PUBLIC
    "-//Checkstyle//DTD Checkstyle Configuration 1.3//EN"
    "https://checkstyle.org/dtds/configuration_1_3.dtd">

<module name="Checker">
    <property name="severity" value="info"/>
    <property name="fileExtensions" value="java, properties, xml"/>
    <module name="SuppressionFilter">
        <property name="file" value="config/checkstyle/suppressions.xml"/>
        <property name="optional" value="true"/>
    </module>
    <module name="NewlineAtEndOfFile"/>
    <module name="Translation"/>
    <module name="FileLength"/>
    <module name="LineLength">
        <property name="fileExtensions" value="java"/>
        <property name="max" value="120"/>
    </module>
    <module name="FileTabCharacter"/>

    <module name="TreeWalker">
        <module name="EmptyForIteratorPad"/>
        <module name="GenericWhitespace"/>
        <module name="MethodParamPad"/>
        <module name="NoWhitespaceAfter"/>
        <module name="NoWhitespaceBefore"/>
        <module name="OperatorWrap"/>
        <module name="ParenPad"/>
        <module name="TypecastParenPad"/>
        <module name="WhitespaceAfter"/>
        <module name="AvoidNestedBlocks"/>
        <module name="EmptyBlock"/>
        <module name="LeftCurly">
            <property name="option" value="nl"/>
        </module>
        <module name="RightCurly">
            <property name="option" value="alone_or_singleline"/>
        </module>
        <module name="ArrayTypeStyle"/>
        <module name="CommentsIndentation"/>
        <module name="Indentation"/>
        <module name="UpperEll"/>
    </module>
</module>
```
check style 모든 rule은 다음 링크에서 확인하실수 있습니다. (https://checkstyle.org/checks.html)  

만약 컨벤션 체크를 제외하고자 하는 별도 파일이 있다면 suppression을 지정해 줄 수도 있습니다.  

```xml
<?xml version="1.0"?>
<!DOCTYPE suppressions PUBLIC
    "-//Checkstyle//DTD SuppressionFilter Configuration 1.2//EN"
    "https://checkstyle.org/dtds/suppressions_1_2.dtd">

<suppressions>
   <suppress checks="OperatorWrap" files=".*IT\.java" />
</suppressions>
```

위에서 설정한 `checkstyle`은 `gradle check` 명령을 통해 실행 시킬수 있습니다.  
이제 준비작업은 끝났으니 파이프라인에 `checkstyle` 스테이지를 추가해보도록 하겠습니다. 수도코드 로직은 다음과 같습니다. 

```
try
{
    check;
    comment("[CHECKSTYLE] SUCCESS"); 
}
catch
{
    comment("[CHECKSTYLE] FAILED");
    throw exception;
}
```

```
    stage ('checkstyle') {
        sh '''
        {
        ./gradlew clean -x test check \
        && curl -X POST -u "$USER" {bitbucket-url}/rest/api/1.0/projects/$PROJECT_KEY/repos/$REPO_SLUG/pull-requests/$PULL_REQUEST_ID/comments -d '{"text":"[CHECKSTYLE] SUCCESS"}' -H 'Content-Type: application/json' 
        }||\
        {
        curl -X POST -u "$USER" {bitbucket-url}/rest/api/1.0/projects/$PROJECT_KEY/repos/$REPO_SLUG/pull-requests/$PULL_REQUEST_ID/comments -d '{"text":"[CHECKSTYLE] FAILED"}' -H 'Content-Type: application/json'; 
        exit 1;
        }
        '''
    }
```
`checkstyle`에서 수행 결과로 나온 이슈들은 '/build/checkstyle' 폴더 내부에 .xml 파일로 남게됩니다.  
해당 스테이지에서 나온 xml파일을 파싱하여 `PR`에 comment를 남길 예정입니다.  

- `stage('sonar')`

이제 정적분석을 통해 버그와 `악취코드`들을 찾을 stage 입니다. 오픈소스인 sonarqube가 구성되어있다는 가정하에, sonarqube 검사를 수행 한 후 결과물을 json 파일로 가지고오는것 까지 이번 스테이지에서 작업을 수행해보도록 하겠습니다.  


```
try
{
    sonar;
    comment("[CHECKSTYLE] SUCCESS"); 
    getSonarResultJsonFile();
}
catch
{
    sonar("[CHECKSTYLE] FAILED");
    throw exception;
}
```
위의 `checkstyle`에서 수행한 내용과 별 다른내용은 없으나 sonarqube는 결과데이터를 파일로 남기지 않기때문에 `sonarqube` api를 활용하여 파일을 가지고오는 별도의 작업이 필요합니다. (sonarqube 옛날버전에서는 json file으로 저장하는것이 가능했으나 최신버전에서는 제공하지 않음)

```
stage('sonar'){
    withCredentials([usernameColonPassword(credentialsId: 'jenkins-credential', variable: 'USER')]) {
        sh '''
        {
        ./gradlew sonarqube -x test --stacktrace --info -Dsonar.projectKey={sonar-project-key} -Dsonar.host.url={sonar-url} -Dsonar.login={soanr-login-key} \
        && curl -X POST -u "$USER" {bitbucket-url}/rest/api/1.0/projects/$PROJECT_KEY/repos/$REPO_SLUG/pull-requests/$PULL_REQUEST_ID/comments -d '{"text":"[SONAR] SUCCESS"}' -H 'Content-Type: application/json'
        }||\
        {
        curl -X POST -u "$USER" {bitbucket-url}/rest/api/1.0/projects/$PROJECT_KEY/repos/$REPO_SLUG/pull-requests/$PULL_REQUEST_ID/comments -d '{"text":"[SONAR] FAILED"}' -H 'Content-Type: application/json'; 
        exit 1;
        }
        '''
    }
}
```

한가지 주의하셔야 할 점은 `gradle` 수행시 `clean`을 수행하게되면 이전 스테이지에서 작업한 `checkstyle`파일이 삭제될수 있으므로 고려해주셔야 합니다.

- `stage('pr-comment')`

자, 이제 파이프라인의 마지막 단계인 정적분석 결과를 파싱하여 `PR`에 `코드리뷰`를 남길 차례입니다.  
이번 스테이지는 사실 파싱과정을 직접 쉘스크립트를 구성하는것이 아닌 Jenkins에서 제공하는 `ViolationsToBitbucketServer` Plugin을 사용하면 간단하게 처리 하실수 있습니다.  

```
stage('comment bitbucket') {
    ViolationsToBitbucketServer([
        bitbucketServerUrl: '{bitbucket-url}',
        commentOnlyChangedContent: true,
        commentOnlyChangedContentContext: 5,
        commentOnlyChangedFiles: true,
        createCommentWithAllSingleFileComments: false,
        createSingleFileComments: true,
        maxNumberOfViolations: 99999,
        keepOldComments: true,
        projectKey: '$PROJECT_KEY', // Use environment variable here
        pullRequestId: '$PULL_REQUEST_ID', // Use environment variable here
        repoSlug: '$REPO_SLUG', // Use environment variable here
        credentialsId: 'ci_searchop',
        commentTemplate: 
            """
            =====================================================

            [ {{violation.reporter}} ]
            [ {{#violation.rule}}{{violation.severity}} - {{violation.rule}}{{/violation.rule}} ]

            =====================================================

            {{violation.message}}

            =====================================================
            """,
        violationConfigs: [
            // Many more formats available, check https://github.com/tomasbjerre/violations-lib
            [parser: 'SONAR', pattern: '.*\\.json\$', reporter: 'Sonar'],
            [parser: 'CHECKSTYLE', pattern: '.*/checkstyle/.*\\.xml\$', reporter: 'Checkstyle']
        ]
    ])
}
```

해당 설정을 통해서 `Sonarqube`, `checkstyle`에서 나온 결과데이터를 파싱하여 `PR`의 코드를 찾아 comment 를 달아 줄 수 있습니다. `violationConfigs`를 보시면 알겠지만, 이번 포스팅에서 사용한 `sonar`,`checkstyle`이외에도 다양한 툴을 사용한 결과를 comment에 반영 하실수 있습니다.  

다음은 최종 파이프라인 쉘스크립트를 공유드립니다.  

```
node {
    withCredentials([usernameColonPassword(credentialsId: '{jenkins-credential}', variable: 'USER')]) {
  
        stage ('clone') {
            git branch: '$BRANCH', credentialsId: 'ci_searchop', url: '{bitbucket-url}/scm/searchbackend/wsin-catalog-cms.git' // git clone
        }
    
        stage ('build'){
            sh '''
            {
            ./gradlew clean build -x test \
            && curl -X POST -u "$USER" {bitbucket-url}/rest/api/1.0/projects/$PROJECT_KEY/repos/$REPO_SLUG/pull-requests/$PULL_REQUEST_ID/comments -d '{"text":"[BUILD] SUCCESS"}' -H 'Content-Type: application/json';
            }||\
            {
            COMMENT_ID=$(curl -X POST -u "$USER" {bitbucket-url}/rest/api/1.0/projects/$PROJECT_KEY/repos/$REPO_SLUG/pull-requests/$PULL_REQUEST_ID/comments -d '{"text":"[BUILD] FAILED"}' -H 'Content-Type: application/json' | python3 -c 'import sys, json; print(json.load(sys.stdin)["id"])'); 
            curl -X POST -u "$USER" -d '{"anchor":{"id":'${COMMENT_ID}',"type":"COMMENT"},"text":"BUILD FAILED"}' {bitbucket-url}/rest/api/1.0/tasks -H 'Content-Type: application/json';
            exit 1;
            }
            '''
        } 
    
        stage ('checkstyle') {
            sh '''
            {
            ./gradlew clean -x test check \
            && curl -X POST -u "$USER" {bitbucket-url}/rest/api/1.0/projects/$PROJECT_KEY/repos/$REPO_SLUG/pull-requests/$PULL_REQUEST_ID/comments -d '{"text":"[CHECKSTYLE] SUCCESS"}' -H 'Content-Type: application/json' 
            }||\
            {
            curl -X POST -u "$USER" {bitbucket-url}/rest/api/1.0/projects/$PROJECT_KEY/repos/$REPO_SLUG/pull-requests/$PULL_REQUEST_ID/comments -d '{"text":"[CHECKSTYLE] FAILED"}' -H 'Content-Type: application/json'; 
            exit 1;
            }
            '''
        }
        
        stage('sonar'){
            sh '''
            {
            ./gradlew sonarqube -x test --stacktrace --info -Dsonar.projectKey={project-key} -Dsonar.host.url={sonar-url} -Dsonar.login={sonar-key} \
            && curl -X POST -u "$USER" {bitbucket-url}/rest/api/1.0/projects/$PROJECT_KEY/repos/$REPO_SLUG/pull-requests/$PULL_REQUEST_ID/comments -d '{"text":"[SONAR] SUCCESS"}' -H 'Content-Type: application/json'
            }||\
            {
            curl -X POST -u "$USER" {bitbucket-url}/rest/api/1.0/projects/$PROJECT_KEY/repos/$REPO_SLUG/pull-requests/$PULL_REQUEST_ID/comments -d '{"text":"[SONAR] FAILED"}' -H 'Content-Type: application/json'; 
            exit 1;
            }
            '''
        }
    
        stage('comment bitbucket') {
            ViolationsToBitbucketServer([
                bitbucketServerUrl: '{bitbucket-url}/',
                commentOnlyChangedContent: true,
                commentOnlyChangedContentContext: 5,
                commentOnlyChangedFiles: true,
                createCommentWithAllSingleFileComments: false,
                createSingleFileComments: true,
                maxNumberOfViolations: 99999,
                keepOldComments: true,
                projectKey: '$PROJECT_KEY', // Use environment variable here
                pullRequestId: '$PULL_REQUEST_ID', // Use environment variable here
                repoSlug: '$REPO_SLUG', // Use environment variable here
                credentialsId: 'ci_searchop',
                commentTemplate: 
                    """
                    =====================================================

                    [ {{violation.reporter}} ]
                    [ {{#violation.rule}}{{violation.severity}} - {{violation.rule}}{{/violation.rule}} ]

                    =====================================================

                    {{violation.message}}

                    =====================================================
                    """,
                violationConfigs: [
                    // Many more formats available, check https://github.com/tomasbjerre/violations-lib
                    [parser: 'SONAR', pattern: '.*\\.json\$', reporter: 'Sonar'],
                    [parser: 'CHECKSTYLE', pattern: '.*/checkstyle/.*\\.xml\$', reporter: 'Checkstyle']
                ]
            ])
        }
    }
}
```

---

### 결과

![6]({{ site.images | relative_url }}/posts/2020-06-22-jenkins-auto-pr-comment/6.png)   
![7]({{ site.images | relative_url }}/posts/2020-06-22-jenkins-auto-pr-comment/7.png)   
![8]({{ site.images | relative_url }}/posts/2020-06-22-jenkins-auto-pr-comment/8.png)   

다행히도 생각했던 시나리오대로 체크를 진행하고 comment를 작성하는것을 확인 할 수 있습니다.  

사실, 이러한 CI를 구성한 이유는 어떻게하면 코드리뷰를 더 잘 할수 있을까(https://taes-k.github.io/2020/05/29/code-review/)에 대한 하나의 방안으로, 컨벤션 및 일반적인 오류체크들을 정적분석 툴에 맡김으로써 리뷰어들의 리뷰 피로도를 줄이고자 하였습니다. 

아직 팀내에서도 따끈따근한 CI라 효용성에 대한 검증은 부족하나, 적어도 기존에 확인하기 불편했던 정적분석툴을 편하게 볼 수 있게되었다는 큰 의미를 가질수 있게 되었다고 자축하고 싶습니다.