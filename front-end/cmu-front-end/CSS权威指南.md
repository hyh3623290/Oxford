# for in

​	for in 是用来遍历对象的属性的，只要是非符号类型（Symbol）都会被遍历到

# for of

​	用来遍历可迭代对象的元素，对象类型不是可迭代的，一般用于数组

# console

`console.log(JSON.strinify(obj, null, 2))`输出格式great

# switch

​	switch会比较括号内的表达式和case中的表达式，如果是全等就会选择它。当满足了一个条件以后，只要没有遇到break，后面所有的语句都会执行

原来true === 1是false

# 手写BrowserRouter

​	BrowserRouter关键点就是直接渲染子元素，并且通过Context传入history及location给子路由，当location变化时通过setState来重新渲染子元素使页面刷新

```jsx
import React, {Component} from "react";
import {createBrowserHistory} from "history";
import {RouterContext} from "./RouterContext";

export default class BrowserRouter extends Component {
  static computeRootMatch(pathname) {
    return {
      path: "/",
      url: "/",
      params: {},
      isExact: pathname === "/"
    };
  }
  constructor(props) {
    super(props);
    this.history = createBrowserHistory();
    this.state = {
      location: this.history.location
    };
    this.unlisten = this.history.listen(location => {
      this.setState({location});
    });
  }
  componentWillUnmount() {
    if (this.unlisten) {
      this.unlisten();
    }
  }
  render() {
    return (
      <RouterContext.Provider
        value={{
          history: this.history,
          location: this.state.location,
          match: BrowserRouter.computeRootMatch(this.state.location.pathname)
        }}>
        {this.props.children}
      </RouterContext.Provider>
    );
  }
}
```

# 手写Link

​	主要是通过返回a元素，让href等于从props里拿到的to，但是a链接会刷新页面，所以给它防止默认事件，然后用history.push来代替。history需要从Consumer里取

```jsx
import React, {Component} from "react";
import {RouterContext} from "./RouterContext";

export default class Link extends Component {
  handleClick = (event, history) => {
    event.preventDefault();
    history.push(this.props.to);
  };
  render() {
    const {to, children} = this.props;
    return (
      <RouterContext.Consumer>
        {context => (
          <a
            href={to}
            onClick={event => this.handleClick(event, context.history)}>
            {children}
          </a>
        )}
      </RouterContext.Consumer>
    );
  }
}
```

# Route原理

​	它要拿到context里的location(表示当前的路径)，需要通过props里的path值判断是否与location匹配。不匹配的话要判断是否有children，

```jsx
{match
  ? children
    ? typeof children === "function"
      ? children(props)
      : children
    : component
      ? // ? React.cloneElement(element, props)
        React.createElement(component, props)
      : render
        ? render(props)
        : null
  : typeof children === "function"
    ? children(props)
    : null}
```

# Switch

​	遍历子元素，当匹配的时候就返回它

# Redirect

​	返回一个组件，当其挂载完成后通过history.push直接跳转即可, 该组件也可直接返回null，因为不需要它

# 虚拟dom

​	就是一个js对象，表示了当前的UI，同步的时候只去同步这个js对象，然后再根据差异性来更新真实的dom

​	React用jsx描述视图，将其通过babel-loader转为React.createElement的形式，就是将vdom转为真实dom

​	jsx本来就是js扩展，转义过程简单直接的多；vue把 template编译为render函数的过程需要复杂的编译器 转换字符串-ast-js函数字符串

```jsx
element = {
  type: "div",
  props: { // children和属性都在props里
    id: "A1",
    children: [
      {
        type: "div",
        props: {
          id: "B1",
        }
      },
      {
        type: "div",
        props: {
          id: "B2"
        }
      }
    ]
  }
}
```



# React原理

​	实现createElement，render方法

```jsx
import React from './hyh/kreact/kreact'
import ReactDOM from './hyh/kreact/ReactDOM'
import Component from './hyh/kreact/Component'

function FuncComponent(props) {
  return <h3>FuncComponent - {props.name}</h3>
}

class ClassComponent extends Component {
  render() {
    const { name } = this.props
    return (
      <h3>ClassComponent - {name}</h3>
    )
  }
}

const jsx = <div className="border">
              app
              <p>一段文本</p>
              <a href="http://www.baidu.com">开课吧</a>
              <FuncComponent name="函数组件" />
              <ClassComponent name="类组件 "/>
            </div>
ReactDOM.render(jsx, document.getElementById("root"))

/*
  首先babel见到jsx以后会将其转为createElement(type, props, children)形式，然后render调用的是
  createElement执行完后的结果
*/
```

kreact - 实现createElement - 其实就是转为vdom对象，之后render会将这个vdom转为真实dom

```js
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.map(child => {
        return typeof child === 'object' ? child : createTextNode(child)
      })
    }
  }
}

function createTextNode(text) {
  return {
    type: "TEXT",
    props: {
      children: [],
      nodeValue: text
    }
    
  }
}

export default {
  createElement
}
```

ReactDOM - 实现render方法 - 记得动态添加属性以及递归render子元素

```js
function render(vnode, container) { 
  const node = createNode(vnode)
  container.appendChild(node)
}

function createNode(vnode) {
  const { type, props  } = vnode
  let node
  if(type === "TEXT") {
    node = document.createTextNode('')
  } else if(typeof type === 'function'){
    node = type.isReactComponent ? updateClassComponent(vnode) : updateFunctionComponent(vnode)
  } else {
    node = document.createElement(type)
  }
  updateNode(node, props)
  reconcilerChilden(props.children, node)
  return node
}

function updateNode(node, nextValue) {
  Object.keys(nextValue).filter(k => k!=="children").forEach(k => {
    node[k] = nextValue[k]
  })
}

function reconcilerChilden(children, node) {
  for(let i=0; i<children.length; i++) {
    render(children[i], node)
  } 
}

function updateFunctionComponent(vnode) {
  const { type, props } = vnode
  const vvnode = type(props)
  const node = createNode(vvnode)
  return node
}

function updateClassComponent(vnode) {
  const { type, props } = vnode
  const cmp = new type(props)
  const vvnode = cmp.render()
  const node = createNode(vvnode)
  return node
}

export default {
  render
}
```

Component

```js
export default class Component {
  static isReactComponent = {}
  constructor(props) {
    this.props = props
  }
}
```

# diff

​	diff的核心就是尽可能多的复用原有节点 - 将一棵树转为另一棵树的最小操作数最好也就是O(n^3)，其中n是树中元素的数量。以下两个假设可以将其变为O(n)。

diff策略：

1. 同级比较
2. 拥有不同类型的两个组件将会生成不同的树形结构
3. 开发者可以通过不同的key，prop来暗示哪些子元素在不同的渲染下能保持稳定

diff操作一般有删除，更新，替换

# fiber

​	对于⼤型项⽬，组件树会很⼤，这个时候递归遍历的 成本就会很⾼，会造成主线程被持续占⽤，结果就是主线程上的布局、动画等周期性任务就⽆法⽴即得到处理，造成视觉上的卡顿，影响⽤户体验。

​	因为js是单线程，而且UI渲染和JS执行是互斥的。如果节点多，则层级特别深，所以页面会卡死，并且它也不会暂停

​	所以为了这个问题用fiber重写

## 帧

- 目前大多数设备的屏幕刷新率是60次/秒
- 当每秒绘制的帧数（FPS）达到60时，页面是流畅的，小于这个值时，用户会感觉卡顿。
- 每个帧的预算时间时16.66毫秒
- 每个帧的开头包括样式计算，布局和绘制
- JavaScript执行JavaScript引擎和页面渲染引擎在同一个渲染进程，GUI渲染和JavaScript执行两者是互斥的
- 如果某个任务执行时间过长，页面会推迟渲染

​    每一帧的生命周期会做很多事情（输入事件，JS执行，响应窗口拖动，布局，绘制等），如果执行完还不到16.66毫秒，则会剩余一些空余时间

以上说那么多意思就是每一帧会有些空余时间

## fiber

​	fiber有两个意义，第一个是执行单元，第二个是指数据结构

​	原来它是递归渲染，一次执行完，但我们用fiber可以把它拆成一个个小任务，然后可以暂停。前面说浏览器会剩余一些空余时间，我们可以这时候做一些render操作

- 执行单元

  每次执行完一个执行单元，React就会检查现在还剩多少时间，如果没有时间就将控制权交还给浏览器

- 数据结构

  使用链表，每个虚拟节点内部表示为一个Fiber

## requestAnimationFrame

​	回调函数会在绘制之前执行，都在浏览器刷新下一帧渲染周期的起点上

## requestIdleCallback

​	这个方法在浏览器空闲时调用。上一个函数的回调会在每一帧确定执行，属于高优先级任务，而这个的回调则不一定，属于低优先级任务。这个函数可以接收两个参数，一个是回调函数，另一个是时间限度。其中回调函数可以接受一个deadline参数，deadline.timeRemaining可以获取当前帧的剩余时间。另一个时间限度表示如果达到这个时间限度，不管你有没有空，都要给我执行。

高层次的思考能力建立在低层次的自动化基础之上

```js
function workLoop(deadline) {
  
}

requestIdleCallback(workLoop) {
  
}
```



# Generator

```js
let a = 0
function *generatorTest() {
  yield (a = 1 + 1)
}
let t = generatorTest() // 返回的是一个遍历器对象
console.log(a) // 0
t.next()
console.log(a) // 2
```

只有调用了next才会遍历下一个内部状态。惰性求值

# 原始值和引用值

​	保存原始值的变量是**按值访问**的，引用值是保存在内存中的对象，而JavaScript是不允许直接访问内存的，所以在操作对象时**访问的是该对象的引用**。

​	函数的参数都是按值传递的，当传递对象的时候可能会让人感觉迷惑

```js
function setName(obj) {
  obj.name = 10
}

let person = new Object()
setName(person)

console.log(person.name) // 10
```

​	这里虽然person的name值变化了，但其实还是按值传递的，因为即使对象是按值传进函数的，obj也会通过引用访问对象。以下可以证明

```js
function setName(obj) {
  obj.name = 10
  obj = new Object()
  obj.name = 20
}

let person = new Object()
setName(person)

console.log(person.name) // 10
```

这里如果是按引用传递的，那么person.name应该是20才对，但其实还是10

**总结：**按值传递就是传个副本，按引用传递指的是传内存空间的地址，即改变一个值会影响另外一个值。而函数的参数传递的是一个副本，所以说它是按值传递的。虽然按值传递，但它传引用自然会操作引用，但你发现给引用重新赋值后并没有影响原来的值，说明它还是保存的一个副本。

# 垃圾回收

​	规则很简单，确定哪个变量不会再被使用，然后释放它占用的资源。比如函数中的局部变量。

​	JavaScript最常用的垃圾回收策略是**标记清理**。当垃圾回收程序开始执行时，会将内存中所有的变量都加上一个标记，之后会将所有在上下文中的变量，以及被这些变量引用的变量的标记去掉。随后垃圾回收程序做一次内存清理，销毁带标记的所有值。这个过程是周期性的执行。

​	因为垃圾回收程序会周期性的执行，所以如果内存中分配了很多的变量，则可能造成性能损失，因此垃圾回收的时间调度很重要。在某些浏览器中是有可能主动触发垃圾回收的，如ie和opera7以上window.CollectGarbage

**内存管理**

​	如果数据不再被需要，应把它设置为null，从而释放其引用。解除引用的关键是在于确保相关的值已经不在上下文里了，因此它在下次垃圾回收时会被回收。

​	通过let和const声明提升性能，因为这有助于改进垃圾回收的进程，因为它们都是块级作用域，所以可能会更早地让垃圾回收程序介入，在块级作用域比函数作用域更早终止的情况下，这就有可能发生。

# redux-saga

不同于redux-thunk，不会再遇到回调地狱的，因此更擅长解决复杂异步场景

saga有三个任务：

1. 调用异步操作；call，fork
2. 状态更新（dispatch）；put
3. 做监听；take，takeEvery

```js
import { call, put, take, takeEvery } from 'redux/effects'

// 先创建一个saga watcher-saga
function *loginSaga(params) {
   yield takeEvery('loginSaga', loginHandle) //（监听的action，对应的处理函数）
}

// worker-saga
function *loginHandle(action) {
  console.log(action) // {type:'loginSaga', payload: "xiaoming"}
  yield put({ type: 'LOGIN_REQUEST' })
  try {
    const res1 = yield call(LoginService.login, action.payload) // 调接口
    const res2 = yield call(LoginService.getMoreUserInfo, res1)
    yield put({ type: 'LOGIN_SUCCESS', payload: {...res1, ...res2} })
  } catch (err) {
    yield put({ type: 'LOGIN_FAILURE', payload: err })
  }
}

export default loginSaga
```

要让里面的具体内容跑起来就要去创建一下 

```js
import {createStore, combineReducers, applyMiddleware} from "redux";
import {userReducer} from "./userReducer";
import createSagaMiddleware from "redux-saga"; // 1
import loginSaga from "../action/loginSaga";

const sagaMiddleware = createSagaMiddleware(); // 2
const store = createStore(
  combineReducers({user: userReducer}),
  applyMiddleware(sagaMiddleware) // 3
);

sagaMiddleware.run(loginSaga); // 4

export default store;
```



call和fork的区别：call是阻塞调用，一般用于顺序执行。一个是非阻塞，相当于放到后台去执行了，类似setState这种不会立马拿到state值

```js
const a = yield call(...)
console.log(a) // 可以拿到值

const a = yield fork(...)
console.log(a) // 不能拿到值
```

# umi

启动项目

yarn create umi - yarn - yarn start

```js
umi g page about
umi g page more/index --less
```

动态路由

```js
umi g page product/'$id'

// config.js
{
  path: '/product/:id',
  component: './product/$id'
}
```







-------------------------------