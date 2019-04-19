---
layout: default
comments: true
title: React.js 동작
parent: react
date: 2019.04.19
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### 실행환경
- MacOS Mojave 10.14.3
- Node 10.15.3
- Yarn 1.15.2

### 리액트 구조
지난 포스팅에서 create-react-app을 통해 리액트 프로젝트를 만들었다.  
프로젝트를 살펴보면 다음과 같다.

![1]({{ site.images | relative_url }}/react_start2/1.jpg)

 `App.js` 
![2]({{ site.images | relative_url }}/react_start2/2.jpg)
위 내용을 보면, App 컴포넌트를 정의하여 JSX(HTML 같은 코드)를 return 하여 렌더링해주는것을 확인 할 수 있다.  
여기서, css나 image 파일또한 import하여 렌더링이 가능하다.  

`index.js`
![3]({{ site.images | relative_url }}/react_start2/3.jpg)
index에서는 위에서 살펴보았던  `App.js` 컴포넌트를 import하여 Dom에 넣어주는것을 확인 할 수 있다.


