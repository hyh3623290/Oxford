[toc]



# 声明式与命令式

- 命令式：告诉计算机怎么做
- 声明式：告诉计算机我们要什么

# 插值表达式

- 不能输出对象，会报错（布尔值，空，未定义会被忽略）
- 可以放数组，会自动展开，去掉逗号

```jsx
const arr1 = [<p>111</p>, <p>111</p>, <p>111</p>]
const arr2 = [1,2,3]
const element = (
  <div>{arr}</div>
);

// 解析后 arr1
<div>
	<p>111</p>
  <p>111</p>
  <p>111</p>
</div>

// arr2
123
```



# 新旧生命周期

## 初始化

1. **getDefaultProps**：这个函数会在组件创建之前被调用一次（有且仅有一次），它被用来初始化组件的 Props；
2. **getInitialState**：用于初始化组件的 state 值；
3. **componentWillMount**：在组件创建后、render 之前，会走到 componentWillMount 阶段。这个阶段我个人一直没用过、非常鸡肋。后来React 官方已经不推荐大家在 componentWillMount 里做任何事情、到现在 **React16 直接废弃了这个生命周期**，足见其鸡肋程度了；
4. **render**：这是所有生命周期中唯一一个你必须要实现的方法。一般来说需要返回一个 jsx 元素，这时 React 会根据 props 和 state 来把组件渲染到界面上；不过有时，你可能不想渲染任何东西，这种情况下让它返回 null 或者 false 即可；
5. **componentDidMount**：会在组件挂载后（插入 DOM 树中后）立即调用，标志着组件挂载完成。一些操作如果依赖获取到 DOM 节点信息，我们就会放在这个阶段来做。此外，这还是 React 官方推荐的发起 ajax 请求的时机。该方法和 componentWillMount 一样，有且仅有一次调用。



## 更新

### state 更新流程：

1. shouldComponentUpdate: 当组件的 state 或 props 发生改变时，都会首先触发这个生命周期函数。它会接收两个参数：nextProps, nextState——它们分别代表传入的新 props 和新的 state 值。拿到这两个值之后，我们就可以通过一些对比逻辑来决定是否有 re-render（重渲染）的必要了。如果该函数的返回值为 false，则生命周期终止，反之继续；
2. componentWillUpdate：当组件的 state 或 props 发生改变时，会在渲染之前调用 componentWillUpdate。componentWillUpdate **是 React16 废弃的三个生命周期之一**。过去，我们可能希望能在这个阶段去收集一些必要的信息（比如更新前的 DOM 信息等等），现在我们完全可以在 React16 的 getSnapshotBeforeUpdate 中去做这些事；
3. componentDidUpdate：componentDidUpdate() 会在UI更新后会被立即调用。它接收 prevProps（上一次的 props 值）作为入参，也就是说在此处我们仍然可以进行 props 值对比（再次说明 componentWillUpdate 确实鸡肋哈）。



### props 更新流程：

- componentWillReceiveProps：它在Component接受到新的 props 时被触发。componentWillReceiveProps 会接收一个名为 nextProps 的参数（对应新的 props 值）。**该生命周期是 React16 废弃掉的三个生命周期之一**。在它被废弃前，一些同学可能习惯于用它来比较 this.props 和 nextProps 来重新setState。在 React16 中，我们用一个类似的新生命周期 getDerivedStateFromProps 来代替它。



## 卸载

​	`componentWillUnmount()` 会在组件卸载及销毁之前直接调用。在此方法中执行必要的清理操作，例如，清除 timer，取消网络请求或清除在 `componentDidMount()` 中创建的订阅等。



## 新的生命周期

- **Render 阶段**：用于计算一些必要的状态信息。这个阶段可能会被 React 暂停，这一点和 React16 引入的 Fiber 架构（我们后面会重点讲解）是有关的；
- **Pre-commit阶段**：所谓“commit”，这里指的是“更新真正的 DOM 节点”这个动作。所谓 Pre-commit，就是说我在这个阶段其实还并没有去更新真实的 DOM，不过 DOM 信息已经是可以读取的了；
- **Commit 阶段**：在这一步，React 会完成真实 DOM 的更新工作。Commit 阶段，我们可以拿到真实 DOM（包括 refs）。



- 挂载过程：
  - **constructor**
  - **getDerivedStateFromProps**
  - **render**
  - **componentDidMount**
- 更新过程：
  - **getDerivedStateFromProps**
  - **shouldComponentUpdate**
  - **render**
  - **getSnapshotBeforeUpdate**
  - **componentDidUpdate**
- 卸载过程：
  - **componentWillUnmount**



**getDerivedStateFromProps**

​	在 render 方法之前调用，它应返回一个对象来更新 state，如果返回 null 则不更新任何内容。它接收 props 和 state 两个入参，从调用时机和实际场景上，它都微妙地对标了**componentWillReceiveProps** 这个已经被废弃的旧方法。
注意！React官方并不推荐开发者使用该方法：

​	此方法适用于罕见的用例，即 state 的值在任何时候都取决于 props。否则，派生状态会导致代码冗余，并使组件难以维护。

​	更何况，大部分的派生状态咱们都有更好的替代方案可以解决



**getSnapshotBeforeUpdate**

​	这个方法会接受 prevProps、prevState 两个入参

​	**这个生命周期函数必须有返回值**——它的返回值会作为第三个参数传递给 `componentDidUpdate`。

​	注意，因为走到这一步时，React 已经更新上了所有状态，所以新状态可以通过 this.props、this.state 获取。所以结合两个入参，我们可以拿到所有的新旧状态。不过即便这个方法能提供的信息如此丰富，它也并不常用：

​	此用法并不常见，但它可能出现在 UI 处理中，如需要以特殊方式处理滚动位置的聊天线程等。

​	结合个人经验来看，因为 getSnapshotBeforeUpdate 触发时，真实的 DOM 节点还没有更新；此外，它又可以和 componentDidUpdate 通信。大家知道，一些场景下，我们是需要对更新前后的 DOM 节点信息作一些对比或是处理的。比如说我想知道更新前后，某一个 div 的位置移动了多少，以此来决定是否来把它矫正回原位、或者是直接帮它移动一个更合适的距离呢？这种情况下，用 getSnapshotBeforeUpdate 就再合适不过啦~



# 组件通信

​	 单向数据流的优点：数据流动变得可预测，使得定位程序错误变得简单

## 父子组件通信

**父传子**

​	props

**子传父**

​	父组件将作用域为自身的函数传递给子组件，子组件在调用该函数时，把目标数据以函数入参的形式交到父组件手中即可。



## 兄弟组件通信

​	通过相同的父组件通信



## 深层嵌套的组件间通信

​	React16.3 开始，Context API 得到了升级

```js
// UserContext.js
const UserContext = React.createContext('default value')
const UserProvider = UserContext.Provider
const UserConsumer = UserContext.Consumer

export { UserProvider, UserConsumer }
```

```js
// App.js
import { UserProvider } from './UserContext.js'
render() {
  return (
  	<UserProvider value="hello context">
      <Component />
    <UserProvider>
  )
}
```

```jsx
// Component.js
import { UserConsumer } from './UserContext.js'
render() {
  return (
  	<UserConsumer>
      {
        (username) => {
          return <div>{username}</div>
        }
      }
    <UserConsumer>
  )
};
```

​	不使用consumer方式

```jsx
import { UserConsumer } from './UserContext.js'

class Child extends Component {
  render() {
    console.log(this) // 发现里面有个context属性
    const theme = this.context
    return (
      <div>{theme}</div>
    )
  }
}
Child.contextType = ThemeContext // 静态属性
// 如果是函数组件的话无法使用类的静态属性，此时只能用Consumer。
```

​	此外如果是多个context的话也要用Consumer，只不过它会获取最近的consumer。



⚠️ 以下这种方式，当每一次Provider重渲染时，里面的所有consumer都会重渲染，因为value总是被赋予新的对象

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



## 任意组件间通信

​	 Event-Bus

```js
npm install events -S

// events.js
import { EventEmitter } from 'events'
export default new EventEmitter()
```

```js
// 1. 导入
import eventEmitter from './events'

// 2. 触发
eventEmitter.emit('changeSiblingsData', msg)

// 3. 监听
eventEmitter.addListener('changeSiblingsData', (msg) => {
  this.setState({
    bus: msg
  })
  console.log(msg)
});
```



### 实现自己的发布订阅

```js

```



# 事件

```js
clickHandle = (event) => {
  console.log(event)
  console.log(event.target) // 触发事件的 -> button
  console.log(event.currentTarget) // 绑定事件的 -> button 假象
  console.log(event.nativeEvent.target) // 触发事件的 -> button
  console.log(event.nativeEvent.currentTarget) // 绑定事件的 -> document
}
```

​	event不是原生的，是合成事件，但是模拟出来DOM事件的所有能力。Vue的话就是原生的

**如何传参**

```js
onClick={e => this.handleClick('参数', e)}
```



# setState和batchUpdate

## 两种形式

​	参数是对象或者函数，函数如下

```js
this.setState((prevState, props) => {
  return {
    key: prevState.key // prevState 是上一次的 state，props 是此次更新被应用时的 props
  }
})
```



## 不可变值

​	主要是在操作数组和对象时需要注意，不能修改原来的

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



## state合并情况

​	其实就是等价于下面

```js
// 后面的数据会覆盖前面的更改，所以最终只加了一次.
Object.assign(
  previousState,
  {val: state.val + 1},
  {val: state.val + 1},
)
```

​	如果想基于当前的 `state` 来计算出新的值，则传递一个函数可以让你在函数内访问到当前的 `state` 值。

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

## 同步&异步情况

​	setTimeout和原生事件中同步



⚠️ state也可以写成静态属性，如下，类似的还有props

```js
class App ... {
  state = { // 注意不能写let之类，因为不是变量，是一个属性
  	name: 1
	},
  static defaultProps = { a: '1' }
}
```



# createRef

```js
// constructor
this.inputRef = React.createRef()

// render
<input ref={this.inputRef}>
```

​	这样`this.inputRef`就会有一个current属性，也就是对应的dom对象

​	它其实也可以获取组件实例，然后可以调用组件实例里面的方法

​	此外还可以通过函数参数的方法获取，如下

```js
// render
<input ref={input => {this.input = input}}>
// 这个参数里的input就是对应的当前这个input元素  
```









# 命题思路点拨

## React 组件设计模式

1. 什么是HoC（Higher-Order Component）？适用于什么场景？

2. 什么时候应该选择用class实现一个组件，什么时候应该用一个函数实现一个组件？

3. 你喜欢React Stateless组件吗？为什么？（发散题目 ）

## setState 深入

1. 当组件的setState函数被调用之后，会发生什么？

2. setState可以接受函数为参数吗？有什么作用？

## 事件系统

React事件机制是怎样的？为什么它要定义一套事件体系？

## React 底层原理

1. 为什么我们利用循环产生的组件中要用上key这个特殊的prop？

2. 什么是Fiber？是为了解决什么问题？











# 参考文献

- 幕客 - 解锁前端面试体系核心攻略 - 修言
  - **33 - 35 虚拟DOM和Fiber**





