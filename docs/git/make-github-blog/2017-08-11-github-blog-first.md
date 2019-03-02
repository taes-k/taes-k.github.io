---
layout: default
title: github로 개인 홈페이지 만들기-Jekyll
parent: git
---



아무래도 정리할곳이 필요할것 같아 장소를 이리저리 찾아보다가  github에 직접 만들어두고 개발 기술블로그를 만들어보았다. 그 과정을 첫 포스트로 작성한다 

(티스토리, 브런치 등등도 고려하였으나 가입하기부터 내맘대로 되지 않아 기분이 상했다.)

<h3>0. 환경</h3>
----

- 맥 OS X 10.12.1

<h3>1. 설치</h3>
-----
<h5>1-1 jekyll 설치</h5>
루비 명령어를 이용해 jekyll을 다운로드한다.

```
[sudo] gem install jekyll
```



\* OS X 에 기본적으로 설치되어있는 루비 2.0.0 버전으로는 최신버전 jekyll을 다운로드 받을수 없으니 업데이트가 필요하다



```
brew update ruby-build

brew install rbenv ruby-build
rbenv install -l (리스트에서 확인후 버전별 다운로드)
rbenv install 2.4.0
rbenv global 2.4.0 (다운로드 한 버전 사용 설정)
brent rehash 

rbenv versions (설치된 버전 확인)

ruby -v (루비 버전 확인)
```





\* 루비 버전이 바로 업데이트가 되지 않을수 있다. 그럴땐 재부팅

<h5>1-2 jekyll deploy</h5>
다운로드 받은 jekyll을 통해 기본 블로그를 배포 해보자.

```
jekyll new {github-user-name}.github.com
```

\* bundler가 설치되어있지 않다면 에러가 날수 있음

```
gem install bundler
```

잘 마쳐졌다면 /Users/{name}/ 아래에 {github-user-name}.github.com 폴더가 만들어졌을 것이다.

```
cd {github-user-name}.github.com
jekyll serve --watch
```


위의 명령어로 기본 블로그 페이지를 로컬 에서 구동해 볼수 있다.
http://localhost:4000
￼

![1]({{ site.images | relative_url }}/2017-08-11-github-blog-first_1.png)



<h3>2. 연동</h3>
-----

이제 만들어진 폴더를 깃허브 저장소와 연동해줄 차례이다.

<h5>2-1 GitHub </h5>
GitHub 계정이 있다는 전제하에 진행하겠다.
깃허브에 접속하여 {github-user-name}.github.io 이름으로 repository를 생성합니다.
￼

![2]({{ site.images | relative_url }}/2017-08-11-github-blog-first_2.png)

친절한 설명이 나와있는데

```
git init
git remote add origin {저장소 주소}
git add .
git commit -m “블로그 시작”
git push -u origin master

{username 입력 / password 입력}
```

위 명령어들로 간단하게 깃허브 저장소에 push를 해줄수 있습니다.

<h5>2-2 확인</h5>
이제 저장소에 jekyll 기본 블로그가 만들어졌으니 접속해보자
2-1 에서 만든 저장소이름 https://{github-user-name}.github.io  으로 접속해보면
￼![3]({{ site.images | relative_url }}/2017-08-11-github-blog-first_3.png)
로컬에서 확인했던것과 같은 블로그가 생성되었다 !

<br>
<h3>3. 변경</h3>
----

<h5>3-1 기본 git 커밋/푸쉬</h5>
Jekyll 에는 많은분들이 올려주신 많은 테마들이 있어 적용하면 예쁜 블로그를 바로 사용할수 있다. 하지만 설정이나 수정을 해야할 필요가 있기때문에 파일 변경후 저장소에 변경하는 방법을 알아두어야한다.

현재 기본사이트를 보면

title : Your awesome title
email : your-email@example.com

등등으로 default 세팅이 되어있는것을 확인할 수 있다.

다시

```
cd ~/{github-user-name}.github.com
```

 폴더로 가보면 
\_config.yml 파일을 확인할수 있다.
이파일을 수정하기 위해 열어보면 ( vi \_config.yml)

```
title: ~
email: ~
description: ~
```

 등등 수정할수 있게끔 되어있음을 확인할수 있을것이다.

모두 수정후 처음 저장소에 올릴때와 같이

```
git add .
git commit -m “설정변경”
git push orgin master
```

위의 명령어를 진행하면 수정사항들이 적용되어 저장소에 저장되고 홈페이지도 변경된것을 확인 가능할것이다.


![4]({{ site.images | relative_url }}/2017-08-11-github-blog-first_4.png)￼

<h5>3-2 테마변경</h5>

http://jekyllthemes.org
지킬 테마 모음 사이트에서 원하는 테마를 골라 다운로드 해준다.
다운로드 받으면 파일들 전체를 내가 만들어둔 지킬 폴더에 넣어준후 위의 과정,
저장소에 올리기를 실행하면 이뻐진 내 사이트를 확인할수 있다.

￼![5]({{ site.images | relative_url }}/2017-08-11-github-blog-first_5.png)

나머지 페이지 설정사항은 3-1처럼 마찬가지로 수정하여 나만의 블로그를 만들어 주면  끝!
<br>
<h3>4. 포스트 올리기</h3>

지킬 기본 디렉토리 구조를 살펴보자.

- _config.yml : 환경설정
- _includes : 헤더/푸터 등의 빌더가 들어있는 폴더
- _layouts : 테마에 사용되는 폴더로 앞으로 올릴 포스트들의 레이아웃 설정 파일 폴더
- _post : 포스트를 올릴 폴더 (\*)
- _site : 컴파일 되면 옮겨지는 폴더

위에서 살펴본것처럼 포스팅을하려면 _post 폴더에 넣어주기만하면 된다!

<h5>4-1 마크다운</h5>

포스팅하는 파일 형식은 마크다운 형식으로 올리면 자동 적용된다.

몇몇가지 주의사항과 html 문법을 알면 간단하게 진행할수 있다.

- 문서제목을 YYYY-MM-DD-[문서이름].[포맷] 형식으로 올려야한다. (2017-08-11-first-blog.md)

\* 자세한 마크다운 문법은 구글에 검색하면 잘 나온다. ( 깃허브 마크다운 사용법 참조 :  https://gist.github.com/ihoneymon/652be052a0727ad59601)



<h5>4-2 마크다운 에디터 </h5>

마크다운 파일을 그냥 vi 로 만들거나 해도 무방하지만 여간 귀찮은 일이 아닐수 없다.

마크다운 형식을 편하게 사용할수 있는 몇몇가지 에디터를 찾다가 현재 이 글을 쓰고 있는 Typora 를 발견하여 만족하며 사용중이다. 가장 마음에 드는점은 길어져도 렉이 없으며 (아톰 XXX) 개발자를 위한 매우 간편한 코드펜스 기능을 지원한다.

```javascript
alert("바로 이것!");
```

https://www.typora.io



참고 블로그 

[놀부님 블로그](https://nolboo.kim/blog/2013/10/15/free-blog-with-github-jekyll/)

