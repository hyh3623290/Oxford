```js
class Greeting extends Component {
  constructor(props) {
    super(props)
    this.state = { count: props.intialCount } // 直接props还没用过
  }
  static defaultProps = {
    name: 132 // 这个name会被挂到props上
  }
}
```



# jsx

​	JavaScript的语法拓展，会被babel编译为`React.createElement`。而这个方法又会返回一个`ReactElement`对象

```js
export function createElement(type, config, children) {
  // propName 变量用于储存后面需要用到的元素属性
  let propName; 
  // props 变量用于储存元素属性的键值对集合
  const props = {}; 
  // key、ref、self、source 均为 React 元素的属性，此处不必深究
  let key = null;
  let ref = null; 
  let self = null; 
  let source = null;
  // config 对象中存储的是元素的属性
  if (config != null) { 
    // 进来之后做的第一件事，是依次对 ref、key、self 和 source 属性赋值
    if (hasValidRef(config)) {
      ref = config.ref;
   	}  
    // 此处将 key 值字符串化
    if (hasValidKey(config)) {
      key = '' + config.key; 
    }
    self = config.__self === undefined ? null : config.__self;
    source = config.__source === undefined ? null : config.__source;
    // 接着就是要把 config 里面的属性都一个一个挪到 props 这个之前声明好的对象里面
    for (propName in config) {
      if (
        // 筛选出可以提进 props 对象里的属性
        hasOwnProperty.call(config, propName) &&
        !RESERVED_PROPS.hasOwnProperty(propName) 
      ) {
        props[propName] = config[propName]; 
      }
    }
  }
  // childrenLength 指的是当前元素的子元素的个数，减去的 2 是 type 和 config 两个参数占用的长度
  const childrenLength = arguments.length - 2; 
  // 如果抛去type和config，就只剩下一个参数，一般意味着文本节点出现了
  if (childrenLength === 1) { 
    // 直接把这个参数的值赋给props.children
    props.children = children; 
    // 处理嵌套多个子元素的情况
  } else if (childrenLength > 1) { 
    // 声明一个子元素数组
    const childArray = Array(childrenLength); 
    // 把子元素推进数组里
    for (let i = 0; i < childrenLength; i++) { 
      childArray[i] = arguments[i + 2];
    }
    // 最后把这个数组赋值给props.children
    props.children = childArray; 
  } 
  // 处理 defaultProps
  if (type && type.defaultProps) {
    const defaultProps = type.defaultProps;
    for (propName in defaultProps) { 
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }
  // 最后返回一个调用ReactElement执行方法，并传入刚才处理过的参数
  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
}
```

​	这里会调用ReactElement方法

```js
const ReactElement = function(type, key, ref, self, source, owner, props) 
  const element = {
    // REACT_ELEMENT_TYPE是一个常量，用来标识该对象是一个ReactElement
    $$typeof: REACT_ELEMENT_TYPE,
    // 内置属性赋值
    type: type,
    key: key,
    ref: ref,
    props: props,
    // 记录创造该元素的组件
    _owner: owner,
  };
  // 
  if (__DEV__) {
    // 这里是一些针对 __DEV__ 环境下的处理，对于大家理解主要逻辑意义不大，此处我直接省略掉，以免混淆视听
  }
  return element;
};
```

​	可以看出这个函数其实就是将参数按照一定的规范做了一个简单的组装，创建，并返回给了`React.createElement`方法。这个时候返回的是虚拟DOM，而真实DOM的渲染是通过ReactDOM.render方法进行的

```js
ReactDOM.render(
    // 需要渲染的元素（ReactElement）
    element, 
    // 元素挂载的目标容器（一个真实DOM）
    container,
    // 回调函数，可选参数，可以用来处理渲染结束后的逻辑
    [callback]
)

ReactDOM.render(
  "新年好", // #1
  document.querySelector("#root"),
  () => {
    console.log("渲染完回调")
  }
)
```

如何区分组件还是标签呢，组件名必须是大写



**style**

```jsx
const styleData = { fontSize: '30px', color: 'red' }
<p style={styleData}></p>

<p style={{ fontSize: '30px', color: 'red' }}></p>

<p style="font-size: 30px;"></p>
// 静态style也非常好写，就跟html的一样
```

**原生html**

```jsx
const rawHtml = <div>123</div>
const rawHtmlData = {
  __html: rawHtml
}

<div dangeriousSetInnerHtml={rawHtmlData}></div>
// 必须要这么写
```

受控组件 - 表单

```jsx
<input value={name1} onChange={this.handleInputChange}/>

<textarea value={name2} onChange={this.handleTextAreaChange}/>

<select value={city} onChange={this.handleSelectChange}>
  <option value="sz">深圳</option>
  <option value="bj">北京</option>
  <option value="sh">上海</option>
</select>

male
<input 
  type="radio" 
  name="gender" 
  value="male" 
  checked={gender === 'male'} 
  onChange={this.handleRadioChange} />
      
female
<input 
  type="radio" 
  name="gender" 
  value="female" 
  checked={gender === 'female'} 
  onChange={this.handleRadioChange} />

<input type="checkbox" checked={flag} onChange={this.handleCheckboxChange}/>
    
// ---- 事件 ----
  handleInputChange = (e) => {
    this.setState({
      name1: e.target.value
    })
  }

  handleTextAreaChange = (e) => {
    this.setState({
      name2: e.target.value
    })
  }

  handleSelectChange = (e) => {
    this.setState({
      city: e.target.value
    })
  }
  
  handleRadioChange = (e) => {
    this.setState({
      gender: e.target.value
    })
  }
  
  handleCheckboxChange = () => {
    this.setState({
      flag: !this.state.flag
    })
  }
```

# key

​	组件每次更新时，会生成一个虚拟dom和原有的虚拟dom进行对比。

​	如果是批量生成一组元素，那就会根据key值进行对比，比如key为1的和key为1的进行对比，需要给它一个唯一标识。如果是index的话对性能不好，因为index是会变的，比如你删除了第一个，那其他的也都会变了，就会全部重新渲染一遍。所以如果列表中发生顺序等操作变化，key一定要用数据的id



# setState

​	**什么时候可以使用setState**：`willMount didMount willReceive didUpdate`。尤其是willUpdate和shouldUpdate，render不能用

​	首先 `setState` 的调用并不会马上引起 `state` 的改变，并且如果你一次调用了多个 `setState` ，那么结果可能并不如你期待的一样。`setState` 异步的原因我认为在于，`setState` 可能会导致 DOM 的重绘，如果调用一次就马上去进行重绘，那么调用多次就会造成不必要的性能损失。设计成异步的话，就可以将多次调用放入一个队列中，在恰当的时候统一进行更新过程。此外setTimeout里同步，自己定义的dom事件同步。

​	多次调用会合并为一次，只有当更新结束后 `state` 才会改变，三次调用等同于如下代码

```js
Object.assign(  
  {},
  { count: this.state.count + 1 },
  { count: this.state.count + 1 },
  { count: this.state.count + 1 },
)
```

​	当然你也可以通过以下方式来实现调用三次 `setState` 使得 `count` 为 3

```js
handle() {
  this.setState((prevState) => ({ count: prevState.count + 1 }))
  this.setState((prevState) => ({ count: prevState.count + 1 }))
  this.setState((prevState) => ({ count: prevState.count + 1 }))
}
```

​	如果想立即拿到setState最新值用回调函数

**不可变值：**如何操作数组和对象呢

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



**如何拿到最新的state呢**

1. 传一个函数

```js
this.setState((state,props) => { this.setState(count: state.count + 1) })
// 这样就不合并了
```

2. setTimeout

```js
setTimeout(() => {
  console.log(this.state.count)
}, 0)
```

3. 原生事件中修改状态

```js
componentDidMount() {
  document.body.addEventListener('click', this.changeValue, false)
}
changeValue = () => {
  this.setState({count: this.state.count + 1})
  console.log(this.state.count)
}
```

**setState主流程**

​	setState后，newState会被存入一个pending队列，然后判断是否处于batch update这个机制中，是则保存组件于dirtyComponent中。否则遍历所有的dirtyCompoennts，调用updateComponent，然后去执行更新。所以setTimeout的时候被判断没有在那个batchUpdate中，就走了同步的方式去更新

​	首先在class里的所有函数，进入的时候都会遇到一个变量叫isBatchingUpdates，刚开始为true，函数执行完就赋值为false

​	dirtyCompoennt就是说现在state已经被更新的组件

注意：

​	当state直接改变值会怎么样呢 `count: ++this.state.count`，左右两边的count一样了，所以在SCU里不好判断。react为了避免有些人的这种不规范的写法，它就没有默认自己在SCU里做比较，如果默认自己比较的话就会有问题了。所以SCU必须要配合不可变值来写

# 生命周期

## componentWillReceiveProps

​	并不是由props变化引起的，而是只要父组件更新就会触发



## shouldComponentUpdate

​		如果子组件有4个li，要动了其中的1个，那么很遗憾，另外三个也会重新渲染

```js
shouldComponentUpdate(nextProps, nextState) {
  return nextProps.label !== this.props.label
}
// _.isEqual(nextProps.list, this.props.list)
// lodash可以做深度比较，一次递归到底，耗费性能
```

​	`PureComponent`为我们自动添加了这个方法。组件将要渲染之前会经过这个东西。我们可以做一个判断，如果props没有变化，我们就不渲染这个组件了。（帮我们节省了状态和属性不变的节点）。在SCU中它会对state和props做一个浅比较，判断是否更新，浅比较适用大部分情况，尽量不要深度比较，写state的时候别写太深。深度比较的话一次递归到底，耗费性能。Vue的defineProperty也是递归到底的，所以data也不要太深（性能优化）



## render

​	组件初始化的时候render方法的作用是生成虚拟dom，然后虚拟dom到真实dom是通过`ReactDOM.render`方法实现。当更新的时候同样会在render方法中生成新的虚拟dom，然后借助diff算法定位出两个虚拟dom的差异，从而针对发生变化的真实dom做定向更新。





 	⚠️**注意：**16.3 版本时，**只有父组件的更新**会触发getDerivedStateFromProps。在 React 16.4 中，**任何因素触发的组件更新流程**（包括由 this.setState 和 forceUpdate 触发的更新流程）都会触发 

```js
// -------- 16.3 之前 ---------
// 1.挂载
constructor
componentWillMount
render
componentDidMount

// 2.1 更新 - 父组件更新引起
componentWillReceiveProps(nextProps) // 可以通过this.props拿到旧props
shouldComponentUpdate(nextProps, nextState) // #2
componentWillUpdate(nextProps, nextState)
render
componentDidUpdate(prevProps, prevState)
// 2.2 更新 - 组件自身更新
shouldComponentUpdate
componentWillUpdate
render
componentDidUpdate

// 3. 卸载
componentWillUnmount

// -------- 16.4 之后 ----------
// 1. 挂载

```

**总结**

​	首先是constructor，做一些初始化操作，初始化state，绑定this之类。注意 render 在执行过程中并不会去操作真实 DOM（也就是说不会渲染），它的职能是把需要渲染的内容返回出来。真实 DOM 的渲染工作，在挂载阶段是由 ReactDOM.render 来承接的。componentDidMount 方法在渲染结束后被触发，此时因为真实 DOM 已经挂载到了页面上，我们可以在这个生命周期里执行真实 DOM 相关的操作。此外，类似于异步请求、数据初始化这样的操作也大可以放在这个生命周期来做。

​	之后就是组件的更新了，分为两种：一种是由父组件更新触发的更新；另一种是组件自身调用自己的 setState 触发的更新。前者会触发receiveProps，可以拿到最新的props值。父组件的重新渲染会导致其所有子组件的重新渲染。	

​	setState，父层组件的render方法被运行，forceUpdate方法被调用会触发组件更新（新版旧版都是）

​	新的生命周期把Will的全都干掉：`willMount，willUpdate，willReceivesProps`

​	只要跟props有关就有getDerivedStateFromProps，如挂载和更新时

​	首先16版本引入fiber机制，在一定程度上影响了部分生命周期的调用，这也是引入新的两个API的原因。

​	对于一些像动画之类的实时性很高的东西，也就是16ms必须渲染一次以保证不卡顿，React会每16ms（以内）暂停一下更新，返回来继续渲染动画。对于这种异步渲染形式主要有两个阶段：`reconciliation`和`commit`。前者可以打断，后者不行，会一直更新界面直到完成。

​	之前，当拥有一个复杂组件时，改动了最上层的state后，函数的调用栈会特别长，再加上里面的一些复杂操作，就可能长时间阻塞主线程，带来不好的用户体验。fiber就是为了解决该问题。

​	对于一些像动画之类的实时性很高的东西，也就是16ms必须渲染一次以保证不卡顿，React会每16ms（以内）暂停一下更新，返回来继续渲染动画。对于这种异步渲染形式主要有两个阶段：`reconciliation`和`commit`。前者可以打断，后者不行，会一直更新界面直到完成。

​	因为 Reconciliation 阶段是可以被打断的，所以 Reconciliation 阶段会执行的生命周期函数就可能会出现调用多次的情况，从而引起 Bug。由此对于 Reconciliation 阶段调用的几个函数，除了 `shouldComponentUpdate` 以外，其他都应该避免去使用，并且 V16 中也引入了新的 API 来解决这个问题

​	`getDerivedStateFromProps` 用于替换 `componentWillReceiveProps` ，该函数会在初始化和 `update` 时被调用

```js
class ExampleComponent extends React.Component {
  state = {};

  static getDerivedStateFromProps(nextProps, prevState) {
    if (prevState.someMirroredValue !== nextProps.someValue) {
      return {
        derivedData: computeDerivedState(nextProps),
        someMirroredValue: nextProps.someValue
      };
    }
    return null;
  }

  // dom渲染前保存了快照，以便后续使用, 拿到的都是更新前的状态，更新后的拿这个有什么用
  getSnapShotBeforeUpdate(prevProps, prevState) {
    return kk
  }

  componentDidUpdate(prevProps, prevState, snapShot) {
    // 第三个参数时上面返回的结果
  }
}
```

​	`getSnapshotBeforeUpdate` 用于替换 `componentWillUpdate` ，该函数会在 `update` 后 DOM 更新前被调用，用于读取最新的 DOM 数据。

# 数据通信

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



# 高阶组件

## 高阶组件

```js
class wrapComp = (Comp) => {
  class NewComp {
    // balabala a = 1
    render() {
      return <Comp a={a}/>
    }
  }
  return NewComp
}
let newCmp = wrapComp(Cmp) 
// 其实就是我接收一个组件，然后在里面新写一个组件
// 新组件返回Cmp，最后再返回新组件
```

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



## **组件的插槽**

```jsx
ReactDOM.createPortal(
  <span className="red">123</span>，
  document.body
)
// red这个css不影响
```

## **Suspense**

​	可以包异步的组件和异步的数据，一灯的没看懂

```jsx
import React, {Suspense, lazy} from "react"

const LazyCmp = lazy(() => { import("./cmp") })

render() {
  return (
    <Suspense fallback={<span>loading</span>}>
      <LazyCmp />
    </Suspense>
  )
}
```

## **memo**

​	将函数组件转换为纯组件（对函数组件做一个缓存）useMemo -> memo（不重新渲染是不是就是缓存了）

​	就是将函数组件转换成类似于PureComponent，只有props改变的时候才会重新渲染

```js
memo(func)
// 也可以接收第二个参数来写我们自己的判断方法
// 或者 export default React.memo(func)
```

## ref

​	必须操作dom时使用，如file文件，必须通过input拿到

```jsx
// Context的Provider会影响redux吗，会的。
constructor() {
  this.ref = React.createRef() // 实际创建了个Symbol，这样绝对不会重复
}
  ...
<Cmp ref={this.ref}></Cmp>
```

```jsx
import React, {Component} from "react";

const TargetComponent = React.forwardRef((props, ref) => (
  <input type="text" ref={ref}/>
))

export default class Comp extends Component {
  constructor(props) {
    super(props)
    this.ref = React.createRef()
  }

  componentDidMount() {
    this.ref.current.name = '转发ref成功🍺'
    console.log(this.ref)
    // 我可以在父组件中拿到子组件的ref
    // 就是说这个ref还可以传递
  }

  render() {
    return (
      <TargetComponent ref={this.ref}/>
    )
  }
}
```



## 事件机制

​	JSX 上写的事件并没有绑定在对应的真实 DOM 上，而是通过事件代理的方式，将所有的事件都统一绑定在了 `document` 上。这样的方式不仅减少了内存消耗，还能在组件挂载销毁时统一订阅和移除事件

​	另外冒泡到 `document` 上的事件也不是原生浏览器事件，而是 React 自己实现的合成事件（SyntheticEvent）。因此我们如果不想要事件冒泡的话，调用 `event.stopPropagation` 是无效的，而应该调用 `event.preventDefault`。

​	那么实现合成事件的目的是什么呢？总的来说在我看来好处有两点，分别是

- 合成事件首先抹平了浏览器之间的兼容问题，另外这是一个跨浏览器原生事件包装器，赋予了跨浏览器开发的能力
- 对于原生浏览器事件来说，浏览器会给监听器创建一个事件对象。如果你有很多的事件监听，那么就需要分配很多的事件对象，造成高额的内存分配问题。但是对于合成事件来说，有一个事件池专门来管理它们的创建和销毁，当事件需要被使用时，就会从池子中复用对象，事件回调结束后，就会销毁事件对象上的属性，从而便于下次复用事件对象。

**⚠️注意：**想获取原生的event可以通过`event.nativeEvent`。target：触发事件的元素，如button。currentTarget：绑定事件的元素，如document

**合成事件过程**

​	比如事件已经挂载完了，现在div触发事件，最终都会冒泡到document上。document会生成一个统一的react event。也就是合成事件的对象，然后它会再去派发事件。在合成事件这一层，它会再去派发事件，派发事件就是说我们能通过target事件知道谁触发了对吧，比如一个a标签，它所在的组件是哪一个我们就知道了，然后这个组件里面到底哪个绑定了onClick事件通过这种方式我们已经知道了。所以说通过判断它的target，判断target所在的组件，及它的事件绑定关系。我们就能触发到所有绑定了这个click事件的函数，然后再把这个合成事件的event传递进来去执行这个函数

## immutable.js

彻底拥抱不可变值，基于共享数据（不是深拷贝），速度好

```js
const map1 = Immutable.Map({ a:1, b:2 })
const map2 = map1.set('b', 50) // 修改b
map1.get('b') // 2
map2.get('b') // 50
```

## 组件复合

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

​	一个 reducer 一定是一个纯函数，它可以有各种各样的内在逻辑，但它最终一定要返回一个 state：

```js
const reducer = (state, action) => {
    // 此处是各种样的 state处理逻辑
    return new_state
}
```

​	当我们基于某个 reducer 去创建 store 的时候，其实就是给这个 store 指定了一套更新规则：

**3. action 的作用是通知 reducer “让改变发生”**

**4. 派发 action，靠的是 dispatch**



# hooks

​	首先探讨类组件和函数组件的区别：类组件可以访问生命周期方法并且可以定义并维护 state（状态），而函数组件不可以；类组件需要继承 class，函数组件不需要；类组件中可以获取到实例化后的 this，并基于这个 this 做各种各样的事情，而函数组件不可以；

​	函数没有自身的状态，相同的props输入必然会获得相同的组件展示（纯），一旦遇到这样的特性， 函数如果要是纯的话第一个是对组件进行缓存。react也应用了这个特性提供了一些钩子可以对组件应用上缓存 

​	函数组件会捕获 render 内部的状态，这是两类组件最大的不同。（没看懂）

​	**函数组件更加契合 React 框架的设计理念**。何出此言？不要忘了这个赫赫有名的 React 公式：

​	`UI = render(data)`

​	Hooks 的本质：一套能够使函数组件更强大、更灵活的“钩子”

**useState**

```js
const [name, setName] = useState('Nicolas')
setName('hyh')
console.log(name) // hyh
```

**useEffect**

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

**useCount**

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



**为什么使用hooks**

- 不需要复杂的生命周期和this。

- 解决业务逻辑难以拆分的问题，

​     以前的话业务逻辑会拆到不同的生命周期函数里去，往往在一个稍微成规模的 React 项目中，一个生命周期不止做一件事情.像这样的生命周期函数，它的体积过于庞大，做的事情过于复杂，会给阅读和维护者带来很多麻烦。而在 Hooks 的帮助下，我们完全可以把这些繁杂的操作**按照逻辑上的关联拆分进不同的函数组件里：\**我们可以有专门管理订阅的函数组件、专门处理 DOM 的函数组件、专门获取数据的函数组件等。Hooks 能够帮助我们\**实现业务逻辑的聚合，避免复杂的组件和冗余的代码**。

- 使状态逻辑复用变得简单可行；

​	过去我们复用状态逻辑，靠的是 HOC（高阶组件）和 Render Props 这些组件设计模式，这是因为 React 在原生层面并没有为我们提供相关的途径。但这些设计模式并非万能，它们在实现逻辑复用的同时，也破坏着组件的结构，其中一个最常见的问题就是“嵌套地狱”现象。

​	Hooks 可以视作是 React 为解决状态逻辑复用这个问题所提供的一个原生途径。现在我们可以通过自定义 Hook，达到既不破坏组件结构、又能够实现逻辑复用的效果。

- 函数组件从设计思想上来看，更加契合 React 的理念。



**关于 Hooks 的局限性**，目前社区鲜少有人讨论。这里我想结合团队开发过程当中遇到的一些瓶颈，和你分享实践中的几点感受：

​	**Hooks 暂时还不能完全地为函数组件补齐类组件的能力**：比如 getSnapshotBeforeUpdate、componentDidCatch 这些生命周期，目前都还是强依赖类组件的。官方虽然立了“会尽早把它们加进来”的 Flag，但是说真的，这个 Flag 真的立了蛮久了……（扶额）

​	**“轻量”几乎是函数组件的基因，这可能会使它不能够很好地消化“复杂”**：我们有时会在类组件中见到一些方法非常繁多的实例，如果用函数组件来解决相同的问题，业务逻辑的拆分和组织会是一个很大的挑战。我个人的感觉是，从头到尾都在“过于复杂”和“过度拆分”之间摇摆不定，哈哈。耦合和内聚的边界，有时候真的很难把握，**函数组件给了我们一定程度的自由，却也对开发者的水平提出了更高的要求**。

​	**Hooks 在使用层面有着严格的规则约束**：这也是我们下个课时要重点讲的内容。对于如今的 React 开发者来说，如果不能牢记并践行 Hooks 的使用原则，如果对 Hooks 的关键原理没有扎实的把握，很容易把自己的 React 项目搞成大型车祸现场。





React 团队面向开发者给出了两条 **React-Hooks 的使用原则**，原则的内容如下：

- 只在 React 函数中调用 Hook；

- 不要在循环、条件或嵌套函数中调用 Hook

​    其实，原则 2 中强调的所有“不要”，都是在指向同一个目的，那就是要确保 Hooks 在每次渲染时都保持同样的执行顺序。代码是没错的，我们调用的是 setName那么它修改的状态也应该是 name，而不是 career。那为什么最后发生变化的竟然是 career 呢？年轻人，不如我们一起来看一看 Hooks 的实现机制吧





# Fiber

​	Fiber 会使原本同步的渲染过程变成异步的。以前的更新过程是同步递归操作的，同步渲染一旦开始，会牢牢抓住主线程不放，直到递归彻底完成。在这个过程中，浏览器没有办法处理任何渲染之外的事情，会进入一种无法处理用户交互的状态。因此若渲染时间稍微长一点，页面就会面临卡顿甚至卡死的风险。（也就是16ms内还在执行逻辑运算）

​	而 React 16 引入的 Fiber 架构，恰好能够解决掉这个风险：Fiber 会将一个大的更新任务拆解为许多个小任务。每当执行完一个小任务时，渲染线程都会把主线程交回去，看看有没有优先级更高的工作要处理，确保不会出现其他任务被“饿死”的情况，进而避免同步渲染带来的卡顿。在这个过程中，渲染线程不再“一去不回头”，而是可以被打断的，这就是所谓的“异步渲染”（假设我有个动画再跑，16ms一帧，但是它其实只需要6ms，我就可以在剩的10ms做其他的事情）

​	而 React 16 引入的 Fiber 架构，恰好能够解决掉这个风险：Fiber 会将一个大的更新任务拆解为许多个小任务。每当执行完一个小任务时，渲染线程都会把主线程交回去，看看有没有优先级更高的工作要处理，确保不会出现其他任务被“饿死”的情况，进而避免同步渲染带来的卡顿。在这个过程中，渲染线程不再“一去不回头”，而是可以被打断的，这就是所谓的“异步渲染”

​	Fiber 架构的重要特征就是可以被打断的异步渲染模式。但这个“打断”是有原则的，根据“能否被打断”这一标准，React 16 的生命周期被划分为了 render 和 commit 两个阶段，而 commit 阶段又被细分为了 pre-commit 和 commit。

- render 阶段：纯净且没有副作用，可能会被 React 暂停、终止或重新启动。
  - constructor - getDerived - shouldComp - render

- pre-commit 阶段：可以读取 DOM。
  - getSnap

- commit 阶段：可以使用 DOM，运行副作用，安排更新。
  - didMount - didUpdate - willUnMount

​    总的来说，render 阶段在执行过程中允许被打断，而 commit 阶段则总是同步执行的。

​    为什么这样设计呢？简单来说，由于 render 阶段的操作对用户来说其实是“不可见”的，所以就算打断再重启，对用户来说也是零感知。而 commit 阶段的操作则涉及真实 DOM 的渲染，再狂的框架也不敢在用户眼皮子底下胡乱更改视图，所以这个过程必须用同步渲染来求稳。





# React & Vue

1. React用JSX拥抱js，Vue使用模板拥抱html

   开发体验的话Vue更像html的结构，上面是一个模板，然后中间是js代码，下面是style。React的话全是以点js结尾的，写Vue的话是以点Vue结尾的，所以写react就是写js，只不过jsx就是个语法糖而已，本质就是createElement函数。

2. react是函数式编程，Vue是声明式编程

   Vue变量得先声明好，你去修改data的值，我去监听，每个操作都是声明式的操作，data.a=100之类。React每一次修改都是setState修改

3. react更多需要自力更生，Vue把想要的都给你

​    jsx较简洁，vue的话还有动态表达式：jsx的话变量全是大括号，字符串的话就是一个双引号。就感觉react的动态静态的逻辑非常简单，只要动态就是大括号，静态就是字符串。还有jsx直接if，else就可以了，vue钟的话还v-if之类的



# SPA

缺点：首次进入处理慢，不利于SEO（因为所有东西都是写在js里的，搜索引擎爬虫没有办法读js里的代码）

优点：更好的用户体验（减少请求和渲染和页面跳转产生的等待和空白），页面切换快

单页面应用的原理

```js
<a href="#view1"> // #1
<a href="#view2">
<a href="#view3">

function gethash() {
  console.log(window.location.hash)
}
window.addEventListener("hashchange", getHash) // #2
```

1. 点击以后，url会产生一个变化，但是页面不会重新去刷新，或者说重新去渲染

   url的变化不会直接发生http请求

2. 这样就可以获取到页面当前的hash值，获取到hash值之后再去给他匹配一些不同的视图







----------------