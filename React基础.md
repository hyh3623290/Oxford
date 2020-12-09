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

# 数据通信

## 父子通信

​	父组件通过 `props` 传递数据给子组件，子组件通过调用父组件传来的函数传递数据给父组件，这两种方式是最常用的父子通信实现办法。

​	这种父子通信方式也就是典型的单向数据流，父组件通过 `props` 传递数据，子组件不能直接修改 `props`， 而是必须通过调用父组件函数的方式告知父组件修改数据。

## 兄弟组件通信

​	对于这种情况可以通过共同的父组件来管理状态和事件函数。比如说其中一个兄弟组件调用父组件传递过来的事件函数修改父组件中的状态，然后父组件将状态传递给另一个兄弟组件。

## 跨多层次组件通信

​	如果你使用 16.3 以上版本的话，对于这种情况可以使用 Context API

16.3以前，context由于一些问题不被建议使用。而从 v 16.3.0 开始，React 对 Context API 进行了改进，新的 Context API 具备更强的可用性。以前的context首先代码书写不够优雅，并且如果中间的一个组件shouleComponentUpdate返回false的话，后面的组件都不会进行更新。新的 Context API 改进了这一点：即便组件的 shouldComponentUpdate 返回 false，它仍然可以“穿透”组件继续向后代组件进行传播**，**进而确保了数据生产者和数据消费者之间数据的一致性。

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



## 任意组件

​	这种方式可以通过 Redux 或者 Event Bus 解决

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

🌰 - 手写一个发布订阅模式

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

1. 有两个参数 `props` 和 `state`，分别指接收到的新参数和当前组件的 `state` 对象，
2. 这个函数会返回一个对象用来更新当前的 `state` 对象，如果不需要更新可以返回 `null`
3. 该函数会在装载时或者接收到新的 `props` 或者调用了 `setState` 和 `forceUpdate` 时被调用。如当我们接收到新的属性想修改我们的 `state` ，就可以使用。

⚠️ **注意**

- 16.3 版本时，**只有父组件的更新**会触发getDerivedStateFromProps。在 React 16.4 中，**任何因素触发的组件更新流程**（包括由 this.setState 和 forceUpdate 触发的更新流程）都会触发 
- 这是个静态方法，所以不能在这个函数里使用 `this`。



## render

​	这个函数只做一件事，就是根据状态 `state` 和属性 `props` **渲染组件**，所以不要在这个函数内做其他业务逻辑

​	组件初始化的时候render方法的作用是**生成虚拟dom**，然后虚拟dom到真实dom是通过`ReactDOM.render`方法实现。当更新的时候同样会在render方法中生成新的虚拟dom，然后借助diff算法定位出两个虚拟dom的差异，从而针对发生变化的真实dom做定向更新。

## componentDidMount

​	组件装载之后调用，此时我们可以获取到 DOM 节点并操作，比如对 canvas，svg 的操作以及服务器请求等。

⚠️ 注意

1. 在 `componentDidMount` 中调用 `setState` 会触发一次额外的渲染，多调用了一次 `render` 函数，由于它是在浏览器刷新屏幕前执行的，所以用户对此没有感知，但是我们应当在开发中避免这样使用，因为这样会带来一定的性能问题，我们尽量在 `constructor` 中初始化我们的 `state` 对象。

## shouldComponentUpdate

```js
shouldComponentUpdate(nextProps, nextState)

// lodash可以做深度比较，一次递归到底，耗费性能: _.isEqual(nextProps.list, this.props.list)
```

​	首先要知道三点：

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

1. 用于读取最新的 DOM 数据。
2. 两个参数 `prevProps` 和 `prevState`，表示更新之前的 `props` 和 `state`。
3. 这个函数必须要和 `componentDidUpdate` 一起使用，并且要有一个返回值，默认是 `null`，这个返回值作为第三个参数传给 `componentDidUpdate`。

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



# 高阶组件

​	y = ax + b	->	x是原组件

## 属性代理

​	我们可以在这里拦截到 `props`， 可以按需对 `props` 进行增加、删除和修改操作，然后将处理后的 `props` 再传递给被包装的组件

```jsx
import React, { Component } from 'react'

const HOC = function(WrappedComponent) {
  return class WrapperComponent extends React.Component {
    render() {
      const newProps = {
        ...this.props,
        name: 'HOCCCCCCCC'
      }
      return <WrappedComponent {...newProps}/>
    }
  }
}

class MyComponent extends React.Component {
  render(){
    return <div>{this.props.name}</div>
  }
}
export default HOC(MyComponent)
```

```jsx
<HOC a="a"/>
```



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



**高阶组件实战**

```jsx
function HOCFactory(...params) {
  return function Factory(Cmp) {
    return class HOC extends Component {
      render() {
        return <Cmp {...this.props}/>
      }
    }
  }
}
// 使用方式1 - 装饰器方便做函数组合
@HOCFactory({})
class Cmp extends Component {}
// 使用方式2
HOCFactory({})(Cmp)
```

**注：**不要在render里使用高阶组件的写法，

```js
const Hoc = enhance(MyComponent) // render中
```

如上，将会导致首先先卸载原来的组件并渲染新的组件，因为它永远会创建一个新的函数。

## Render Props

核心思想是通过一个函数将class组件的state作为props传递给纯函数组件

```jsx
class Factory extends Component {
  constructor() {
    this.state = {
       // 多个组件公共的逻辑的数据
    }
  }
  render() {
    return <div>{this.props.render(this.state)}</div>
  }
}

const App = () => (
  <Factory render={(props) => <p>{props.x}</p>}/>x
  // render是一个函数组件
)
```



```jsx
class Mouse extends Component {
  constructor(props) {
    super(props)
    this.state = { x: 0, y: 0}
  }

  handleMove = (e) => {
    this.setState({
      x: e.clientX,
      y: e.clientY
    })
  }

  render() {
    return (
      <div onMouseMove={this.handleMove}>
        {this.props.render(this.state)}
      </div>
    )
  }
}

const App = (props) => (
  <div>
    <p>{props.other}</p>
    <Mouse render={({x, y}) => <h1>鼠标的位置是: ({x}, {y})</h1>}/>
  </div>
)
```

​	反正我Mouse就提供个数据，至于你怎么用这个数据以及用这个数据要渲染什么样的jsx之类的都由你定

​	妙啊。

对比：

HOC模式简单，但会增加组件层级，相应的会增加组件透传成本，及透传过程中是否有覆盖的，维护上风险大一些，Render Props的话代码比较简洁，学习成本高一些

# 事件机制

​	JSX 上写的事件并没有绑定在对应的真实 DOM 上，而是通过事件代理的方式，将所有的事件都统一绑定在了 `document` 上。这样的方式不仅减少了内存消耗，还能在组件挂载销毁时统一订阅和移除事件

​	另外冒泡到 `document` 上的事件也不是原生浏览器事件，而是 React 自己实现的合成事件（SyntheticEvent）。因此我们如果不想要事件冒泡的话，调用 `event.stopPropagation` 是无效的，而应该调用 `event.preventDefault`。

​	那么实现合成事件的目的是什么呢？总的来说在我看来好处有两点，分别是

- 合成事件首先抹平了浏览器之间的兼容问题，另外这是一个跨浏览器原生事件包装器，赋予了跨浏览器开发的能力
- 对于原生浏览器事件来说，浏览器会给监听器创建一个事件对象。如果你有很多的事件监听，那么就需要分配很多的事件对象，造成高额的内存分配问题。但是对于合成事件来说，有一个事件池专门来管理它们的创建和销毁，当事件需要被使用时，就会从池子中复用对象，事件回调结束后，就会销毁事件对象上的属性，从而便于下次复用事件对象。

**⚠️注意：**想获取原生的event可以通过`event.nativeEvent`。target：触发事件的元素，如button。currentTarget：绑定事件的元素，如document

**合成事件过程**

​	比如事件已经挂载完了，现在div触发事件，最终都会冒泡到document上。document会生成一个统一的react event。也就是合成事件的对象，然后它会再去派发事件。在合成事件这一层，它会再去派发事件，派发事件就是说我们能通过target事件知道谁触发了对吧，比如一个a标签，它所在的组件是哪一个我们就知道了，然后这个组件里面到底哪个绑定了onClick事件通过这种方式我们已经知道了。所以说通过判断它的target，判断target所在的组件，及它的事件绑定关系。我们就能触发到所有绑定了这个click事件的函数，然后再把这个合成事件的event传递进来去执行这个函数

# immutable.js

彻底拥抱不可变值，基于共享数据（不是深拷贝），速度好

```js
const map1 = Immutable.Map({ a:1, b:2 })
const map2 = map1.set('b', 50) // 修改b
map1.get('b') // 2
map2.get('b') // 50
```

# 组件复合

​	Composition，就是vue插槽的概念，props.children到底是什么，就是任意的一个js表达式，所以根据这个我们也可实现具名插槽的功能

```jsx
function Dialog(props) {
  return (
    <div style={{ border: "1px solid blue" }}>
      <div>
        {props.children.default}
      <div>
      <div>
        {props.children.footer}
      <div>
    <div>
  );
}

<Dialog>
  {{
    title: <h1>标题</h1>,
    content: <p>内容<p>
  }}
<Dialog>
```







# @