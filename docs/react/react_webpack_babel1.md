---
layout: default
comments: true
title: React+Webpack+Babel(1)
parent: react
date: 2019.04.20
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### 실행환경
- MacOS Mojave 10.14.3
- Node 10.15.3
- Webpack 4.30.0
- Webpack-cli 3.3.0 

### Webpack
js 에서 로딩하는 모듈이 많아질수록 모듈간의 의존성이 증가하고 비효율적이게된다. 웹팩은 엔트리를 통해 필요한 모듈을 로딩하고 하나의 파일로 묶어주는 역할을 한다.   

![1]({{ site.images | relative_url }}/webpack/1.png)    


### Webpack 사용하기

*** 설치하기 ***
```c
npm install -g webpack
```

*** 프로젝트에 webpack 설치 ***
```c
npm init
npm install --save-dev webpack
```
webpack을 설치하고나면, `node_module`과 `package-lock.json`이 설치되며 기존 `package.json`에 다음과 같이 dependecy가 추가된다.  

```c
"devDependencies": {
"webpack": "^4.30.0"
}
```

*** 파일 생성 ***
`/src/index.js`를 생성 해준다.
```c
console.log("hello my world");
```

*** webpack 실행 ***
`webpack` 명령어를 통해 webpack 을 실행시키게되면 `/dist/main.js`가 생기며, 하나의 js 결과 파일로 묶어주는것을 확인 할 수 있다.
```c
!function(e){var t={};function r(n){if(t[n])return t[n].exports;var o=t[n]={i:n,l:!1,exports:{}};return e[n].call(o.exports,o,o.exports,r),o.l=!0,o.exports}r.m=e,r.c=t,r.d=function(e,t,n){r.o(e,t)||Object.defineProperty(e,t,{enumerable:!0,get:n})},r.r=function(e){"undefined"!=typeof Symbol&&Symbol.toStringTag&&Object.defineProperty(e,Symbol.toStringTag,{value:"Module"}),Object.defineProperty(e,"__esModule",{value:!0})},r.t=function(e,t){if(1&t&&(e=r(e)),8&t)return e;if(4&t&&"object"==typeof e&&e&&e.__esModule)return e;var n=Object.create(null);if(r.r(n),Object.defineProperty(n,"default",{enumerable:!0,value:e}),2&t&&"string"!=typeof e)for(var o in e)r.d(n,o,function(t){return e[t]}.bind(null,o));return n},r.n=function(e){var t=e&&e.__esModule?function(){return e.default}:function(){return e};return r.d(t,"a",t),t},r.o=function(e,t){return Object.prototype.hasOwnProperty.call(e,t)},r.p="",r(r.s=0)}([function(e,t){console.log("hello my world")}]);
```



