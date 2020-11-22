# SourceMap

sourceMap开发环境默认开启的，开启以后就很容易找到错误

```js
entry: ""
output: ""
devtool: "source-map"
// none关闭 source-map开启 
```

1. none：关闭
2. source-map：开启，大而全，并会产生对应的map文件
3. eval：不会产生单独的文件，但会显示行和列
4. cheap-module-source-map：不会产生列，但会产生单独的文件，但是不会跟代码关联起来，显示第几行
5. cheap-module-eval-source-map：不会产生文件，不会产生列

# React生命周期

render阶段：纯净且没有副作用，可能会被React暂停，中止或重新启动

```js
constructor
getDirevedStateFromFrops // newProps, setState, forceUpdate触发
shouldComponentUpdate
render
```

preCommit阶段：可以读取DOM

```js
getSnapShotBeforeUpdate
```

commit阶段：可以使用DOM，运行副作用，安排更新

```js
componentDidMount
componentDidUpdate
componentWillUnmount
```

⚠️即使你的 `props` 没有任何变化，由父组件的 `state` 的改动导致的 `render`，`getDirevedStateFromFrops`这个生命周期依然会被调用

## 执行流程

👋**挂载阶段**

- `constructor`
- `getDerivedStateFromProps`
- `render`
- `componentDidMount`

👋**更新阶段**

- `getDerivedStateFromProps`
- `shouldComponentUpdate`
- `render`
- `getSnapshotBeforeUpdate`
- `componentDidUpdate`

👋**卸载阶段**

- `componentWillUnmount`

## constructor

- 初始化组件的 state
- 给事件处理方法绑定 this

## getDerivedStateFromProps

```js
static getDerivedStateFromProps(props, state)
```

​	这是个静态方法，所以不能在这个函数里使用 `this`，有两个参数 `props` 和 `state`，分别指接收到的新参数和当前组件的 `state` 对象，这个函数会返回一个对象用来更新当前的 `state` 对象，如果不需要更新可以返回 `null`

​	该函数会在装载时，接收到新的 `props` 或者调用了 `setState` 和 `forceUpdate` 时被调用。如当我们接收到新的属性想修改我们的 `state` ，就可以使用。

⚠️**注意：**16.3 版本时，**只有父组件的更新**会触发getDerivedStateFromProps。在 React 16.4 中，**任何因素触发的组件更新流程**（包括由 this.setState 和 forceUpdate 触发的更新流程）都会触发 

## render

​	这个函数只做一件事，就是根据状态 `state` 和属性 `props` 渲染组件，所以不要在这个函数内做其他业务逻辑

​	组件初始化的时候render方法的作用是生成虚拟dom，然后虚拟dom到真实dom是通过`ReactDOM.render`方法实现。当更新的时候同样会在render方法中生成新的虚拟dom，然后借助diff算法定位出两个虚拟dom的差异，从而针对发生变化的真实dom做定向更新。

## componentDidMount

​	组件装载之后调用，此时我们可以获取到 DOM 节点并操作，比如对 canvas，svg 的操作以及服务器请求等。在 `componentDidMount` 中调用 `setState` 会触发一次额外的渲染，多调用了一次 `render` 函数，由于它是在浏览器刷新屏幕前执行的，所以用户对此没有感知，但是我们应当在开发中避免这样使用，因为这样会带来一定的性能问题，我们尽量在 `constructor` 中初始化我们的 `state` 对象。

## shouldComponentUpdate

```js
shouldComponentUpdate(nextProps, nextState)

// lodash可以做深度比较，一次递归到底，耗费性能: _.isEqual(nextProps.list, this.props.list)
```

​	首先要知道两点：

1. setState 函数在任何情况下都会导致组件重新渲染，甚至如下

```js
this.setState({number: this.state.number})
```

2. 如果是父组件重新渲染时，不管传入的 props 有没有变化，都会引起子组件的重新渲染。
3. 如果子组件有4个li，要动了其中的1个，那么很遗憾，另外三个也会重新渲染

​    我们可以比较 `this.props` 和 `nextProps` ，`this.state` 和 `nextState` 值是否变化，来确认返回 true 或者 `false`。`PureComponent`为我们自动添加了这个方法做一个浅比较。深度比较的话一次递归到底，耗费性能。Vue的defineProperty也是递归到底的，所以data也不要太深

## getSnapshotBeforeUpdate

```js
getSnapshotBeforeUpdate(prevProps, prevState)
```

​	用于读取最新的 DOM 数据。两个参数 `prevProps` 和 `prevState`，表示更新之前的 `props` 和 `state`。这个函数必须要和 `componentDidUpdate` 一起使用，并且要有一个返回值，默认是 `null`，这个返回值作为第三个参数传给 `componentDidUpdate`。

👋**滚动条例子**

```jsx
class ScrollingList extends React.Component {
  constructor(props) {
    super(props);
    this.listRef = React.createRef();
  }

  getSnapshotBeforeUpdate(prevProps, prevState) {
    // 增加了新的一条聊天消息
    // 获取滚动条位置，以便我们之后调整
    if (prevProps.list.length < this.props.list.length) {
      const list = this.listRef.current;
      return list.scrollHeight - list.scrollTop;
    }
    return null;
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    // snapshot 是上个生命周期的返回值，当有新消息加入时，调整滚动条位置，使新消息及时显示出来
    if (snapshot !== null) {
      const list = this.listRef.current;
      list.scrollTop = list.scrollHeight - snapshot;
    }
  }

  render() {
    return (
      <div ref={this.listRef}>{/* ...contents... */}</div>
    );
  }
}
```

## componentDidUpdate

```js
componentDidUpdate(prevProps, prevState, snapshot)
```

​	在这个方法中我们可以操作 DOM，发起 http 请求，也可以 `setState` 更新状态，但注意这里使用 `setState` 要有条件，不然就会陷入死循环。

## componentWillUnmount

​	可以执行一些清理工作，比如清除定时器，取消未完成的网络请求，或清理事件监听等。



​	⚠️React 16 提供了一个内置函数 **componentDidCatch**，如果 render() 函数抛出错误，则会触发该函数，打印错误日志，并且显示回退的用户界面。它的出现，解决了早期的 React 开发中，一个小的组件抛出错误将会破坏整个应用程序的情况

## --- 16.3以前 ---

👋**挂载阶段**

- `constructor`
- `componentWillMount`
- `render`
- `componentDidMount`

👋**更新阶段**

- `componentWillReceiveProps(nextProps) // 自身更新不调用`
- `shouldComponentUpdate(nextProps, nextState) `
- `componentWillUpdate(nextProps, nextState)`
- `render`
- `componentDidUpdate(prevProps, prevState)`

## componentWillReceiveProps

​	只要父组件更新就会触发

# setState

​	直接使用 `this.state.key = value` 方式来更新状态，这种方式 React 内部无法知道我们修改了组件，因此也就没办法更新到界面上。

​	`setState` 可以接收两个参数：第一个参数可以是对象或函数，第二个参数是函数。
 	第一个参数是对象的写法：

```jsx
this.setState({
    key: newState
});
```

 	第一个参数是函数写法：

```jsx
this.setState((prevState, props) => {
  return {
    key: prevState.key // prevState 是上一次的 state，props 是此次更新被应用时的 props
  }
})
```

## 合并

```js
handleClick () {
  this.setState({
      val: this.state.val + 1
  })
  this.setState({
      val: this.state.val + 1
  })
}
```

上面代码等价于

```js
// 后面的数据会覆盖前面的更改，所以最终只加了一次.
Object.assign(
  previousState,
  {val: state.val + 1},
  {val: state.val + 1},
)
```

如果想基于当前的 `state` 来计算出新的值，则传递一个函数可以让你在函数内访问到当前的 `state` 值。

```js
handleClick () {
  this.setState((state, props) => ({
    counter: state.counter + 1
  }));
  this.setState((state, props) => ({
    counter: state.counter + 1
  }));
}
```

## 同步&异步

08 don't understand - dirtyComponents - batchedUpdates - 事务流

​	setState后，newState会被存入一个pending队列，然后判断是否处于batch update这个机制中，是则保存组件于dirtyComponent中，先将需更新的组件存起来，稍后更新。否则遍历所有的dirtyCompoennts，调用updateComponent，然后去执行更新。进入更新流程后，会将 `isInsideEventHandler` 设置为 `true` 。

​	在正处于一个更新流程中调用 `setState` 并不会立即更新 `state` ，此时 `isInsideEventHandler` 为 `true`，所以会将更新放入 `dirtyComponents` 中等待稍后更新。所以setTimeout和原生事件没有被判断没有在那个batchUpdate中，不会放入 `dirtyComponent` 进行异步更新，其结果自然是同步的。

​	dirtyCompoennt就是说现在state已经被更新的组件

⚠️结论：setTimeout和原生事件中同步

## 不可变值

​	如何操作数组对象

```js
// 追加
let { array } = this.state
array: array.concat(10) // 不会影响原来的array
array: [...array, 10]

// 截取
array: array.slice(0, 3)

// 筛选
array: array.filter(k => k > 10)

// 其他操作
const arrayCoppy = array.slice() // 生成副本
arrayCoppy.balabaka // 修改完后再setState

// 操作对象的方式
obj: { ...this.state.obj, a: 10 }
obj: Object.assign({}, this.state.obj, {a: 10})
```

## 死循环

​	willUpdate和shouldUpdate，render不能用

# 高阶组件

​	y = ax + b   :   `y` 是改造之后输出的组件，`x` 是我们想改造的组件，`a` 和 `b` 就是改造的方法。

## 属性代理

​	我们可以在这里拦截到 `props`， 可以按需对 `props` 进行增加、删除和修改操作，然后将处理后的 `props` 再传递给被包装的组件

```jsx
const HOC = function(WrappedComponent) {
  return class WrapperComponent extends React.Component {
    render() {
      const newProps = {
        ...this.props,
        name: 'HOC'
      }
      return <WrappedComponent {...newProps}/>
    }
  }
}
```

```jsx
class MyComponent extends React.Component {
  render(){
    return <div>{this.props.name}</div>
  }
}
export default HOC(MyComponent)
```

​	以上操作就完成了 HOC 的 `props` 添加, 除了添加操作，当然你也可以自己尝试修改和删除操作，原理都是一样。

## 获取 refs的引用

​	当 `component` 被高阶组件包装过后，这时通过 `this.refs.component` 获取的并不是 `component`，而是其包装组件。

```jsx
class ParentComponent extends React.Component {
  constructor(props) {
    super(props)
    this.state = {} 
  }
  componentDidMount() {
    // childCP 是真正的子组件
    // childCpWrapper 是被包装的子组件
    console.log(this.childCp)
    console.log(this.childCpWrapper)
  }
  render () {
    return (
     <ChildComponent 
       ref={(withRouter) => { this.childCpWrapper = withRouter }}  
       // 这里获取的是`withRouter`组件，一般没啥用，这里写出来只是为了对比
       getInstance={(childCp) => { this.childCp = childCp }} 
       // 这里通过`getInstance`传一个回调函数接收`childComponent`实例即可
    />
    )
  }
 }
```

​	这样我们就解决了这个问题，不管用多少个高阶组件包装组件，我们都可以在父组件通过一个 `getInstance` 来获取到。

## 抽象state

​	高阶组件可以通过将被包装组件的状态提升到高阶组件自身内部实现。一个典型的场景是，利用高阶组件将受控组件需要自己维护的状态统一提升到高阶组件中。

⚠️09 HOC 稍后

# Hooks

​	Hook 是一个特殊的函数，它可以让你“钩入” React 的特性。

## useState

```js
const [name, setName] = useState('Nicolas')
setName('hyh')
console.log(name) // hyh
```

## useEffect

```js
useEffect(callBack) // 1.每次渲染都会执行副作用

useEffect(()=>{
  ...
}, [])	// 2.只调用一次副作用

useEffect(() => {
  console.log('count变化触发')
}, [count]) // 3. 只有count变化会触发副作用

useEffect(()=>{
  ...
  return ()=>{
  }
}, []) // 3.return内的内容将会在卸载时调用副作用（清除函数）
```

## useContext

​	使用下面代码就可以创建一个上下文。

```jsx
const ThemeContext = React.createContext() // 1

function App() {
  return (
     // 2
    <ThemeContext.Provider value={themes.light}>
      <ThemedButton />
    </ThemeContext.Provider>
  );
}

function ThemedButton(props) {
  const theme = useContext(ThemeContext) // 3

  return (
    <button style={{ background: theme.background, color: theme.foreground }}>
      I am styled by theme context!
    </button>
  )
}
```

## useReducer

​	提供类似 `Redux` 的功能

```js
const [state, dispatch] = useReducer((state, action) => {
    // 根据派发的 action 类型，返回一个 newState
}, initialArg, init)
```

​	`useReducer` 接收 `reducer` 函数作为参数，`reducer` 接收两个参数，一个是 `state`，另一个是 `action`，然后返回一个状态 `state` 和 `dispatch`，`state` 是返回状态中的值，而 `dispatch` 是一个可以发布事件来更新 `state` 的函数

```jsx
const initialState = {count: 0};

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    default:
      throw new Error()
  }
}

import React, { useReducer } from 'react'

function App() {
  const [state, dispatch] = useReducer(reducer, initialState)
    
  return (
    <div>
      <h1>you click {state.count} times</h1>
      <input type="button" onClick={()=> dispatch({type: 'increment'})} value="click me" />
    </div>
  ) 
}

export default App
```

## useCallback && useMemo

​	在 `shouldComponentUpdate` 中可以通过判断前后的 `props` 和 `state` 的变化，来判断是否需要阻止更新渲染。但使用函数组件形式失去了 `shouldComponentUpdate`，我们无法通过判断前后状态来决定是否更新，这就意味着函数组件的每一次调用都会执行其内部的所有逻辑，会带来较大的性能损耗。`useMemo` 和 `useCallback` 的出现就是为了解决这一性能问题。`useCallback` 和 `useMemo` 的第一个参数是一个执行函数，第二个参数可用于定义其依赖的所有变量。如果其中一个变量发生变化，则 `useMemo` 或者 `useCallback` 会运行。这两个 API 都会在组件第一次渲染的时候执行，之后会在其依赖的变量发生改变时再次执行，并且这两个 Hooks 都返回缓存的值，`useMemo` 返回缓存的变量， `useCallback` 返回缓存的函数。

```js

```

⚠️10 稍后



## useCount

​	表示state可以对外共享了

```jsx
const useCount = (initialCount = 0) => {
	const [count, setCount] = useState(initialCount)
  return [count, () => setCount(count+1), () => setCount(count-1)]
}

export default () => {
  const [count, increment, decrement] = useCount(1)
  return (
  	<div>666</div>
  )
}
```

# Redux

​	Redux 主要由三部分组成：**store、reducer 和 action**。**在 Redux 的整个工作过程中，数据流是严格单向的**。这一点一定一定要背下来，面试的时候也一定一定要记得说——不管面试官问的是 Redux 的设计思想还是工作流还是别的什么概念性的知识，开局先放这句话，准没错。

**1. 使用 createStore 来完成 store 对象的创建**

```js
// 引入 redux
import { createStore } from 'redux'
// 创建 store
const store = createStore(
    reducer,
    initial_state,
    applyMiddleware(middleware1, middleware2, ...)
);
```

​	这其中一般来说，只有 reducer 是你不得不传的。下面我们就看看 reducer 的编码形态是什么样的。

**2. reducer 的作用是将新的 state 返回给 store**

​	state 的计算过程就称为 reducer。一个 reducer 一定是一个纯函数，它可以有各种各样的内在逻辑，但它最终一定要返回一个 state：

```js
const reducer = (state, action) => {
  // 此处是各种样的 state处理逻辑，如下
  switch (action.type) {
    case "ADD":
      return {...state, count: state.count+1}
    case "MINUS":
      return {...state, count: state.count-1}
    default:
      return state;
  }
}
```

​	当我们基于某个 reducer 去创建 store 的时候，其实就是给这个 store 指定了一套更新规则：

**3. action 的作用是通知 reducer “让改变发生”**

​	View 会发送多种消息，这就需要定义多种 Action，如果每个都手写，会重复工作。我们可以定义一个函数来生成 Action，这个函数就称为 Action Creator。

```js
const ADD_TODO = 'ADD_TODO'

function addTodo(type, text) {
    return {
        type,
        payload: text
    }
}

const action = addTodo(ADD_TODO, 'Learn Redux')
```

**4. 派发 action，靠的是 dispatch**

combineReducer

```js
createStore(
	combineReducer({
    count: countReducer
  })
)
// store.getState().count
```

**5. store.subscribe()**

​	Store 允许使用 `store.subscribe()` 方法设置监听函数，State 发生变化时会自动执行这个函数。`store.subscribe` 方法返回一个函数，调用这个函数就可以解除监听。

```js
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
)

unsubscribe()
```

**6.中间件与异步操作**

​	Action 发出后，Reducer 会立刻算出 State，称为同步。如果 Action 发出后，过一段时间再执行 Reducer，怎样让 Reducer 在异步操作结束之后自动执行呢？这时候就需要用到新的工具：中间件（middleware）。

​	中间件其实就是一个函数，它对 `store.dispatch()` 方法进行了改造，在发出 Action 和执行 Reducer 这两步之间，添加了其他功能。

⚠️11 稍后

# 数据传递

## 父子通信

​	父组件通过 `props` 传递数据给子组件，子组件通过调用父组件传来的函数传递数据给父组件，这两种方式是最常用的父子通信实现办法。

​	这种父子通信方式也就是典型的单向数据流，父组件通过 `props` 传递数据，子组件不能直接修改 `props`， 而是必须通过调用父组件函数的方式告知父组件修改数据。

## 兄弟组件通信

​	对于这种情况可以通过共同的父组件来管理状态和事件函数。比如说其中一个兄弟组件调用父组件传递过来的事件函数修改父组件中的状态，然后父组件将状态传递给另一个兄弟组件。

## 跨多层次组件通信

​	如果你使用 16.3 以上版本的话，对于这种情况可以使用 Context API

## 任意组件

​	这种方式可以通过 Redux 或者 Event Bus 解决，另外如果你不怕麻烦的话，可以使用这种方式解决上述所有的通信情况。发布订阅模式

## props类型检查

```js
import PropTypes from 'prop-types'
class MyComponent extends Component { 
    //... 
}

MyComponent.propTypes = {
  list: PropTypes.arrayOf(PropTypes.object).isRequired
  // 这个list必须得是一个数组, 数组每一项是个对象, 必传
}
```



## EventEmitter

发布订阅模式

首先安装 events:

```js
npm install events -S
```

初始化

```jsx
// events.js
import { EventEmitter } from 'events'
export default new EventEmitter()
```

注册监听事件：

```jsx
// 引入我们定义的 EventEmitter 实例
import eventEmitter from './events'

eventEmitter.addListener('changeSiblingsData', (msg) => {
  this.setState({
    bus: msg
  })
  console.log(msg)
});
```

触发：

```jsx
import eventEmitter from './events'
eventEmitter.emit('changeSiblingsData', msg)
```

发布订阅者模式其实就是注册我们需要的事件，在某些操作的时候，触发该事件即可。它是由事件驱动的。



手写一个发布订阅模式

```js
class myEventEmitter {
  constructor() {
    // eventMap 用来存储事件和监听函数之间的关系
    this.eventMap = {};
  }
  // type 这里就代表事件的名称
  on(type, handler) {
    // hanlder 必须是一个函数，如果不是直接报错
    if (!(handler instanceof Function)) {
      throw new Error("哥 你错了 请传一个函数");
    }
    // 判断 type 事件对应的队列是否存在
    if (!this.eventMap[type]) {
      // 若不存在，新建该队列
      this.eventMap[type] = [];
    }
    // 若存在，直接往队列里推入 handler
    this.eventMap[type].push(handler);
  }
  // 别忘了我们前面说过触发时是可以携带数据的，params 就是数据的载体
  emit(type, params) {
    // 假设该事件是有订阅的（对应的事件队列存在）
    if (this.eventMap[type]) {
      // 将事件队列里的 handler 依次执行出队
      this.eventMap[type].forEach((handler, index) => {
        // 注意别忘了读取 params
        handler(params);
      });
    }
  }
  off(type, handler) {
    if (this.eventMap[type]) {
      this.eventMap[type].splice(this.eventMap[type].indexOf(handler) >>> 0, 1);
    }
  }
}
```



## context

​	16.3以前，context由于一些问题不被建议使用。而从 v 16.3.0 开始，React 对 Context API 进行了改进，新的 Context API 具备更强的可用性。以前的context首先代码书写不够优雅，并且如果中间的一个组件shouleComponentUpdate返回false的话，后面的组件都不会进行更新。新的 Context API 改进了这一点：即便组件的 shouldComponentUpdate 返回 false，它仍然可以“穿透”组件继续向后代组件进行传播**，**进而确保了数据生产者和数据消费者之间数据的一致性。

​	Context API 有 3 个关键的要素：React.createContext、Provider、Consumer。

```js
const AppContext = React.createContext(defaultValue) // 选择性的传默认值

const { Provider, Consumer } = AppContext
```

​	Provider提供value

```jsx
<Provider value={title: this.state.title, content: this.state.content}>
  <Title />
  <Content />
</Provider>
```

​	Consumer通过函数接收一个value

```jsx
<Consumer>
  {value => <div>{value.title}</div>}
</Consumer>

// 另一种方式获取
Child.contextType = ThemeContext // 静态属性

render() {
  const theme = this.context
  return (
    <div>{theme}</div>
  )
}
```

⚠️注意事项1

​	当 Consumer 没有对应的 Provider 时，value 参数会直接取创建 context 时传递给 createContext 的 defaultValue。一般会单独创建一个context文件，再导出它，然后在需要的地方引入。如果是函数组件的话无法使用类的静态属性，此时只能用Consumer。此外如果是多个context的话也要用Consumer，只不过它会获取最近的consumer。

​	对于简单的跨层级组件通信，我们可以使用发布-订阅模式或者 Context API 来搞定。但是随着应用的复杂度不断提升，需要维护的状态越来越多，组件间的关系也越来越难以处理的时候，我们就需要请出 Redux 来帮忙了。

⚠️注意事项2

以下这种方式，当每一次Provider重渲染时，里面的所有consumer都会重渲染，因为value总是被赋予新的对象

```jsx
<Provider value={{ a: 'a' }}>
	...
</Provider>
// {a: 'a'} === {a: 'a'} 返回false
```

为防止这种现象，应把value的状态放到state中

```jsx
this.state = {
  value: { a: 'a' }
}
<Provider value={this.state.value}>
	...
</Provider>
// 这样value没改变的话就不重新渲染
```



















# --

⚠️函数组件性能高，因为类组件使用的时候要实例化，而函数组件直接执行函数取返回结果即可。



⚠️注意：组件名称必须以大写字母开头，小写字母开头被视为原生 DOM 标签



⚠️只有 ES6 的箭头函数方式传参时，事件对象 event 需要显式的进行传递

```jsx
<button onClick={(event) => {this.handleClick(id, event)}}>click me</button>
```



⚠️Key是一种身份标识，每个 key 对应一个组件。 key 值在兄弟元素之间必须是唯一的，不过，好在它们不用是全局唯一，否则 key 值的问题将成为一个头号难题。React 中的唯一标识 key 在更新 DOM 时会用到，与虚拟 DOM diff 算法强关联



⚠️ 06 讲表单，要看看



【参考文献】

- muke专栏 React深度刨析
- lagou专栏 React深入浅出
- muke框架面试





# ⚠️

- 题 - 是否真的会了 - 是否隔了几天之后还能写出，并了解思路
- 难理解 - 专注 + 发散 - 拆解 - 类比比喻
- 知识点 - 记忆

