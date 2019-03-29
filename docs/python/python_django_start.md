---
layout: default
comments: true
title: Python+Django 웹프로젝트 시작하기
parent: python
date: 2019.03.29
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### 실행환경
- Python 3.7.3
- Django 2.1.7
- PyCharm

### 개요
Python+Django로 처음 웹프로젝트를 시작하려는 사람들을 위해 index 페이지를 띄우기까지 포스팅을 하려한다.   
기본적으로 Python 및 Django가 설치되어있다는 가정하에 진행하도록 하겠다.

### 프로젝트 생성
먼저, Django 프로젝트를 생성해주어야한다.  
터미널에서 저장 폴더로 들어간 후 `django-admin startproject 프로젝트명`으로 프로젝트를 생성해 준다.  
> ![1]({{site.images}}/python_django_start/1.png)

다음으로, 파이참으로 생성해준 프로젝트를 열어주면 이와같은 파일들을 볼 수 있을것이다.
> ![2]({{site.images}}/python_django_start/2.png)

`settings.py` > 장고 설정 파일  
`urls.py` > url 설정 파일  
`manage.py` > 웹 서비스 실행 파일  

우선적으로는 위 3개의 파일들이 주요 파일들이라고 생각하면 된다.

### 앱 생성
위에서 프로젝트를 생성하였고 이제 app을 생성할 차례이다.   
장고에서 app은 특정 기능단위의 웹 어플리케이션을 말한다. 
먼저 파이참에서 command를 사용하기위해 `View > Tool window > Terminal`을 클릭하여 터미널 뷰를 띄워주도록 한다.

이제 커맨드창에 `python manage.py startapp 앱이름` 스크립트를 실행시켜 app을 만들어 주면 디렉토리에 다음과같이 파일들이 생성될 것이다..
> ![3]({{site.images}}/python_django_start/3.png)

### url 설정
먼저, 프로젝트 `urls.py` 를 살펴보자
> ![4]({{site.images}}/python_django_start/4.png)
현재 admin url이 기본적으로 설정되어 있다.  

여기에 새로만들어준 app인 startWeb app의 url 기본패스를 설정해줘야한다. 
> ![5]({{site.images}}/python_django_start/5.png)
다음과 같이 startWeb에 기본 Path 를 설정해준것이다.

다음은 app별로 url을 설정해주어야 한다. 현재 기본으로 생성된 파일중에는 startWeb app내에 `urls.py` 파일이 없을텐데, 새로 만들어주자. 새로 만든 `urls.py`에는 app내의 `views.py`와 연결되어 url들을 지정해주게된다. 
> ![6]({{site.images}}/python_django_start/6.png)
다음과같이 index를 일단 연결해두도록 하자.

### views 설정 & template 설정

위에서 index url을 미리 설정해 두었다. 이제 Vieiw를 띄워줄 차례인데 html 파일을 열어주기위해 먼저 template를 설정하도록 하자.  
template는 프론트엔드 소스가 올라갈 디렉토리라고 생각하면 될것이다.   
root 디렉토리에 `template/index.html`을 만들자.  
> ![7]({{site.images}}/python_django_start/7.png)

이제 다시 `startWeb/views.py`에서 아까 만들어준 `index.html`을 연결시켜보자
> ![8]({{site.images}}/python_django_start/8.png)

이렇게까지 완성시켜두고 `python manage.py runserver` 실행을 통해 프로젝트를 실행켜 웹을 실행 시켜보면 
> ![9]({{site.images}}/python_django_start/9.png)
> ![10]({{site.images}}/python_django_start/10.png)

다음과같이 404 에러가 날것이다. 이것은 처음에 `urls.py`에서 start/ url을 지정해주었기때문이다. 그래서 이제 `127.0.0.1:8000/start`로 접속해보면 index.html을 찾지 못했다는 에러가날것이다.  
이것은 우리가 template을 통해 index.html을 설정해주었지만 django 설정에 template의 위치를 설정해주지 않았기때문이다.  

template의 설정을위해 `setting.py`로 이동해보자. setting 내용을 보면 TEMPLATES가 설정된 부분이 있는데, 기본 DIRS가 비어있는것이 보일것이다. 이위치에 우리가만들어준 template 폴더의 dir을 등록시켜주어야한다. 
> ![11]({{site.images}}/python_django_start/11.png) 
다음과같이 `os.path.join(BASE_DIR, 'template'),` 등록시켜주고 다시 서버를 실행해보자.

> ![12]({{site.images}}/python_django_start/12.png) 

이로써 Django 기본 웹 프로젝트 구성이 완성되었다.

