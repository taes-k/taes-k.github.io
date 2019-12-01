---
layout: post
comments: true
title: React Custom component 만들기  
tags: [react]
---

### 실행환경
- MacOS Mojave 10.14.3
- Node 10.15.3
- Yarn 1.15.2

### 리액트 구조
지난 포스팅에서 create-react-app을 통해 리액트 프로젝트를 만들었다.  
프로젝트를 살펴보면 다음과 같다.

![1]({{ site.images | relative_url }}/react_start2/1.png)

 `App.js` 
![2]({{ site.images | relative_url }}/react_start2/2.png)
위 내용을 보면, App 컴포넌트를 정의하여 JSX(HTML 같은 코드)를 return 하여 렌더링해주는것을 확인 할 수 있다.  
여기서, css나 image 파일또한 import하여 렌더링이 가능하다.  

`index.js`
![3]({{ site.images | relative_url }}/react_start2/3.png)
index에서는 위에서 살펴보았던  `App.js` 컴포넌트를 import하여 Dom에 넣어주는것을 확인 할 수 있다.

### 리액트 실행
```c
npm start
```
![4]({{ site.images | relative_url }}/react_start2/4.png)

### 컴포넌트 만들어보기
News 컴포넌트를 만들어 index에 넣어보도록 하자.

`News.js`
```c
import React, { Component } from 'react';

class News extends Component {
    state = {
        like: 0
    }

    likeIncrease = () => {
        this.setState({
            like: this.state.like + 1
        });
    }

    static defaultProps = {
        title: '제목',
        contents: '내용',
    }

    render() {
        return (
            <div className="News">

            <div class="Title">
                {this.props.title}
            </div>
            <div class="Contents">
                {this.props.contents}
            </div>


            <div>Like :{this.state.like}</div>
            <button onClick={this.likeIncrease}>Like</button>
            </div>
        );
    }
}

export default News;
```
  
  
`App.js`
```c
import React, { Component } from 'react';
import './App.css';
import News from './News'

class App extends Component {
    render() {
        return (
            <div className="App">
            <News />
            <News title="수출 7조원 달성" contents="2019년 04월 19일 수출이 7조원이 달성했습니다."/>
            <News title="KBO 홈런왕" contents="KIA 나지완 200홈런-LG 김민성 100홈런 달성"/>
            </div>
        );
    }
}

export default App;
```
  
![5]({{ site.images | relative_url }}/react_start2/5.png)

위 예제와 같이 props와 state를 통해 원하는 UI 컴포넌트에 데이터를 넣어 표현해 줄 수있다. 



