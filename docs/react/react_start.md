---
layout: default
comments: true
title: React 시작하기
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

### 리액트(React.js)란?
페이스북에서 만들어진 오픈소스 자바스크립트 라이브러리로써, interactive UI 구현에 최적화되어있다.  

특징
- UI Component
```c
class SearchBar extends React.Component{
    render(){
        <input><button>
    }
}

class App extends React.Component{
    render(){
        <div>
            <SearchBar />
        </div>
    }
}

ReactDOM.render(<App />, document.querySelector('#root'));
```
위 코드에서 알수있듯이 마치 backend 개발에서 사용되는것처럼 컴포넌트화 시켜 UI를 구성시킬수 있다.

- Virtual DOM
일반적으로 Javascript로 DOM을 조작하게되면 실제 DOM을 변경하여 웹을 동적으로 구성하게된다. 이때  전체 DOM이 갱신되게 되는데, 반복적으로 DOM 이 변경되거나 많은양의 정보가 바뀌는경우에는 비효율적이고 많은 리소스 저하가 발생할 수 있다.  

이러한 문제점을 보완하기 위해 React 실제 DOM이아닌 가상의 DOM을 만들어 변경사항이 있을때 전체가 아닌 해당 부분만 변경하여 반영시키는 방식으로 작업을 수행하여 효율성과 속도를 높일 수 있다.

- 단방향 데이터 바인딩
일반적으로 View의 변경이 스스로 변경시키것과 달리, 리액트에서는 컴포넌트마다의 별도의 state를 가져 View 를 업데이트 하게된다. View의 변경이 이루어질때, state의 변경을 일이키는 callback을 호출하여 state를 업데이트 시키고, 업데이트된 state는  View를 변경시키는 순환적 단방향 데이터 바인딩이 일어난다.

### 리액트 설치
먼저, 리액트는 node.js기반으로 만들어진 라이브러리로, 
1. Node.js 설치
리액트는 node.js 기반으로 만들어진 라이브러리로  Node.js의 설치가 필요하다.
1-1) nvm 설치
```c
curl https://raw.githubusercontent.com/creationix/nvm/v0.30.2/install.sh | bash
```
1-2) Node.js 설치
```c
nvm install stable
```
1-3) Node.js 설치버전 확인
```c
node -v
```


2. Yarn 설치
Yarn은 조금더 개선된 성능의 패키지 매니저 도구이다.

2-1) Yarn 설치
```c
brew install yarn
```

2-2) Yarn 설치버전 확인
```c
yarn --version
```

2-3) Yarn global 설정
```c
echo 'export PATH="$(yarn global bin):$PATH"' >> ~/.bash_profile
source /.bash_profile
```


3. create-react-app 설치
```c
yarn global add create-react-app
```

### React app 생성
리액트 프로젝트 디렉토리로 이동하여 간단하게 생성 할 수 있다.
```c
create-react-app frontend
```
