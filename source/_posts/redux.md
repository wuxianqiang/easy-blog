---
title: 深入浅出redux知识
date: 2019-06-30 18:23:43
tags: React
---
{% asset_img redux.png 图片 %}

redux状态管理的容器。

<!-- more -->

# 开始使用
```js
// 定义常量
const INCREMENT = 'INCREMENT'
const DECREMENT = 'DECREMENT'
// 编写处理器函数
const initState = { num: 0 }
function reducer(state = initState, action) {
  switch (action.type) {
    case INCREMENT:
      return { num: state.num + 1 }
    case DECREMENT:
      return { num: state.num - 1 }
    default:
      return state
  }
}
// 创建容器
const store = createStore(reducer)
```
reducer函数需要判断动作的类型去修改状态，需要注意的是函数必须要有返回值。此函数第一个参数是 `state` 状态，第二个参数是 `action` 动作，`action` 参数是个对象，对象里面有一个不为 `undefined` 的 `type` 属性，就是根据这个属性去区分各种动作类型。

在组件中这样使用
```js
const actions = {
  increment() {
    return { type: INCREMENT }
  },
  decrement() {
    return { type: DECREMENT }
  }
}
class Counter extends Component {
  constructor(props) {
    super(props);
    // 初始化状态
    this.state = {
      num: store.getState().num
    }
  }
  componentDidMount() {
    // 添加订阅
    this.unsubscribe = store.subscribe(() => {
      this.setState({ num: store.getState().num })
    })
  }
  componentWillUnmount() {
    // 取消订阅
    this.unsubscribe()
  }
  increment = () => {
    store.dispatch(actions.increment())
  }
  decrement = () => {
    store.dispatch(actions.decrement())
  }
  render() {
    return (
      <div>
        <span>{this.state.num}</span>
        <button onClick={this.increment}>加1</button>
        <button onClick={this.decrement}>减1</button>
      </div>
    );
  }
}
```
我们都知道组件中的 `state` 和 `props` 改变都会导致视图更新，每当容器里面的状态改变需要修改 `state`，此时就需要用到 `store` 中的 `subscribe` 订阅这个修改状态的方法，该方法的返回值是取消订阅，要修改容器中的状态要用`store` 中的 `dispatch` 表示派发动作类型，`store` 中的 `getState` 表示获取容器中的状态。

# bindActionCreators

为了防止自己手动调用 `store.dispatch` ，一般会使用redux的这个 `bindActionCreators` 方法来自动绑定 `dispatch` 方法，用法如下。
```js
let actions = {
  increment() {
    return { type: INCREMENT }
  },
  decrement() {
    return { type: DECREMENT }
  }
}

actions = bindActionCreators(actions, store.dispatch)

class Counter extends Component {
  constructor(props) {
    super(props);
    // 初始化状态
    this.state = {
      num: store.getState().num
    }
  }
  componentDidMount() {
    // 添加订阅
    this.unsubscribe = store.subscribe(() => {
      this.setState({ num: store.getState().num })
    })
  }
  componentWillUnmount() {
    // 取消订阅
    this.unsubscribe()
  }
  increment = () => {
    actions.increment()
  }
  decrement = () => {
    actions.decrement()
  }
  render() {
    return (
      <div>
        <span>{this.state.num}</span>
        <button onClick={this.increment}>加1</button>
        <button onClick={this.decrement}>减1</button>
      </div>
    );
  }
}

export default Counter;
```
# react-redux
这个库是连接库，用来和react和redux进行关联的，上面使用redux的时候发现一个痛点就是要订阅设置状态的方法还要取消订阅，而react-redux却可以通过props自动完成这个功能。

```js
import {Provider} from 'react-redux'
import {createStore} from 'redux'

const INCREMENT = 'INCREMENT'
const DECREMENT = 'DECREMENT'

const initState = { num: 0 }
function reducer(state = initState, action) {
  switch (action.type) {
    case INCREMENT:
      return { num: state.num + 1 }
    case DECREMENT:
      return { num: state.num - 1 }
    default:
      return state
  }
}
const store = createStore(reducer)

ReactDOM.render((
  <Provider store={store}>
    <Counter />
  </Provider>
), document.getElementById('root'))
```
`Provider` 是个高阶组件，需要传入store参数作为store属性，高阶组件包裹使用状态的组件。这样就完成了跨组件属性传递。
```js
import {connect} from 'react-redux'
const INCREMENT = 'INCREMENT'
const DECREMENT = 'DECREMENT'
let actions = {
  increment() {
    return { type: INCREMENT }
  },
  decrement() {
    return { type: DECREMENT }
  }
}

class Counter extends Component {
  render() {
    return (
      <div>
        <span>{this.props.num}</span>
        <button onClick={() => this.props.increment()}>加1</button>
        <button onClick={() => this.props.decrement()}>减1</button>
      </div>
    );
  }
}
const mapStateToProps = state => {
  return state
}
const mapDispatchToProps = actions

export default connect(mapStateToProps, mapDispatchToProps)(Counter);
```
组件中使用`connect`方法关联组件和容器，这个高阶函数，需要执行两次，第一次需要传入两个参数，第一个参数是将状态映射为属性，第二个是将`action`映射为属性，第二次需要传入组件作为参数。

# mapStateToProps

该参数是个函数返回对象的形式，参数是store中的 `state`，可以用来筛选我们需要的属性，防止组件属性太多，难以维护

比如我们状态是这样的`{ a: 1, b: 2 }` 想改成这样的`{ a: 1 }`，使用如下
```js
const mapStateToProps = state => {
  return { a: state.a }
}
```
# mapDispatchToProps
这个方法将action中的方法映射为属性，参数是个函数返回对象的形式，参数是store中的 `dispatch`，可以用来筛选`action`
```js
let actions = {
  increment() {
    return { type: INCREMENT }
  },
  decrement() {
    return { type: DECREMENT }
  }
}
```
现在`action`中有两个方法，我们只需要一个的话就可以这么做了。
```js
const mapDispatchToProps = dispatch => {
  return {
    increment: (...args) => dispatch(actions.increment(...args))
  }
}
```
# 原理

## createStore原理
现在你已经掌握了react和react-redux两个库的使用，并且知道他们的作用分别是干什么的，那么我们就看看原理，先学习redux原理，先写一个`createStore`方法。 
```js
import createStore from './createStore'

export {
  createStore
}
```
回顾一下`createStore`是怎么使用的，使用的时候需要传入一个处理器`reducer`函数，根据动作类型修改状态然后返回状态，只有在调用`dispatch`方法修改状态的时候才会执行`reducer` 才能得到新状态。
```js
import isPlainObject from './utils/isPlainObject'
import ActionTypes from './utils/actionTypes'


function createStore(reducer, preloadedState) {
  let currentState = preloadedState

  function getState() {
    return currentState
  }

  function dispatch(action) {
    // 判断是否是纯对象
    if (!isPlainObject(action)) {
      throw new Error('类型错误')
    }
    // 计算新状态
    currentState = currentReducer(currentState, action)
  }
  
  dispatch({ type: ActionTypes.INIT })
  
  return {
    dispatch,
    getState
  }
}

export default createStore
```
在调用 `dispatch` 方法的时候，需要传入一个对象，并且有个 `type` 属性，为了保证传入的参数的正确性，调用了`isPlainObject` 方法，判断是否是一个对象。

```js
function isPlainObject (obj) {
  if (typeof obj !== 'object' || obj === null) {
    return false
  }
  let proto = obj
  while (Object.getPrototypeOf(proto) !== null) {
    proto = Object.getPrototypeOf(proto)
  }
  return Object.getPrototypeOf(obj) === proto
}
export default isPlainObject
```

该方法的原理就是判断 `obj.__proto__ === Object.prototype` 是否相等。

redux中还有订阅和取消订阅的方法，每当状态改变执行订阅的函数。发布订阅是我们再熟悉不过的原理了，我就不多说了。
```js
function createStore(reducer, preloadedState) {
  let currentState = preloadedState
  let currentReducer = reducer
  let currentListeners = []
  let nextListeners = currentListeners

  // 拷贝
  function ensureCanMutateNextListeners() {
    if (nextListeners === currentListeners) {
      nextListeners = currentListeners.slice()
    }
  }

  function getState() {
    return currentState
  }
  function subscribe(listener) {
    if (typeof listener !== 'function') {
      throw new Error('类型错误')
    }
    // 订阅
    ensureCanMutateNextListeners()
    nextListeners.push(listener)
    return function unsubscribe() {
      ensureCanMutateNextListeners()
      const index = nextListeners.indexOf(listener)
      nextListeners.splice(index, 1)
    }
  }
  function dispatch(action) {
    // 判断是否是纯对象
    if (!isPlainObject(action)) {
      throw new Error('类型错误')
    }
    // 计算新状态
    currentState = currentReducer(currentState, action)
    // 发布
    const listeners = (currentListeners = nextListeners)
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }
  }
  dispatch({ type: ActionTypes.INIT })
  return {
    dispatch,
    getState,
    subscribe
  }
}

```
`ensureCanMutateNextListeners` 的作用是，如果是在 `listeners` 被调用期间发生订阅(subscribe)或者解除订阅(unsubscribe),在本次通知中并不会立即生效，而是在下次中生效。

代码里面有个值得注意的是调用了一次`dispatch` 方法，派发一次动作，目的是为了得到默认值，而且为了这个动作类型与众不同，防止定义的类型冲突，所以redux这么来写。
```js
const randomString = () => Math.random().toString(36).substring(7).split('').join('.')

const ActionTypes = {
  INIT: `@@redux/INIT${randomString()}`
}

export default ActionTypes
```

## bindActionCreators原理

`bindActionCreators` 在上面已经介绍了他的作用，就是为每个方法自动绑定`dispatch`方法。
```js
function bindActionCreator(actionCreator, dispatch) {
  return function() {
    return dispatch(actionCreator.apply(this, arguments))
  }
}

export default function bindActionCreators(actionCreators, dispatch) {
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch)
  }

  const boundActionCreators = {}
  for (const key in actionCreators) {
    const actionCreator = actionCreators[key]
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }
  return boundActionCreators
}
```
