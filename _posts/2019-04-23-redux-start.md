---
layout: post
comments: true
title: React+Redux
tags: [react]
---

### React 프로젝트에 Redux 적용하기
기존 리액트 보일러 플레이트에 뉴스를 나열해주는 페이지 구현으로 리덕스를 추가 적용하는 예제를 진행해보도록 하겠다.  

*** 프로젝트에 redux 설치 ***
```c
npm install --save redux react-redux
```

*** 뉴스 액션,리듀서 생성 ***
`/src/store/module/News.js`
```c
//Type
const ADD_LIKE = "ADD_LIKE";
const DEL_LIKE = "DEL_LIKE";

//Action
function addLike(data) {
    return {
        type: ADD_LIKE,
        like: 1
    }
}
function delLike(data) {
    return {
        type: ADD_LIKE,
        like: -1
    }
}

//State
const initNewsState = {
    like:0
}

// Reducer
function reducer(state = initNewsState, action) {
    switch (action.type) {
    case ADD_LIKE:
        console.log(state)
        return Object.assign({}, state, {
            like: state.like + 1;
        });
    case DEL_LIKE:
        return Object.assign({}, state, {
            like: state.like - 1;
        });
    default:
        return state;
    }
}

// Export
const actionCreators = {
    addLike,
    delLike
};
export { actionCreators };  // action export
export default reducer;     // reducer export
```

*** 통합 리듀서 모듈 생성 ***
`/src/store/module/index.js`
```c
import { combineReducers } from 'redux';
import news from './News';

export default combineReducers({
    news
});
```

*** 뉴스 컴포넌트 생성 ***
`/src/componenet/NewsComponenet.js`
```c
import React, { Component } from 'react';
import { connect } from 'react-redux';
import news from '../store/modules/News';
import { actionCreators } from '../store/modules/News';

class NewsComponents extends Component{
    constructor(props) {
    super(props);
    this.state = {
    like: 1
    }
}
render() {
    return(
        <div className="News">
        <div className="Title">
            NEWS 제목
        </div>
        <div className="Contents">
            NEWS 컨텐츠
        </div>
        <div>Like :{ this.props.like }</div>
        <button onClick={this.props.likeIncrease}>Like</button>
        </div>
        );
    }
}

const mapStateToProps = ({ news }) => ({
    like: news.like
});

let mapDispatchToProps = (dispatch) => {
    return {
        likeIncrease: () => dispatch(actionCreators.addLike())
    }
}

export default connect(mapStateToProps,mapDispatchToProps)(NewsComponents);
```

connect를 통해 store와의 연동 설정을  할 수 있다.  다음 두 api를 통해  state 값을 가져오거나, 액션을 실행 시킬수 있다.  
`mapStateToProps`  :   state값을  props로 가져온다.
`mapDispatchToProps` :  action을 연결해준다.


*** App.js ***
`/src/App.js`
```c
import React from "react";
import '../css/App.css';
import News from './components/NewsComponents'

const App = () => {
    return (
        <div class="App">
            <p>Hello, world!</p>
            <News/>
        </div>
    );
};

export default App;
```

*** index.js ***
`/src/index.js`
```c
import App from "./App.js";
import React from "react";
import ReactDom from "react-dom";
import { createStore } from 'redux';
import rootReducer from './store/modules';
import { Provider } from 'react-redux';

const store = createStore(rootReducer);
console.log(store.getState());

ReactDom.render( <Provider store={store}>
                    <App />
                </Provider>, document.getElementById("root"));
```
index에서 store를 만들어 사용해준다.  



![1]({{ site.images | relative_url }}/posts/2019-04-23-redux-start/1.png)    
![2]({{ site.images | relative_url }}/posts/2019-04-23-redux-start/2.png)    

최종 구조는 다음과 같다.  
![3]({{ site.images | relative_url }}/posts/2019-04-23-redux-start/3.png)    
