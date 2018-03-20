# Redux 源码解析

## 源码结构

```
.src
├── utils                #工具函数
├── applyMiddleware.js
├── bindActionCreators.js
├── combineReducers.js
├── compose.js
├── createStore.js  
└── index.js             #入口 js
```

### index.js

这是整个项目的入口

```javascript
import createStore from './createStore';
import combineReducers from './combineReducers';
import bindActionCreators from './bindActionCreators';
import applyMiddleware from './applyMiddleware';
import compose from './compose';
import warning from './utils/warning';
import __DO_NOT_USE__ActionTypes from './utils/actionTypes';

/*
 * This is a dummy function to check if the function name has been altered by minification.
 * If the function has been minified and NODE_ENV !== 'production', warn the user.
 */
function isCrushed() {}

if (
  process.env.NODE_ENV !== 'production' &&
  typeof isCrushed.name === 'string' &&
  isCrushed.name !== 'isCrushed'
) {
  warning('...');
}

export {
  createStore,
  combineReducers,
  bindActionCreators,
  applyMiddleware,
  compose,
  __DO_NOT_USE__ActionTypes
};
```

**isCrushed** 函数用于判断当前环境是否为 非发布环境下 isCrushed 是否被压缩了，如果被压缩的话会给开发者一个警告，这种提示在很多类库都有。

最后暴露 **createStore、combineReducers、bindActionCreators、applyMiddlewars、compose** 以及私有 actionType **\_\_DO_NOT_USE_ActionTypes** 给开发者使用

### creatStore.js

```javascript
import $$observable from 'symbol-observable'

import ActionTypes from './utils/actionTypes'
import isPlainObject from './utils/isPlainObject'

/**
 * 这个函数用来创建一个 Redux store 来存放应用中的 state 树，调用 dispatch 是改变 state 的唯一方法，每个应用都应该只有一个 store，你可以使用 combineReducers 来将多个 reducers 合并成单个 reducer以区分不同的 state
 *
 * @param {Function} reducer
 * 通过当前的 state 和 action 返回下一个 state 树
 *
 * @param {any} [preloadedState]
 * 初始化 state 数据，可选的参数，如果需要融合来自服务器的数据或者来自之前用户的 session 等情况下可以使用，如果你使用 combineReducers 来生成顶级 reducer ，那么preloadedState 必须为对象
 *
 * @param {Function} [enhancer]
 * store 的增强器函数，可以指定为 第三方的中间件，时间旅行，持久化等等，但是这个函数只能用 Redux 提供的 applyMiddleware 函数来生成；
 *
 * @returns {Store} A Redux store that lets you read the state, dispatch actions
 * and subscribe to changes.
 * 调用完返回一个Redux Store供开发者获取 state 和 触发 action 以及监听状态改变
 */
export default function createStore(reducer, preloadedState, enhancer) {
  // 根据参数个数，来指定 reducer、preloadedState 和 enhancer
  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState
    preloadedState = undefined
  }
  // 如果 enhancer 存在且为函数，那么调用 enhancer，并且终止当前函数执行
  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.')
    }

    return enhancer(createStore)(reducer, preloadedState)
  }

  if (typeof reducer !== 'function') {
    throw new Error('Expected the reducer to be a function.')
  }

  let currentReducer = reducer
  let currentState = preloadedState
  let currentListeners = []
  let nextListeners = currentListeners
  let isDispatching = false

  // 根据当前的监听函数的列表生成新的下一个监听函数列表的引用
  function ensureCanMutateNextListeners() {
    if (nextListeners === currentListeners) {
      nextListeners = currentListeners.slice()
    }
  }

  function getState() {
    ...
  }
  function subscribe(listener) {
    ...
  }

  function dispatch(action) {
    ...
  }

  function replaceReducer(nextReducer) {
    ...
  }

  function observable() {
    ...
  }

  dispatch({ type: ActionTypes.INIT })

  return {
    dispatch,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
  }
}
```

createStore 的功能和参数在代码注释中已经解释了，接下来看下里面实现的几个参数：

#### getState

```javascript
function getState() {
  if (isDispatching) {
    throw new Error(
      'You may not call store.getState() while the reducer is executing. ' +
        'The reducer has already received the state as an argument. ' +
        'Pass it down from the top reducer instead of reading it from the store.'
    );
  }

  return currentState;
}
```

从名字上就知道是返回当前 state 树，如果在 dispatching 中则会抛出错误；createStore 中的 currentState 存储当前的 state树，这是一个闭包，并且所有的状态操作都是在修改这个引用

#### subscribe

```javascript
  function subscribe(listener) {
    if (typeof listener !== 'function') {
      throw new Error('Expected listener to be a function.')
    }

    if (isDispatching) {
      throw new Error(
        ...
      )
    }

    let isSubscribed = true

    ensureCanMutateNextListeners()
    nextListeners.push(listener)

    return function unsubscribe() {
      if (!isSubscribed) {
        return
      }

      if (isDispatching) {
        throw new Error(
          ...
        )
      }

      isSubscribed = false

      ensureCanMutateNextListeners()
      const index = nextListeners.indexOf(listener)
      nextListeners.splice(index, 1)
    }
  }
```

这个 suscribe函数可以给 store 状态添加 d 订阅监听函数，一旦调用 dispatch，所有的监听函数都会被执行；

**ensureCanMutateNextListeners** 函数调用时会判断 nextListeners 是否等于 currentListeners，相等则通过 slice 返回一个新的引用地址；

调用 subscribe会返回 **unsubscribe** 用于解绑监听函数钩子，通过 nextListeners 删除对应的函数来实现，同时在 **subscribe** 中定义了**isSubscribed** 来判断是否已经解绑；

#### dispatch.js

```javascript
  function dispatch(action) {
    if (!isPlainObject(action)) {
      throw new Error(
        ...
      )
    }

    if (typeof action.type === 'undefined') {
      throw new Error(
        ...
      )
    }

    if (isDispatching) {
      throw new Error('Reducers may not dispatch actions.')
    }

    // 对抛出 error 的兼容，但是无论如何都会继续执行 isDispatching = false 的操作
    try {
      isDispatching = true
      // 调用 reducer 处理当前状态和 action，然后将当前状态放回 currentState
      currentState = currentReducer(currentState, action)
    } finally {
      isDispatching = false
    }

    const listeners = currentListeners = nextListeners
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }

    return action
  }
```

dispatch用于触发状态改变，接受 action 作为参数，并判断 action是否为对象（isPlainObject)且含有 type 属性，然后调用 **currentReducer** 来 生成新的状态，最后遍历监听函数的列表，依次执行并返回 action

#### replaceReducer

```javascript
function replaceReducer(nextReducer) {
  if (typeof nextReducer !== 'function') {
    throw new Error('Expected the nextReducer to be a function.');
  }

  currentReducer = nextReducer;
  dispatch({ type: ActionTypes.REPLACE });
}
```

**replaceReducer**用于替换当前 reducer 函数，如果项目中使用了代码分割并且想要动态加载 reducers，那么这个函数就派上用场，实现上很简单，只需要将 currentReducer 设置为出入的 nextReducer，最后触发 ActionTypes.REPLACE 就完成了 状态树的初始化

####observable

```javascript
function observable() {
  const outerSubscribe = subscribe;
  return {
    subscribe(observer) {
      if (typeof observer !== 'object') {
        throw new TypeError('Expected the observer to be an object.');
      }

      function observeState() {
        if (observer.next) {
          observer.next(getState());
        }
      }

      observeState();
      const unsubscribe = outerSubscribe(observeState);
      return { unsubscribe };
    },

    [$$observable]() {
      return this;
    }
  };
}
```

**observable** 函数提供给其他观察者模式或响应库使用[具体文档](https://github.com/tc39/proposal-observable)

### compose.js

```javascript
export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg;
  }

  if (funcs.length === 1) {
    return funcs[0];
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)));
}
```

**compose**接收一组函数，然后从右向左来组合这些函数，最后返回一个组合函数，例如：

```javascript
function a() {
  console.log('a:', arguments);
  return 'a';
}
function b() {
  console.log('b:', arguments);
  return 'b';
}
function c() {
  console.log('c:', arguments);
  return 'c';
}
var d = [a, b, c];
e = compose(...d);
e('begin'); // a(b(c('begin')))
// c: Arguments ["begin", callee: ƒ, Symbol(Symbol.iterator): ƒ]
// b: Arguments ["c", callee: ƒ, Symbol(Symbol.iterator): ƒ]
// a: Arguments ["b", callee: ƒ, Symbol(Symbol.iterator): ƒ]
// "a"
```


### applyMiddleware.js
``` javascript
import compose from './compose'

/**
 * Creates a store enhancer that applies middleware to the dispatch method
 * of the Redux store. This is handy for a variety of tasks, such as expressing
 * asynchronous actions in a concise manner, or logging every action payload.
 *
 * 通过组合多个中间件来创建一个 enhancer，可供各种任务场景使用，例如提供异步 action，打印各种 action 日志
 *
 * See `redux-thunk` package as an example of the Redux middleware.
 * 
 * Because middleware is potentially asynchronous, this should be the first
 * store enhancer in the composition chain.
 * 由于中间件可能是异步的，所以需要作为第一个 enhancer 放在组合链中
 *
 * Note that each middleware will be given the `dispatch` and `getState` functions
 * as named arguments.
 *
 * @param {...Function} middlewares The middleware chain to be applied.
 * @returns {Function} A store enhancer applying the middleware.
 */
export default function applyMiddleware(...middlewares) {
  return (createStore) => (...args) => {
    const store = createStore(...args)
    let dispatch = () => {
      throw new Error(
        `Dispatching while constructing your middleware is not allowed. ` +
        `Other middleware would not be applied to this dispatch.`
      )
    }
    let chain = []

    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```
从函数可以看到执行过程通过 ```applyMiddleware(middlewares)(createStore)(reducer, preloadState, enhancer)``` 最后得到 **store** 和新的 **dispacth**

比较特别的是传给 chain 中中间件的 dispatch 是一个空函数，表示中间件在构造的时候是不允许生成 action 的，因为其他中间件无法对该 action 做出响应

最后返回的 dispatch 不是原来的 dispatch，而是经过所有中间件组合后放回的 dispatch

### combineReducers.js
``` javascript
export default function combineReducers(reducers) {
  // 获得最终合法的 reducers
  const reducerKeys = Object.keys(reducers)
  const finalReducers = {}
  for (let i = 0; i < reducerKeys.length; i++) {
    const key = reducerKeys[i]

    if (process.env.NODE_ENV !== 'production') {
      if (typeof reducers[key] === 'undefined') {
        warning(`No reducer provided for key "${key}"`)
      }
    }

    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]
    }
  }
  const finalReducerKeys = Object.keys(finalReducers)

  let unexpectedKeyCache
  if (process.env.NODE_ENV !== 'production') {
    unexpectedKeyCache = {}
  }

  // 确保 reducer 在初始化和对定义外的 action 的处理是否正常
  let shapeAssertionError
  try {
    assertReducerShape(finalReducers)
  } catch (e) {
    shapeAssertionError = e
  }

  return function combination(state = {}, action) {
    if (shapeAssertionError) {
      throw shapeAssertionError
    }

    if (process.env.NODE_ENV !== 'production') {
      const warningMessage = getUnexpectedStateShapeWarningMessage(state, finalReducers, action, unexpectedKeyCache)
      if (warningMessage) {
        warning(warningMessage)
      }
    }

    let hasChanged = false
    const nextState = {}
    for (let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys[i]
      const reducer = finalReducers[key]
      const previousStateForKey = state[key]
      const nextStateForKey = reducer(previousStateForKey, action)
      if (typeof nextStateForKey === 'undefined') {
        const errorMessage = getUndefinedStateErrorMessage(key, action)
        throw new Error(errorMessage)
      }
      nextState[key] = nextStateForKey
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    return hasChanged ? nextState : state
  }
}

```
最终返回的**combine** 函数才是真正的 reducer ，每次对传入的 action 都会遍历 finalReducerKeys 的 **key**

先通过 **key** 获得之前的状态：```previousStateForKey = state[key]```

从这行代码可以明白为什么传入的 key 必须为 state 对应的变量名

然后拿到下一个状态的： ```nextStateForKey = reducer(previousStateForKey, action)```

再更新新的 key 对应的值： ```nextState[key] = nextStateForKey```

每次判断有修改：```hasChanged = hasChanged || nextStateForKey !== previousStateForKey```

最后如果 hasChanged 为 true 则返回 nextState, 否则返回原来的 state

### bindActionCreators.js

``` javascript
function bindActionCreator(actionCreator, dispatch) {
  return function() { return dispatch(actionCreator.apply(this, arguments)) }
}

export default function bindActionCreators(actionCreators, dispatch) {
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch)
  }

  if (typeof actionCreators !== 'object' || actionCreators === null) {
    throw new Error(
      `bindActionCreators expected an object or a function, instead received ${actionCreators === null ? 'null' : typeof actionCreators}. ` +
      `Did you write "import ActionCreators from" instead of "import * as ActionCreators from"?`
    )
  }

  const keys = Object.keys(actionCreators)
  const boundActionCreators = {}
  for (let i = 0; i < keys.length; i++) {
    const key = keys[i]
    const actionCreator = actionCreators[key]
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }
  return boundActionCreators
}

```
通常情况下我们会定义一个 action 生成函数：
``` javascript
let actionCreate = (text) => {
  return {
    type: 'action_name',
    payload: text
  }
}
```
然后需要触发 action 的时候：
``` javascript
store.dispatch(actionCreate('hello world'))
```

而 bindActionCreators 则是帮我们节省了 **store.dispatch** 的调用：
``` javascript
let bindActionCreate = bindActionCreators(actionCreate, store.dispatch)
//触发 action：
bindActionCreate('hello world')
```

具体就是接收传入的 action 生成器和 store.dispatch, 返回一个执行便触发绑定了这两者的函数：

``` javascript
return function() { return dispatch(actionCreator.apply(this, arguments)) }
```

如果传入的是对象，则遍历所有的 key，依次进行绑定从而实现如下的效果：

``` javascript
let obj = {
  action1: function (text) { return { type: 'action1', payload: text}},
  action2: function (text) { return { type: 'action2', payload: text}},
}

let objAfterBind = bindActionCreators(obj, store.dispatch)

objAfterBind.action1('hello') // 等于 store.dispatch(obj.action1('hello'))
objAfterBind.action2('hello') // 等于 store.dispatch(obj.action2('hello'))

```