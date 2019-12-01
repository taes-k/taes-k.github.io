---
layout: post
comments: true
title: React 기본 API
tags: [react]
---
### 실행환경
- MacOS Mojave 10.14.3
- Node 10.15.3
- Yarn 1.15.2

### 컴포넌트 라이프사이클 API
컴포넌트의 생성/업데이트 시점에 따라 동작할수있는 라이프사이클 API 를 활용할 수 있다.  
기본적으로 사용되는 API들은 다음과 같다.  

`componentWillMount()` - 생성 직전  

`componeneDidMount()` - 생성 직후  
컴포넌트가 화면에 나는 시점에서 호출된다.   
주로 해당 컴포넌트에서 필요한 데이터를 가져오기위해 ajax요청을 하거나 DOM 속성을 읽어올때 사용한다.

`compnentWillRecieveProps(nextProps)` - 업데이트 직전  
컴포넌트가 새로운 props을 받는 시점에서 호출된다.    
주로 변경된 props를 통한 state의 변경 로직을 작성한다.    
이 기능의경우 v16.3이후, 새로운 API  getDerivedStateFromProps()  으로 대체될수있다.

`static getDerivedStateFromProps(nextProps, prevState) ` - 업데이트 직전    
props의 변경에따른 state의 동기화작업시 사용한다.

`componentDidUpdate(prevProps, prevState)` - 업데이트 직후  

`componentWillUnmount()` - 언마운트 직전   

`shouldComponentUpdate(nextProps, nextState)` - 업데이트 직전, 처음  
업데이트를 확인하여 불필요한 업데이트시 render를 막아줄 수 있다.   

`componentDidCatch(error, info)` - 에러발생

### 직접 사용해 보기
`News.js`  
```c
import React, { Component } from 'react';

class News extends Component {
    state = {
        like: 0,
        news:{
        title: '제목',
        contents: '내용',
        }
    }

    likeIncrease = () => {
        this.setState({
            like: this.state.like + 1
        });
    }

    componentDidMount(){
        this.setState({
            news:{
                title: this.props.title,
                contents: this.props.contents
            }
        })
    }

    componentDidUpdate(prevProps, prevState){
        console.log("STATE Change : ",prevState)
    }

    render() {
        return (
            <div className="News">

            <div class="Title">
                {this.state.news.title}
            </div>
            <div class="Contents">
                {this.state.news.contents}
            </div>


            <div>Like :{this.state.like}</div>
            <button onClick={this.likeIncrease}>Like</button>
            </div>
        );
    }
}

export default News;

```

![1]({{ site.images | relative_url }}/react_start3/1.png)  
![2]({{ site.images | relative_url }}/react_start3/2.png)  

다음과같이 `componentDidMount` api를 통해 데이터 바인딩이 가능하고,  `componentDidUpdate` api를 통해 State변화 이벤트를 캐치하여 작업을 할 수 있다.
