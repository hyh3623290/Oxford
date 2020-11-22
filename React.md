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

# 生命周期

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















​    

----------------