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