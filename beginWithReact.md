# Hello world !

使用 React.render 渲染 “hello world！”

[codepen](https://codepen.io/chenjsh36/pen/vdoOOr)

# React.createElement

createElement 创建元素：

```javascript
var rootElement = React.createElement(
  'div',
  {},
  React.createElement('h1', {}, 'Contacts'),
  React.createElement(
    'ul',
    {},
    React.createElement(
      'li',
      {},
      React.createElement('h2', {}, 'James Nelson'),
      React.createElement(
        'a',
        { href: 'mailto:james@jamesknelson.com' },
        'james@jamesknelson.com'
      )
    ),
    React.createElement(
      'li',
      {},
      React.createElement('h2', {}, 'Joe Citizen'),
      React.createElement(
        'a',
        { href: 'mailto:joe@example.com' },
        'joe@example.com'
      )
    )
  )
);

ReactDOM.render(rootElement, document.getElementById('react-app'));
```

[codepen](https://codepen.io/chenjsh36/pen/PQMqGL)

# React 是什么？

React 强调自己只是 MVC 的 V 层，所以 拿 react 和 Angular 或者 Ember 比较是不合理的，那么它被创建出来的目的是什么？优势是什么？缺点是什么？应该拿什么和它比较才合适？

## React 的一些关键点

JSX:在 JS 脚本里面写 HTML，JSX 会将 html 转化为脚本对象

* Virtual-DOM JavaScript 模拟的 DOM 树
* React.createClass--创建一个新组件（16 后 废弃了, 使用 create-react-class 或者 ES6 语法创建)
* render
* ReactDOM.render--将指定组件渲染到对应的 DOM 节点
* state--组件内部数据存放的地方
* getInitialState-- 设置组件初始值的方法
* setState
* props--组件传递数据的方式
  * propTypes-- 控制传递给子组件的 props 格
  * getDefaultProps--设置初始 props
* LifeCycle
  * componentWillMount — Fired before the component will mount
  * componentDidMount — Fired after the component mounted
  * componentWillReceiveProps — Fired whenever there is a change to props
  * componentWillUnmount — Fired before the component will unmount
* Events
  * onClick
  * onSubmit
  * onChange

## 为什么是 ReactDom.render 而不是 React.js ?

14 版本前为 React.render, 后来才改为 ReactDom.render， 让 React 更加模块化，且不仅仅只能在 Dom 上进行渲染

## 为什么要将 HTML 和 javascript 融合在一起 ？

### Component in JSX

```javascript
var HelloWorld = React.createClass({
  render: function() {
    return <div>Hello World!</div>;
  }
});

ReactDOM.render(<HelloWorld />, document.getElementById('app'));
```

### Component in JS

```javascript
var HelloWorld = React.createClass({
  displayName: 'HelloMessage',
  render: function() {
    return React.createElement('div', null, 'Hello World');
  }
});
```

## 关于 VirtualDOM

[React then isolates the changes between the old and new virtual DOM and then only updates the real DOM with the necessary changes](https://www.youtube.com/watch?v=-DX3vJiqxm4)

## React.createClass

16 版本 已经废弃，现在创建 class 有两种方法

* 采用 es6 语法 [codepen]()

```javascript
class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

* 引入 create-react-class

```javascript
var createReactClass = require('create-react-class');
var Greeting = createReactClass({
  render: function() {
    return <h1>Hello, {this.props.name}</h1>;
  }
});
```

## Events

[codepen](https://codepen.io/chenjsh36/pen/RMNMVY?editors=1111)

events -> state of component set a new value -> React render new virtual DOM -> React Diff change -> Real DOM update

## props + state + defaultProps + events 创建组件间通信

[codepen](https://codepen.io/chenjsh36/pen/qoEYZQ?editors=1111)

```jsx
class ShowList extends React.Component {
  render() {
    let list = this.props.list.map((item, index) => {
      return <li key={item}>{item}</li>;
    });
    return <ul>{list}</ul>;
  }
}
ShowList.defaultProps = {
  list: ['default']
};
class AddItem extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: ''
    };
    this.handleInputChange = this.handleInputChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleInputChange(e) {
    this.setState({
      value: e.target.value
    });
  }
  handleSubmit() {
    this.props.onAdd(this.state.value);
    this.setState({ value: '' });
  }
  render() {
    return (
      <div>
        <input value={this.state.value} onChange={this.handleInputChange} />
        <button onClick={this.handleSubmit}>Add</button>
      </div>
    );
  }
}
class Element extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      friends: ['cjj', 'cjy', 'cjw']
    };
    this.onAddFriend = this.onAddFriend.bind(this);
  }

  onAddFriend(value) {
    this.setState({
      friends: this.state.friends.concat([value])
    });
  }

  render() {
    return (
      <div>
        <h1>Friends </h1>
        <AddItem onAdd={this.onAddFriend} />
        <ShowList list={this.state.friends} />
      </div>
    );
  }
}
ReactDOM.render(<Element />, document.getElementById('root'));
```



## Life Cycle

[The Component Lifecycle](https://reactjs.org/docs/react-component.html#the-component-lifecycle)

### Mounting 时

* constructor
* componentWillMount
* render
* componentDidMount

```flow
st=>start: Mount
1=>operation: constructor()
2=>operation: componentWillMount()
3=>operation: render()
4=>operation: componentDidMount()
e=>end
st->1->2->3->4->e
```

### Update 时

* componentWillReceiveProps
* shouldComponentUpdate
* componentWillUpdate
* render
* componentDidUpdate

```flow
st=>start: Update
1=>operation: componentWillReceiveProps()
2=>operation: shouldComponentUpdate()
3=>operation: componentWillUpdate()
4=>operation: render()
5=>operation: componentDidUpdate()
e=>end
st->1->2->3->4->5->e
```

### Unmounting 时

* componentWillUnmount

### Error handing

* componentDidCatch （16 版本后新增处理错误钩子）

### 示例

[codepen](https://codepen.io/chenjsh36/pen/ZxGbrO?editors=1111)

* 首次加载触发 mounting 流程

* 点击 add 按钮触发 update 流程

* 点击 清空列表 触发 unmounting 流程

* 输入 error ，点击 add 按钮，触发 error handing 流程
