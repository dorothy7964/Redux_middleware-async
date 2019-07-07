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

# redux-thunk

### 비동기 작업을 처리하기 위한 미들웨어 사용해보기

미들웨어가 어떤 방식으로 작동하는지 이해를 했으니, 오픈소스 커뮤니티에 공개된 **미들웨어들을 설치하고 이를 통해 비동기 액션** 들을 다루는 방법들을 배워보겠습니다.   
여기서 다루는 미들웨어는 **redux-thunk, redux-promise-middleware, redux-pender** 입니다.   
이 세 라이브러리는 각각 다른 방식으로 비동기 액션을 처리하는데요, 한번 하나하나 직접 사용해보면서 익혀봅시다.  

```javascript
yarn add redux-thunk
```

### thunk 란?

hunk란, **특정 작업을 나중에 하도록 미루기 위해서 함수형태로 감싼것** 을 칭합니다.  

예를 들어서 여러분들이 1 + 1 을 지금 당장 하고싶다면 이렇게 하겠죠?

```javascript
const x = 1 + 2;
```

이 코드가 실행되면 1 + 2 의 연산이 바로 진행됩니다.

하지만 다음과 같이 하면 어떨까요?

```javascript
const foo = () => 1 + 2;
```

이렇게 하면, 1 + 2 의 연산이 코드가 실행 될 때 바로 이뤄지지 않고 나중에 foo() 가 호출 되어야만 이뤄집니다.

### redux-thunk 는 뭘 하는 미들웨어일까?

가장 간단히 설명하자면, **이 미들웨어는 객체 대신 함수를 생성하는 액션 생성함수를 작성 할 수 있게 해줍니다.**

리덕스에서는 기본적으로는 액션 객체를 디스패치합니다.   

일반 액션 생성자는, 다음과 같이 파라미터를 가지고 액션 객체를 생성하는 작업만합니다.  

```javascript
const actionCreator = (payload) => ({action: 'ACTION', payload});
```

만약에 특정 액션이 몇초뒤에 실행되게 하거나, 현재 상태에 따라 아예 액션이 무시되게 하려면, 일반 액션 생성자로는 할 수가 없습니다.  

하지만, redux-thunk 는 이를 가능케 합니다.

우선 1초뒤 액션이 디스패치되게 하는 예제코드를 살펴보겠습니다:

```javascript
const INCREMENT_COUNTER = 'INCREMENT_COUNTER';

function increment() {
  return {
    type: INCREMENT_COUNTER
  };
}

function incrementAsync() {
  return dispatch => { // dispatch 를 파라미터로 가지는 함수를 리턴합니다.
    setTimeout(() => {
      // 1 초뒤 dispatch 합니다
      dispatch(increment());
    }, 1000);
  };
}
```

이렇게 한다면 나중에 store.dispatch(incrementAsync()); 를 하면 INCREMENT_COUNTER 액션이 1초뒤에 디스패치됩니다.

이번엔 조건에 따라 액션을 디스패치하거나 무시하는 코드를 살펴봅시다

```javascript
function incrementIfOdd() {
  return (dispatch, getState) => {
    const { counter } = getState();

    if (counter % 2 === 0) {
      return;
    }

    dispatch(increment());
  };
}
```
만약에, **리턴하는 함수에서 dispatch, getState 를 파라미터로 받게 한다면 스토어의 상태에도 접근 할 수있습니다.** 따라서, **현재의 스토어 상태의 값에 따라 액션이 dispatch 될 지 무시될지 정해줄 수 있는것이죠.**

간단하게 정리를 하자면 redux-thunk 는 일반 액션 생성자에 날개를 달아줍니다. **보통의 액션생성자는 그냥 하나의 액션객체를 생성 할 뿐이지만 redux-thunk 를 통해 만든 액션생성자는 그 내부에서 여러가지 작업을 할 수 있습니다. 이 곳에서 네트워크 요청을 해도 무방하죠.** 또한, 이 안에서 **액션을 여러번 디스패치 할 수도 있습니다.**

### 여기서 dispatch, getState 는 어디서 오는건가요?

간단합니다. **redux-thunk 미들웨어에서, 전달받은 액션이 함수 형태 일 때, 그 함수에 `dispatch` 와 `getState` 를 넣어서 실행해줍니다.** 실제로, redux-thunk 의 코드는 정말로 간단합니다. 한번 코드를 보는게 작동방식을 이해는데에 도움이 될거예요.

```javascript
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
```

### 설치와 적용

npm 설치 후 그 다음엔, 스토어를 생성 할 때 미들웨어를 적용하세요

### **`src/store.js`**

```javascript
import { createStore, applyMiddleware } from 'redux';
import modules from './modules';

import { createLogger } from 'redux-logger';
import ReduxThunk from 'redux-thunk';

/* 로그 미들웨어를 생성 할 때 설정을 커스터마이징 할 수 있습니다.
   https://github.com/evgenyrodionov/redux-logger#options
*/
const logger = createLogger();

const store = createStore(modules, applyMiddleware(logger, ReduxThunk))

export default store;
```

### 카운터를 비동기적으로 만들어보기

기존에 작동하던 카운터를 비동기적으로 작동하도록 코드를 추가해보겠습니다.

### **`src/modules/counter.js`**

```javascript
import { handleActions, createAction } from 'redux-actions';

const INCREMENT = 'INCREMENT';
const DECREMENT = 'DECREMENT';

export const increment = createAction(INCREMENT);
export const decrement = createAction(DECREMENT);


export const incrementAsync = () => dispatch => {
  // 1초 뒤 액션 디스패치
  setTimeout(
    () => { dispatch(increment()) },
    1000
  );
}

export const decrementAsync = () => dispatch => {
  // 1초 뒤 액션 디스패치
  setTimeout(
    () => { dispatch(decrement()) },
    1000
  );
}

export default handleActions({
  [INCREMENT]: (state, action) => state + 1,
  [DECREMENT]: (state, action) => state - 1
}, 0);
```

그 다음에는, App 컴포넌트에서  
`increment` -> `incrementAsync`,  
`decrement` -> `decrementAsync` 로 치환하세요.

```javascript
import React, { Component } from 'react';
import { bindActionCreators } from 'redux';
import { connect } from 'react-redux';
import * as counterActions from './modules/counter';

class App extends Component {
  render() {
    const { CounterActions, number } = this.props;
    return (
      <div>
        <h1>{number}</h1>
        <button onClick={CounterActions.incrementAsync}>+</button>
        <button onClick={CounterActions.decrementAsync}>-</button>
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

다음 섹션에서는 redux-thunk 를 사용하여 웹 요청을 처리하는 방법을 배워보겠습니다.
