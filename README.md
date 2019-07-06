# NPM 설치

```javascript
yarn add redux react- redux redux-actions
```

`redux` 리덕스만을 위한 것  

**combineReducers** :  하나의 리듀서로 합쳐 준다  

**createStore** :  데이터 보관 장소  

`react- redux` 리액트에서 리덕스를 사용할 수있게 한다.  

**Provider** :  React App에 Redux를 적용할 때 Provider(공급자)를 사용  

**connect** :  리듀서와 컴포넌트 연결

connect 첫번째 파라미터(mapStateToProps) : 데이터 가져오는 부분  
connect 두번째 파라미터(mapDispatchToProps) : 액션 가져오는 부분   

`redux-actions` 라이브러리를 활용하여 리덕스 모듈 작성을 더욱 손쉽게 하는 방법  

**createAction**  
액션 생성 함수를 간편하게 만들수 있게해주는  redux-actions 의 createAction 이라는 함수  
FSA 규칙을 따르는 액션 객체를 만들어주는데,   
이 FSA 규칙은 읽기 쉽고, 유용하고, 간단한 액션 객체를 만들기 위해서 만들어졌습니다.  

**handleActions**	  
이제 이 모듈의 초기 상태와 리듀서를 정의해주겠습니다.   
리듀서를 만들 땐, redux-actions 의 handleActions 를 사용하면 훨씬 편하게 작성 할 수 있습니다.	  

<br>

# 로거 미들웨어 만들기

src/lib/ 디렉토리에, loggerMiddleware.js 파일을 생성

### **`src/lib/loggerMiddleware.js`**

```javascript
const loggerMiddleware = store => next => action => {
  /* 미들웨어 내용 */
}
```
여기서 store 와 action 은 익숙하겠지만, next 는 익숙하지 않습니다.  
`next 는 여기서 store.dispatch`와 비슷한 역할을 하는데요,  
차이점은, **next(action) 을 했을 때에는 바로 리듀서로 넘기거나, 혹은 미들웨어가 더 있다면 다음 미들웨어 처리가 되도록 진행됩니다.**  
하지만, store.dispatch 의 경우에는 처음부터 다시 액션이 디스패치 되는 것 이기 때문에 현재 미들웨어를 다시한번 처리하게 됩니다. <br>

현재 상태를 한번 기록하고, 방금 전달 받은 액션을 기록하고,  
그 다음 리듀서에 의해 액션이 처리된 다음의 스토어 값을 기록

```javascript
const loggerMiddleware = store => next => action => {
  // 현재 스토어 상태값 기록
  console.log('현재 상태', store.getState());
  // 액션 기록
  console.log('액션', action);

  // 액션을 다음 미들웨어, 혹은 리듀서로 넘김
  const result = next(action);

  // 액션 처리 후의 스토어 상태 기록
  console.log('다음 상태', store.getState());
  console.log('\n'); // 기록 구분을 위한 비어있는 줄 프린트

  return result; // 여기서 반환하는 값은 store.dispatch(ACTION_TYPE) 했을때의 결과로 설정됩니다
}

export default loggerMiddleware; // 불러와서 사용 할 수 있도록 내보내줍니다.
```

<br>

# 미들웨어 적용

미들웨어는 store 를 생성 할 때에 설정을 하는데요.  
redux 모듈 안에 들어있는 applyMiddleware 를 사용하여 설정 할 수 있습니다.

### **`src/store.js`**

```javascript
import { createStore, applyMiddleware } from 'redux';
import modules from './modules';
import loggerMiddleware from './lib/loggerMiddleware';

// 미들웨어가 여러개인경우에는 파라미터로 여러개를 전달해주면 됩니다. 예: applyMiddleware(a,b,c)
// 미들웨어의 순서는 여기서 전달한 파라미터의 순서대로 지정됩니다.
const store = createStore(modules, applyMiddleware(loggerMiddleware))

export default store;
```

### **`src/index.js`**

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import { Provider } from 'react-redux';
import store from './store'

ReactDOM.render(
  <Provider store={store}>
    <App/>
  </Provider>,
  document.getElementById('root')
);
```

<br>

# 컴포넌트 App에 연결

### **`src/App.js`**

```javascript
import React, { Component } from 'react';
import { bindActionCreators } from 'redux';
import { connect } from 'react-redux';
import * as counterActions from './modules/counter';

class App extends Component {
  render() {
    const { CounterActions, number } = this.props;
    return(
      <div>
        <h1>{number}</h1>
        <button onClick={CounterActions.increment}>+</button>
        <button onClick={CounterActions.decrement}>-</button>
      </div>
    );
  }
}

export default connect(
  (state) => ({
    number: state.counter
  }),
  (dispatch) => ({
    CounterActions: bindActionCreators(counterActions, dispatch)
  })
)(App);
```

<br>

# redux-logger
오픈소스 커뮤니티에는 더 잘 만들어진 로거 미들웨어 적용하기    
Redux DevTool 을 사용한다면 redux-logger 는 사실 쓸모가 없습니다.   
Redux Devtool 이 이미 그 기능을 갖추고있고 훨씬 강력하기 때문이죠.   
하지만 Redux Devtool 을 사용하지못하는 환경이라면 redux-logger 는 매우 유용한 미들웨어입니다.  

```javascript
yarn add redux-logger
```

이전에 만들었던 `lib/loggerMiddleware.js` 로거 미들웨어는 더 이상 사용할 필요가 없어졌으니 삭제하셔도 됩니다.

### **`src/store.js`**

```javascript
import { createStore, applyMiddleware } from 'redux';
import modules from './modules';

import { createLogger } from 'redux-logger';

/* 로그 미들웨어를 생성 할 때 설정을 커스터마이징 할 수 있습니다.
  https://github.com/evgenyrodionov/redux-logger#options
*/
const logger = createLogger();

const store = createStore(modules, applyMiddleware(logger))

export default store;
```

<br>
