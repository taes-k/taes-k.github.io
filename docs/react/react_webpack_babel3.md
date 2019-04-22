---
layout: default
comments: true
title: React+Webpack+Babel(3)
parent: react
date: 2019.04.22
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### 실행환경
- MacOS Mojave 10.14.3
- Node 10.15.3
- Webpack 4.30.0
- Babel 7.4.3
- React 16.8.6

### React
이전 포스팅에서 진행했던 웹팩, 바벨에 이어 리액트를 적용시켜보도록 하겠다. 리액트에대한 설명은 [React 시작하기](https://taes-k.github.io/docs/react/react_start/) 에서 확인 하길 바란다.  

### React 사용 설정
*** 프로젝트에 react 설치하기 ***
```c
npm install --save-dev react react-dom @babel/preset-react
```

*** 프로젝트에 html 플러그인 설치하기 ***
웹팩 패킹시 html 파일도 찾아 패킹할 수 있도록 함
```c
npm install --save-dev html-webpack-plugin html-loader
```

*** 프로젝트에 css 플러그인 설치하기 ***
웹팩 패킹시 css 파일도 찾아 패킹할 수 있도록 함
```c
npm install --save-dev css-loader style-loader
```

*** babel 설정에 react 추가 ***
`.babelrc`  
```c
{
"presets": [
"@babel/preset-env",
"@babel/preset-react"
]
}
```

*** webpack 설정에 html, css 플러그인 추가 ***
리액트의 jsx, html, css를 패킹할수 있도록 설정해 준다.  
`webpack.config.js`  
```c
const HtmlWebPackPlugin = require("html-webpack-plugin");

module.exports = {
    module:{
        rules:[
            {
                test: /\.(js|jsx)$/,
                exclude: /node_modules/,
                loader:"babel-loader"
            },
            {
                test: /\.html$/,
                loader:"html-loader"
            },
            {
                test: /\.css$/,
                use:["style-loader","css-loader"]
            }
        ]
    },
    plugins:[
        new HtmlWebPackPlugin({
            template: "./public/index.html",
            filename: "./index.html"
        })
    ]
};
```

### React 사용하기
`App.css`
```c
.App{
width:100%;
height:200px;
background-color:#d0d0d0;
}
```

`App.js`
```c
import React from "react";
import '../css/App.css';

const App = () => {
    return (
        <div class="App">
        <p>Hello, world!</p>
        </div>
    );
};

export default App;
```  

`index.js`
```c
import App from "./App.js";
import React from "react";
import ReactDom from "react-dom";

ReactDom.render(<App />, document.getElementById("root"));
```

`index.html`
```c
<!DOCTYPE Html>
<html lang='en'>
<head>
    <meta charset="UTF-8">
    <title>react+webpack+babel</title>
</head>
<body>
    <div id="root"></div>
</body>
</html>
```

![1]({{ site.images | relative_url }}/react_webpakc_babel3/1.png)    


이로써 리액트 + 웹팩 + 바벨 환경의 보일러플레이트 구축이 완료되었다.
