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
