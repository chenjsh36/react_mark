# Redux

随着单页面应用越来越复杂，代码需要控制越来越多的  状态, 包括从服务器获取的数据、缓存的数据、没有更新到服务器上的数据、UI 层表现  用到的数据（ 激活的路由、激活的标签页、加载状态、分页器等等）这会导致代码失去控制、状态难以维护、 调试、debug 变得  困难、不可追踪，而 react 本身并不解决这个问题， 基于 Flux、CQRS、Event Sourcing， Redux 通过制定严格的条件规定更新如何  产生和什么时候产生，来让状态变化变得可预测

[官网](https://redux.js.org/)

## 三个原则

### Single source of truth

The state of your whole application is stored in an object tree within a single store.

### State is read-only

The only way to change the state is to emit an action, an object describing what happened.

### Changes are made with pure functions

To specify how the state tree is transformed by actions, you write pure reducers.

## 独立的 Redux

[codepen](https://codepen.io/chenjsh36/pen/rdVjRm?editors=1010)

Redux 可以独立开发，也可以配合 React、Angular 这些  框架进行开发，上面的例子  展示了纯 web 开发配合 Redux 的能力

## React + Redux

```JSX
import React from 'react'
import ReactDOM from 'react-dom'
import { createStore } from 'redux'
import Counter from './components/Counter'
import counter from './reducers'

const store = createStore(counter)
const rootEl = document.getElementById('root')

const render = () => ReactDOM.render(
  <Counter
    value={store.getState()}
    onIncrement={() => store.dispatch({ type: 'INCREMENT' })}
    onDecrement={() => store.dispatch({ type: 'DECREMENT' })}
  />,
  rootEl
)

render()
store.subscribe(render)
```
