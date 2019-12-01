---
layout: post
comments: true
title: React+Webpack+Babel(2)
tags: [react]
---

### 실행환경
- MacOS Mojave 10.14.3
- Node 10.15.3
- Webpack 4.30.0
- Babel 7.4.3

### Babel
js 컴파일러로써, 진화하고있는 자바스크립트와 브라우저간의 호환을 위해 사용하는 라이브러리이다. ES6, ES7 등의 최신문법들을 지원하지 않는 브라우저들에서도 정상동작 하도록 컴파일 해주는 역할을 한다.

### ES6 문법
- arrow function의 사용  
list.forEach(item => console.log(item.value))  
- class 사용  
- var -> let & const 사용  
- template String 사용  
const msg = `Hello, This is ${name}`

등 기존 ES5문법보다 편리하게 사용할 수 있다.  

### Babel 사용하기

*** 프로젝트에 babel 설치하기 ***
```c
npm install --save-dev @babel/core babel-loader @babel/preset-env
```

webpack을 설치하고나면, `node_module`과 `package-lock.json`이 설치되며 기존 `package.json`에 다음과 같이 dependecy가 추가된다.  

```c
"devDependencies": {
    "@babel/core": "^7.4.3",
    "@babel/preset-env": "^7.4.3",
    "babel-loader": "^8.0.5",
    "webpack": "^4.30.0",
    "webpack-cli": "^3.3.0"
}
```

*** 바벨 설정 ***
`.babelrc` babel 설정 파일을 생성 해준다.
```c
{
    "presets":[
        "@babel/preset-env"
    ]
}
```

*** webpack 설정 ***
`webpack.config.js` webpack 설정 파일 생성 해준다. 이는 패킹하는 과정에서 위에서 설치한  babel 을 사용하여 패킹 할 수 있도록 설정 해 준다.
```c
module.exports = {
    module:{
    rules:[
        {
            test: /\.js$/,
            exclude: /node_modules/,
            use:{
            loader: "babel-loader"
            }
        }
    ]
    }
};
```

### Babel 적용 확인해보기
`index.js`에 ES6 문법 을 작성하고 빌드시켜본다.
```c
//index.js
console.log("hello my world");

const arr = [1, 2, 3, 4 ,5 ];
const itemFunc = () => arr.forEach(item => console.log(item))

itemFunc()
```

```c
npm run build
```  

```c
//main.js
!function(e){var t={};function n(r){if(t[r])return t[r].exports;var o=t[r]={i:r,l:!1,exports:{}};return e[r].call(o.exports,o,o.exports,n),o.l=!0,o.exports}n.m=e,n.c=t,n.d=function(e,t,r){n.o(e,t)||Object.defineProperty(e,t,{enumerable:!0,get:r})},n.r=function(e){"undefined"!=typeof Symbol&&Symbol.toStringTag&&Object.defineProperty(e,Symbol.toStringTag,{value:"Module"}),Object.defineProperty(e,"__esModule",{value:!0})},n.t=function(e,t){if(1&t&&(e=n(e)),8&t)return e;if(4&t&&"object"==typeof e&&e&&e.__esModule)return e;var r=Object.create(null);if(n.r(r),Object.defineProperty(r,"default",{enumerable:!0,value:e}),2&t&&"string"!=typeof e)for(var o in e)n.d(r,o,function(t){return e[t]}.bind(null,o));return r},n.n=function(e){var t=e&&e.__esModule?function(){return e.default}:function(){return e};return n.d(t,"a",t),t},n.o=function(e,t){return Object.prototype.hasOwnProperty.call(e,t)},n.p="",n(n.s=0)}([function(e,t){console.log("hello my world");[1,2,3,4,5].forEach(function(e){return console.log(e)})}]);
```

위와같이 ES6로 작성된 문법이 ES5로 컴파일되어 패킹된것을 확인 할 수 있다.
