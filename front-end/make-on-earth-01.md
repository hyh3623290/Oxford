# React

## 类组件

​	类组件比函数组件好在哪呢，有完备的生命周期。以后类组件会完全没有的

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

​	`PureComponent`为我们自动添加了shouldComponentUpdate这个方法。组件将要渲染之前会经过这个东西。我们可以做一个判断，如果props没有变化，我们就不渲染这个组件了。（帮我们节省了状态和属性不变的节点）

​	如果子组件有4个li，要动了其中的1个，那么很遗憾，另外三个也会重新渲染

```js
shouldComponentUpdate(nextProps, nextState) {
  return nextProps.label !== this.props.label
}
```

​	子组件的更新触发父组件。父组件更新全部子组件要diff，一旦更改了state类型，上面方法全部歇菜

## 函数组件

​	没有自身的状态，相同的props输入必然会获得相同的组件展示（纯），一旦遇到这样的特性， 函数如果要是纯的话第一个是对组件进行缓存。react也应用了这个特性提供了一些钩子可以对组件应用上缓存 

## 高阶组件

​	就是我接收一个组件，我做了一些通用的处理后返回这个组件并给它传参数，扔什么返什么，如果反的其他的叫回调函数。

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

**实战**

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

## 组件的插槽

```jsx
ReactDOM.createPortal(
  <span>123</span>，
  document.body
)
```

## Suspense

可以包异步的组件和异步的数据，一灯的没看懂

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



## memo

​	将函数组件转换为纯组件（对函数组件做一个缓存）useMemo -> memo（不重新渲染是不是就是缓存了）

​	就是将函数组件转换成类似于PureComponent，只有props改变的时候才会重新渲染

```js
memo(func)
// 也可以接收第二个参数来写我们自己的判断方法
// 或者 export default React.memo(func)
```

## ref

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

## 生命周期

### 16.3之前

```js
// 挂载
constructor
componentWillMount
render
componentDidMount

// 更新 - 父组件更新引起
componentWillReceiveProps(nextProps) // 可以通过this.props拿到旧props
shouldComponentUpdate(nextProps, nextState) // #2
componentWillUpdate(nextProps, nextState)
render
componentDidUpdate(prevProps, prevState)
// 更新 - 组件自身更新
shouldComponentUpdate
componentWillUpdate
render
componentDidUpdate

// 卸载
componentWillUnmount
```

​	首先是constructor，做一些初始化操作，初始化state，绑定this之类。注意 render 在执行过程中并不会去操作真实 DOM（也就是说不会渲染），它的职能是把需要渲染的内容返回出来。真实 DOM 的渲染工作，在挂载阶段是由 ReactDOM.render 来承接的。componentDidMount 方法在渲染结束后被触发，此时因为真实 DOM 已经挂载到了页面上，我们可以在这个生命周期里执行真实 DOM 相关的操作。此外，类似于异步请求、数据初始化这样的操作也大可以放在这个生命周期来做。

​	之后就是组件的更新了，分为两种：一种是由父组件更新触发的更新；另一种是组件自身调用自己的 setState 触发的更新。前者会触发receiveProps，可以拿到最新的props值。父组件的重新渲染会导致其所有子组件的重新渲染。	

​	setState，父层组件的render方法被运行，forceUpdate方法被调用会触发组件更新（新版旧版都是）

### 16.4之后

**创建时**

- constructor - getDerived - render - didMount

**更新时**

- newProps - getDerived - shouldComponentUpdate - render - getSnapShot
- setState - shouldComponentUpdate - render - getSnapShot

​    在 React 16.4 中，任何因素触发的组件更新流程（包括由 this.setState 和 forceUpdate 触发的更新流程）都会触发 getDerivedStateFromProps；

​    而在 v 16.3 版本时，只有父组件的更新会触发该生命周期。

​	新的生命周期把Will的全都干掉：`willMount，willUpdate，willReceivesProps`

​	只要跟props有关就有getDerivedStateFromProps，如挂载和更新时

​	首先16版本引入fiber机制，在一定程度上影响了部分生命周期的调用，这也是引入新的两个API的原因。

​	之前，当拥有一个复杂组件时，改动了最上层的state后，函数的调用栈会特别长，再加上里面的一些复杂操作，就可能长时间阻塞主线程，带来不好的用户体验。fiber就是为了解决该问题。

​	对于一些像动画之类的实时性很高的东西，也就是16ms必须渲染一次以保证不卡顿，React会每16ms（以内）暂停一下更新，返回来继续渲染动画。对于这种异步渲染形式主要有两个阶段：`reconciliation`和`commit`。前者可以打断，后者不行，会一直更新界面直到完成。

**Reconciliation阶段**

- `componentWillMount`
- `componentWillReceiveProps`
- `shouldComponentUpdate`
- `componentWillUpdate`

**Commit** 阶段

- `componentDidMount`
- `componentDidUpdate`
- `componentWillUnmount`

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

## React setState

​	**什么时候可以使用setState**：`willMount didMount willReceive didUpdate`

​	尤其是willUpdate和shouldUpdate，render不能用

首先 `setState` 的调用并不会马上引起 `state` 的改变，并且如果你一次调用了多个 `setState` ，那么结果可能并不如你期待的一样。

​	`setState` 异步的原因我认为在于，`setState` 可能会导致 DOM 的重绘，如果调用一次就马上去进行重绘，那么调用多次就会造成不必要的性能损失。设计成异步的话，就可以将多次调用放入一个队列中，在恰当的时候统一进行更新过程。

​	多次调用会合并为一次，只有当更新结束后 `state` 才会改变，三次调用等同于如下代码

```js
Object.assign(  
  {},
  { count: this.state.count + 1 },
  { count: this.state.count + 1 },
  { count: this.state.count + 1 },
)
```

当然你也可以通过以下方式来实现调用三次 `setState` 使得 `count` 为 3

```js
handle() {
  this.setState((prevState) => ({ count: prevState.count + 1 }))
  this.setState((prevState) => ({ count: prevState.count + 1 }))
  this.setState((prevState) => ({ count: prevState.count + 1 }))
}
```

如果想立即拿到setState最新值用回调函数

## 组件的通信

### 父子通信

​	父组件通过 `props` 传递数据给子组件，子组件通过调用父组件传来的函数传递数据给父组件，这两种方式是最常用的父子通信实现办法。

​	这种父子通信方式也就是典型的单向数据流，父组件通过 `props` 传递数据，子组件不能直接修改 `props`， 而是必须通过调用父组件函数的方式告知父组件修改数据。

### 兄弟组件通信

​	对于这种情况可以通过共同的父组件来管理状态和事件函数。比如说其中一个兄弟组件调用父组件传递过来的事件函数修改父组件中的状态，然后父组件将状态传递给另一个兄弟组件。

### 跨多层次组件通信

​	如果你使用 16.3 以上版本的话，对于这种情况可以使用 Context API

### 任意组件

​	这种方式可以通过 Redux 或者 Event Bus 解决，另外如果你不怕麻烦的话，可以使用这种方式解决上述所有的通信情况。发布订阅模式



## 事件机制

​	JSX 上写的事件并没有绑定在对应的真实 DOM 上，而是通过事件代理的方式，将所有的事件都统一绑定在了 `document` 上。这样的方式不仅减少了内存消耗，还能在组件挂载销毁时统一订阅和移除事件

​	另外冒泡到 `document` 上的事件也不是原生浏览器事件，而是 React 自己实现的合成事件（SyntheticEvent）。因此我们如果不想要事件冒泡的话，调用 `event.stopPropagation` 是无效的，而应该调用 `event.preventDefault`。

​	那么实现合成事件的目的是什么呢？总的来说在我看来好处有两点，分别是

- 合成事件首先抹平了浏览器之间的兼容问题，另外这是一个跨浏览器原生事件包装器，赋予了跨浏览器开发的能力
- 对于原生浏览器事件来说，浏览器会给监听器创建一个事件对象。如果你有很多的事件监听，那么就需要分配很多的事件对象，造成高额的内存分配问题。但是对于合成事件来说，有一个事件池专门来管理它们的创建和销毁，当事件需要被使用时，就会从池子中复用对象，事件回调结束后，就会销毁事件对象上的属性，从而便于下次复用事件对象。

## hooks

### useState

```js
const [name, setName] = useState('Nicolas')
setName('hyh')
console.log(name) // hyh
```

### useEffect

为我们合并了didMount，didUpdate，willUnMount

```js
useEffect(() => {
  console.log('didMount & didUpdate')
})

useEffect(() => {
  return () => {
    console.log('unMount')
  }
})

useEffect(() => {
  console.log('count变化触发')
}, [count])

useEffect(() => {
  console.log('didMount')
}, [])
```

### useCount

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

### useRef

### useMemo

​	它如果返回一个函数就等于useCallback

# JavaScript数据类型

## 原始类型

​	在 JS 中，存在着 6 种原始值，分别是：

- `boolean`

- `null`

- `undefined`

- `number`

- `string`

- `symbol`

  原始类型存储的都是值，是没有函数可以调用的

  

## 对象类型

​	原始类型存储的是值，对象类型存储的是地址（指针）。当你创建了一个对象类型的时候，计算机会在内存中帮我们开辟一个空间来存放值，但是我们需要找到这个空间，这个空间会拥有一个地址（指针）。

```js
const a = []
```

​	对于常量 `a` 来说，假设内存地址（指针）为 `#001`，那么在地址 `#001` 的位置存放了值 `[]`，常量 `a` 存放了地址（指针） `#001`，再看以下代码

```js
const a = []
const b = a
b.push(1)
```

​	当我们将变量赋值给另外一个变量时，复制的是原本变量的地址（指针），也就是说当前变量 `b` 存放的地址（指针）也是 `#001`，当我们进行数据修改的时候，就会修改存放在地址（指针） `#001` 上的值，也就导致了两个变量的值都发生了改变。

​	接下来我们来看函数参数是对象的情况

```js
function test(person) {
  person.age = 26
  person = {
    name: 'yyy',
    age: 30
  }

  return person
}
const p1 = {
  name: 'yck',
  age: 25
}
const p2 = test(p1)
console.log(p1) // -> ?
console.log(p2) // -> ?
```

- 首先，函数传参是传递对象指针的副本
- 到函数内部修改参数的属性这步，我相信大家都知道，当前 `p1` 的值也被修改了
- 但是当我们重新为 `person` 分配了一个对象时就出现了分歧，请看下图

p1        ------> address: #001 ------> value

person ------> address: #001 ------> value

重新为person分配对象后

p1        ------> address: #001 ------> value

person ------> address: #002 ------> value

所以最后 `person` 拥有了一个新的地址（指针），也就和 `p1` 没有任何关系了，导致了最终两个变量的值是不相同的。

# typeof和InstanceOf

​	`typeof` 对于原始类型来说，除了 `null` 都可以显示正确的类型

​	`typeof` 对于对象来说，除了函数都会显示 `object`，函数时`function`所以说 `typeof` 并不能准确判断变量到底是什么类型

​	如果我们想判断一个对象的正确类型，这时候可以考虑使用 `instanceof`，因为内部机制是通过原型链来判断的	

```js
p1 instanceof Person // true
```

# this和闭包以及深浅拷贝

- 对于直接调用 `foo` 来说，不管 `foo` 函数被放在了什么地方，`this` 一定是 `window`
- 对于 `obj.foo()` 来说，我们只需要记住，谁调用了函数，谁就是 `this`，所以在这个场景下 `foo` 函数中的 `this` 就是 `obj` 对象
- 对于 `new` 的方式来说，`this` 被永远绑定在了 `c` 上面，不会被任何方式改变 `this`

说完了以上几种情况，其实很多代码中的 `this` 应该就没什么问题了，下面让我们看看箭头函数中的 `this`

```js
function a() {
  return () => {
    return () => {
      console.log(this)
    }
  }
}
console.log(a()()())
```

​	首先箭头函数其实是没有 `this` 的，箭头函数中的 `this` 只取决包裹箭头函数的第一个普通函数的 `this`。在这个例子中，因为包裹箭头函数的第一个普通函数是 `a`，所以此时的 `this` 是 `window`。另外对箭头函数使用 `bind` 这类函数是无效的。

​	最后种情况也就是 `bind` 这些改变上下文的 API 了，对于这些函数来说，`this` 取决于第一个参数，如果第一个参数为空，那么就是 `window`。

​	那么说到 `bind`，不知道大家是否考虑过，如果对一个函数进行多次 `bind`，那么上下文会是什么呢？

```js
let a = {}
let fn = function () { console.log(this) }
fn.bind().bind(a)() // => ?
```

​	其实不管我们给函数 `bind` 几次，`fn` 中的 `this` 永远由第一次 `bind` 决定，所以结果永远是 `window`

​	以上就是 `this` 的规则了，但是可能会发生多个规则同时出现的情况，这时候不同的规则之间会根据优先级最高的来决定 `this` 最终指向哪里。

​	首先，`new` 的方式优先级最高，接下来是 `bind` 这些函数，然后是 `obj.foo()` 这种调用方式，最后是 `foo` 这种调用方式，同时，箭头函数的 `this` 一旦被绑定，就不会再被任何方式所改变。

**闭包**

​	闭包的定义其实很简单：函数 A 内部有一个函数 B，函数 B 可以访问到函数 A 中的变量，那么函数 B 就是闭包。在 JS 中，闭包存在的意义就是让我们可以间接访问函数内部的变量

**浅拷贝**

​	我们了解了对象类型在赋值的过程中其实是复制了地址，从而会导致改变了一方其他也都被改变的情况。通常在开发中我们不希望出现这样的问题，我们可以使用浅拷贝来解决这个情况。

​	首先可以通过 `Object.assign` 来解决这个问题

```js
let a = {
  age: 1
}
let b = Object.assign({}, a)
a.age = 2
console.log(b.age) // 1
```

​	另外我们还可以通过展开运算符 `...` 来实现浅拷贝

```js
let a = {
  age: 1
}
let b = { ...a }
a.age = 2
console.log(b.age) // 1
```

​	通常浅拷贝就能解决大部分问题了，但是当我们遇到如下情况就可能需要使用到深拷贝了

```js
let a = {
  age: 1,
  jobs: {
    first: 'FE'
  }
}
let b = { ...a }
a.jobs.first = 'native'
console.log(b.jobs.first) // native
```

​	浅拷贝只解决了第一层的问题，如果接下去的值中还有对象的话，那么就又回到最开始的话题了，两者享有相同的地址。要解决这个问题，我们就得使用深拷贝了

**深拷贝**

​	这个问题通常可以通过 `JSON.parse(JSON.stringify(object))` 来解决。

​	但是该方法也是有局限性的：

- 会忽略 `undefined`
- 会忽略 `symbol`
- 不能序列化函数（忽略）
- 不能解决循环引用的对象

​    lodash深拷贝函数

# 原型及原型链

​	可以得出一个结论：对于 `obj` 来说，可以通过 `__proto__` 找到一个原型对象，在该对象中定义了很多函数让我们来使用。原型的 `constructor` 属性指向构造函数，构造函数又通过 `prototype` 属性指回原型，但是并不是所有函数都具有这个属性，`Function.prototype.bind()` 就没有这个属性。

```js
// 1. 所有的引用类型（对象，函数，数组），都具有对象特性，即可自由扩展属性（除null）
var obj={}; obj.a=100;
var array=[]; array.a=100
function fn(){}; fn.a=100;

// 2. 所有的引用类型（对象，函数，数组）都有一个__proto__属性，属性值是一个普通的对象
obj.__proto__;  array.__proto__;    fn.__proto__;

// 3. 所有的函数类型都有一个prototype属性，属性值也是一个普通的对象
fn.prototype()

// 4. 所有的引用类型（对象，函数，数组），__proto__属性值指向它的构造函数的prototype
obj.__proto__ === Object.prototype

// 5. 当试图得到一个对象的属性时，如果这个对象本身没有这个属性，就会到它的__proto__中去找
//    即它的构造函数
function Person(name,age){
    this.name=name
}
Person.prototype.printName=function(){
    console.log(this.name)
}
var p = new Person()
p.alertName = function(){ alert(this.name) }
p.alertName()
p.printName()
p.toString() // p.__proto__.__proto__有toString方法
// f.hasOwnProperty(item) => 只包含自身的属性，不包含原型上的属性
```



# Event Loop

​	当我们执行 JS 代码的时候其实就是往执行栈中放入函数，那么遇到异步代码的时候该怎么办？其实当遇到异步的代码时，会被**挂起**并在需要执行的时候加入到 Task（有多种 Task） 队列中。一旦执行栈为空，Event Loop 就会从 Task 队列中拿出需要执行的代码并放入执行栈中执行，所以本质上来说 JS 中的异步还是同步行为。

​	不同的任务源会被分配到不同的 Task 队列中，任务源可以分为 **微任务**（microtask） 和 **宏任务**（macrotask）。在 ES6 规范中，microtask 称为 `jobs`，macrotask 称为 `task`。下面来看以下代码的执行顺序

```js
console.log('script start')

async function async1() {
  await async2()
  console.log('async1 end')
}
async function async2() {
  console.log('async2 end')
}
async1()

setTimeout(function() {
  console.log('setTimeout')
}, 0)

new Promise(resolve => {
  console.log('Promise')
  resolve()
})
  .then(function() {
    console.log('promise1')
  })
  .then(function() {
    console.log('promise2')
  })

console.log('script end')
// script start => async2 end => 
// Promise => script end => promise1 => promise2 => 
// async1 end => setTimeout
// 同步 > promise > setTimeout
```

# 事件机制

**事件触发有三个阶段：**

- `window` 往事件触发处传播，遇到注册的捕获事件会触发
- 传播到事件触发处时触发注册的事件
- 从事件触发处往 `window` 传播，遇到注册的冒泡事件会触发

**注册事件：**

​	通常我们使用 `addEventListener` 注册事件，该函数的第三个参数可以是布尔值，也可以是对象。对于布尔值 `useCapture` 参数来说，该参数默认值为 `false` ，`useCapture` 决定了注册的事件是捕获事件还是冒泡事件。对于对象参数来说，可以使用以下几个属性

- `capture`：布尔值，和 `useCapture` 作用一样
- `once`：布尔值，值为 `true` 表示该回调只会调用一次，调用后会移除监听
- `passive`：布尔值，表示永远不会调用 `preventDefault`

​    一般来说，如果我们只希望事件只触发在目标上，这时候可以使用 `stopPropagation` 来阻止事件的进一步传播。通常我们认为 `stopPropagation` 是用来阻止事件冒泡的，其实该函数也可以阻止捕获事件。`stopImmediatePropagation` 同样也能实现阻止事件，但是还能阻止该事件目标执行别的注册事件。

```js
node.addEventListener(
  'click',
  event => {
    event.stopImmediatePropagation()
    console.log('冒泡')
  },
  false
)
// 点击 node 只会执行上面的函数，该函数不会执行
node.addEventListener(
  'click',
  event => {
    console.log('捕获 ')
  },
  true
)
```

**事件代理：**

​	如果一个节点中的子节点是动态生成的，那么子节点需要注册事件的话应该注册在父节点上

```html
<ul id="ul">
	<li>1</li>
  <li>2</li>
	<li>3</li>
	<li>4</li>
	<li>5</li>
</ul>
<script>
	let ul = document.querySelector('#ul')
	ul.addEventListener('click', (event) => {
		console.log(event.target);
	})
</script>
```

事件代理的方式相较于直接给目标注册事件来说，有以下优点：

- 节省内存
- 不需要给子节点注销事件

# 跨域

​	因为浏览器出于安全考虑，有同源策略。也就是说，如果协议、域名或者端口有一个不同就是跨域，Ajax 请求会失败。

​	那么是出于什么安全考虑才会引入这种机制呢？ 其实主要是用来防止 CSRF 攻击的。简单点说，CSRF 攻击是利用用户的登录态发起恶意请求。

​	也就是说，没有同源策略的情况下，A 网站可以被任意其他来源的 Ajax 访问到内容。如果你当前 A 网站还存在登录态，那么对方就可以通过 Ajax 获得你的任何信息。当然跨域并不能完全阻止 CSRF。

​	然后我们来考虑一个问题，请求跨域了，那么请求到底发出去没有？ 请求必然是发出去了，但是浏览器拦截了响应。你可能会疑问明明通过表单的方式可以发起跨域请求，为什么 Ajax 就不会。因为归根结底，跨域是为了阻止用户读取到另一个域名下的内容，Ajax 可以获取响应，浏览器认为这不安全，所以拦截了响应。但是表单并不会获取新的内容，所以可以发起跨域请求。同时也说明了跨域并不能完全阻止 CSRF，因为请求毕竟是发出去了。

## JSONP

​	JSONP 的原理很简单，就是利用 `<script>` 标签没有跨域限制的漏洞。通过 `<script>` 标签指向一个需要访问的地址并提供一个回调函数来接收数据当需要通讯时。

```html
<script src="http://domain/api?param1=a&param2=b&callback=jsonp"></script>
<script>
    function jsonp(data) {
    	console.log(data)
	}
</script>
```

​	JSONP 使用简单且兼容性不错，但是只限于 `get` 请求。

​	在开发中可能会遇到多个 JSONP 请求的回调函数名是相同的，这时候就需要自己封装一个 JSONP，以下是简单实现

## CORS

​	CORS 需要浏览器和后端同时支持。IE 8 和 9 需要通过 `XDomainRequest` 来实现。

​	浏览器会自动进行 CORS 通信，实现 CORS 通信的关键是后端。只要后端实现了 CORS，就实现了跨域。

​	服务端设置 `Access-Control-Allow-Origin` 就可以开启 CORS。 该属性表示哪些域名可以访问资源，如果设置通配符则表示所有网站都可以访问资源。

​	虽然设置 CORS 和前端没什么关系，但是通过这种方式解决跨域问题的话，会在发送请求时出现两种情况，分别为简单请求和复杂请求。

以 Ajax 为例，当满足以下条件时，会触发简单请求

1. 使用下列方法之一：
   - `GET`
   - `HEAD`
   - `POST`
2. `Content-Type` 的值仅限于下列三者之一：
   - `text/plain`
   - `multipart/form-data`
   - `application/x-www-form-urlencoded`

​    请求中的任意 `XMLHttpRequestUpload` 对象均没有注册任何事件监听器； `XMLHttpRequestUpload` 对象可以使用 `XMLHttpRequest.upload` 属性访问。

**复杂请求**

​	那么很显然，不符合以上条件的请求就肯定是复杂请求了。

​	对于复杂请求来说，首先会发起一个预检请求，该请求是 `option` 方法的，通过该请求来知道服务端是否允许跨域请求。

​	对于预检请求来说，如果你使用过 Node 来设置 CORS 的话，可能会遇到过这么一个坑。

​	以下以 express 框架举例：

```js
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*')
  res.header('Access-Control-Allow-Methods', 'PUT, GET, POST, DELETE, OPTIONS')
  res.header(
    'Access-Control-Allow-Headers',
    'Origin, X-Requested-With, Content-Type, Accept, Authorization, Access-Control-Allow-Credentials'
  )
  next()
})
```

​	该请求会验证你的 `Authorization` 字段，没有的话就会报错。

​	当前端发起了复杂请求后，你会发现就算你代码是正确的，返回结果也永远是报错的。因为预检请求也会进入回调中，也会触发 `next` 方法，因为预检请求并不包含 `Authorization` 字段，所以服务端会报错。

​	想解决这个问题很简单，只需要在回调中过滤 `option` 方法即可

```js
res.statusCode = 204
res.setHeader('Content-Length', '0')
res.end()
```

**document.domain**

​	该方式只能用于**二级域名相同**的情况下，比如 `a.test.com` 和 `b.test.com` 适用于该方式。

​	只需要给页面添加 `document.domain = 'test.com'` 表示二级域名都相同就可以实现跨

​	这种方式通常用于获取嵌入页面中的第三方页面数据。一个页面发送消息，另一个页面判断来源并接收消息

```js
// 发送消息端
window.parent.postMessage('message', 'http://test.com')
// 接收消息端
var mc = new MessageChannel()
mc.addEventListener('message', event => {
  var origin = event.origin || event.originalEvent.origin
  if (origin === 'http://test.com') {
    console.log('验证通过')
  }
})
```

# 面向对象

​	首先它是一个编程思想吧，js本身就是基于面向对象构建出来的。比如说，js里面有很多内置类...像Promise就是ES6新增的内置类，我们一般都是基于new Promise来创建一个实例来管理异步编程。还有用的Vue，React也都是基于面向对象的，平常开发的时候也都是创建他们的实例来进行操作（new Vue，class extends component等等）。

  不过js的面向对象跟其他语言还不太一样，如重载，一般的语言是多个方法方法名相同，参数个数或者类型不同来实现，但是js的话相同的函数名后者就覆盖了前者，不过我们可以写一个方法，在方法里面判断参数类型来实现重载

  面向对象三大概念：重载，重写和继承。重写就是子类可以重写父类的方法，继承就是子类继承父类的属性和方法。

# 存储

 cookie

​	一般服务器生成，可以设置过期时间，4K，每次都会携带在 header 中，对于请求性能影响

indexDB

​	除非被清理，否则一直存在，存储无限

# 常考算法题

## 位运算

​	位运算在算法中很有用，速度可以比四则运算快很多。

**左移**

```js
10 << 1 // -> 20
```

​	左移就是将二进制全部往左移动，`10` 在二进制中表示为 `1010` ，左移一位后变成 `10100` ，转换为十进制也就是 20，所以基本可以把左移（a左移b位）看成以下公式 `a * (2 ^ b)`

**算数右移**

```js
10 >> 1 // -> 5
```

​	算数右移就是将二进制全部往右移动并去除多余的右边，`10` 在二进制中表示为 `1010` ，右移一位后变成 `101` ，转换为十进制也就是 5，所以基本可以把右移看成以下公式 `int v = a / (2 ^ b)`

右移很好用，比如可以用在二分算法中取中间值

```js
13 >> 1 // -> 6
```

​	总结是左移一位相当于乘2，右移相当于除以2

**按位与**

每一位都为 1，结果才为 1

```js
8 & 7 // -> 0
// 1000 & 0111 -> 0000 -> 0
```

**按位或**

其中一位为 1，结果就是 1

```js
8 | 7 // -> 15
// 1000 | 0111 -> 1111 -> 15
```

**按位异或**

每一位都不同，结果才为 1

```js
8 ^ 7 // -> 15
8 ^ 8 // -> 0
// 1000 ^ 0111 -> 1111 -> 15
// 1000 ^ 1000 -> 0000 -> 0
```

从以上代码中可以发现按位异或就是不进位加法

**面试题**：两个数不使用四则运算得出和

这道题中可以按位异或，因为按位异或就是不进位加法，`8 ^ 8 = 0` 如果进位了，就是 16 了，所以我们只需要将两个数进行异或操作，然后进位。那么也就是说两个二进制都是 1 的位置，左边应该有一个进位 1，所以可以得出以下公式 `a + b = (a ^ b) + ((a & b) << 1)` ，然后通过迭代的方式模拟加法

```js
function sum(a, b) {
    if (a == 0) return b
    if (b == 0) return a
    let newA = a ^ b
    let newB = (a & b) << 1
    return sum(newA, newB)
}

// 10 ^ 10 = 00 10&10=100
```

## 排序

**冒泡排序**

​	冒泡排序的原理如下，从第一个元素开始，把当前元素和下一个索引元素进行比较。如果当前元素大，那么就交换位置，重复操作直到比较到最后一个元素，那么此时最后一个元素就是该数组中最大的数。下一轮重复以上操作，但是此时最后一个元素已经是最大数了，所以不需要再比较最后一个元素，只需要比较到 `length - 2` 的位置。

```js
function bubble(array) {
  checkArray(array);
  for (let i = array.length - 1; i > 0; i--) {
    // 从 0 到 `length - 1` 遍历
    for (let j = 0; j < i; j++) {
      if (array[j] > array[j + 1]) swap(array, j, j + 1)
    }
  }
  return array;
}
```

​	第一个和第二个比较大的放后面，第二个和第三个比较大的放后面。。。一轮后最后面的就是最大的

​	该算法的操作次数是一个等差数列 `n + (n - 1) + (n - 2) + 1` ，去掉常数项以后得出时间复杂度是 O(n * n)

**插入排序**

​	插入排序的原理如下。第一个元素默认是已排序元素，取出下一个元素和当前元素比较，如果当前元素大就交换位置。那么此时第一个元素就是当前的最小数，所以下次取出操作从第三个元素开始，向前对比，重复之前的操作。

以下是实现该算法的代码

```js
function insertion(array) {
  checkArray(array);
  for (let i = 1; i < array.length; i++) {
    for (let j = i - 1; j >= 0 && array[j] > array[j + 1]; j--)
      swap(array, j, j + 1);
  }
  return array;
}
```

该算法的操作次数是一个等差数列 `n + (n - 1) + (n - 2) + 1` ，去掉常数项以后得出时间复杂度是 O(n * n)

**选择排序**

​	选择排序的原理如下。遍历数组，设置最小值的索引为 0，如果取出的值比当前最小值小，就替换最小值索引，遍历完成后，将第一个元素和最小值索引上的值交换。如上操作后，第一个元素就是数组中的最小值，下次遍历就可以从索引 1 开始重复上述操作。

以下是实现该算法的代码

```js
function selection(array) {
  checkArray(array);
  for (let i = 0; i < array.length - 1; i++) {
    let minIndex = i;
    for (let j = i + 1; j < array.length; j++) {
      minIndex = array[j] < array[minIndex] ? j : minIndex;
    }
    swap(array, i, minIndex);
  }
  return array;
}
```

该算法的操作次数是一个等差数列 `n + (n - 1) + (n - 2) + 1` ，去掉常数项以后得出时间复杂度是 O(n * n)

# 数据结构

## 时间复杂度

​	在进入正题之前，我们先来了解下什么是时间复杂度。

​	通常使用最差的时间复杂度来衡量一个算法的好坏。

​	常数时间 O(1) 代表这个操作和数据量没关系，是一个固定时间的操作，比如说四则运算。

​	对于一个算法来说，可能会计算出操作次数为 aN + 1，N 代表数据量。那么该算法的时间复杂度就是 O(N)。因为我们在计算时间复杂度的时候，数据量通常是非常大的，这时候低阶项和常数项可以忽略不计。

​	当然可能会出现两个算法都是 O(N) 的时间复杂度，那么对比两个算法的好坏就要通过对比低阶项和常数项了。

## 栈

​	栈是一个线性结构，在计算机中是一个相当常见的数据结构。

​	栈的特点是只能在某一端添加或删除数据，遵循先进后出的原则

​	每种数据结构都可以用很多种方式来实现，其实可以把栈看成是数组的一个子集，所以这里使用数组来实现

```js
class Stack {
  constructor() {
    this.stack = []
  }
  push(item) {
    this.stack.push(item)
  }
  pop() {
    this.stack.pop()
  }
  peek() {
    return this.stack[this.getCount() - 1]
  }
  getCount() {
    return this.stack.length
  }
  isEmpty() {
    return this.getCount() === 0
  }
}
```



## 类

​	类虽然是函数，但是它不能提升，也会受块级作用域影响。类可以包含构造函数方法，实例方法，静态类方法，获取函数和设置函数，默认情况下，类定义内的代码都在严格模式下执行。

### 构造函数

​	constructor会告诉解释器在使用new操作符创建新实例时，应该调用这个函数。使用new调用类的构造函数会执行如下操作。

1. 在内存中创建一个新对象
2. 这个新对象内部的原型指针（`__proto__`）被赋值为构造函数的prototype属性
3. 构造函数内部的this被赋值为这个新对象（即this指向新对象）
4. 执行构造函数内部的代码（给新对象添加属性）
5. 如果构造函数返回非空对象，则返回该对象，否则返回刚创建的新对象

​    类实例化时传入的参数会作为构造函数的参数。constructor内是自有属性和方法，constructor外是原型属性和方法



### 静态类方法

​	可以在类上定义静态方法。这些方法通常用于执行不特定于实例的操作，也不要求存在类的实例，每个类上只能有一个静态成员。

```js
class Person {
  static sayName() {
    console.log('sayName')
  }
}
Person.name = 'my name' // ?
```



































































------------------------------------------------------------------

​	如果你对设计模式有所了解的话，就会知道，一个 API 并非越庞大越复杂才越优秀。或者说得更直接一点，庞大和复杂的 API 往往会带来维护层面的“灾难”。getDerivedStateFromProps







