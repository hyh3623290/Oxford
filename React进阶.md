# Form

​	⽤Form.create()的⽅式实现： 我希望可以通过getFieldDecorator与input进行绑定，方便以后用getFieldsValue之类拿到各个表单的值，并且能够用validateFields进行校验

- getFieldDecorator： ⽤于和表单进⾏双向绑定

  思路：首先肯定要把表单组件传进来，然后需要传一个name值来标示它，此外克隆它并给它传入value值和onChange事件，存入高阶组件的state中，把子组件的数据拿到了其实就简单了。

- getFieldsValue：获取⼀组输⼊控件的值，如不传⼊参数，则获 取全部组件的值 

- getFieldValue： 获取⼀个输⼊控件的值 

- validateFields：校验并获取⼀组输⼊域的值与 Error，若 fieldNames 参数为空，则校验全部组件

## 效果

```jsx
import React, {Component} from "react";
import kFormCreate from "../components/kFormCreate";

const nameRules = {required: true, message: "please input ur name"};
const passwordRules = {required: true, message: "please input ur password"};

@kFormCreate
class MyFormPage extends Component {
  submit = () => {
    const {getFieldsValue, getFieldValue, validateFields} = this.props;
    validateFields((err, values) => {
      if (err) {
        console.log("err", err); //sy-log
      } else {
        console.log("success", values); //sy-log
      }
    });
    console.log("submit", getFieldsValue(), getFieldValue("password"));
  };
  render() {
    console.log("props", this.props); //sy-log
    const {getFieldDecorator} = this.props;
    return (
      <div>
        <h3>MyFormPage</h3>
        {getFieldDecorator("name", {rules: [nameRules]})(
          <input type="text" placeholder="please input ur name" />
        )}
        {getFieldDecorator("password", {rules: [passwordRules]})(
          <input type="password" placeholder="please input ur password" />
        )}
        <button onClick={this.submit}>提交</button>
      </div>
    );
  }
}

export default MyFormPage;

```

首先App是一个表单组件，但这个组件比较弱智，既不能收集信息也不能校验，所以我们要对它进行扩充，我们使用KFormCreate。它接收这个表单组件，并返回一个新的class组件，这个class组件return这个表单组件。



## kFormCreate

```jsx
import React, {Component} from "react";

export default function kFormCreate(Cmp) {
  return class extends Component {
    constructor(props) {
      super(props);
      this.state = {};
      this.options = {};
    }

    handleChange = e => {
      let {name, value} = e.target;
      this.setState({[name]: value});
    };

    getFieldDecorator = (field, option) => {
      this.options[field] = option;
      return InputCmp => {
        //克隆一份
        return React.cloneElement(InputCmp, {
          name: field,
          value: this.state[field] || "",
          onChange: this.handleChange
        });
      };
    };

    getFieldsValue = () => {
      return {...this.state};
    };

    getFieldValue = field => {
      return this.state[field];
    };
    
    validateFields = callback => {
      // 校验错误信息
      const errors = {};
      const state = {...this.state};
      for (let name in this.options) {
        if (state[name] === undefined) {
          // 没有输入，判断为不合法
          errors[name] = "error";
        }
      }
      if (JSON.stringify(errors) === "{}") {
        // 合法
        callback(undefined, state);
      } else {
        callback(errors, state);
      }
    };
    render() {
      return (
        <div className="border">
          <Cmp
            getFieldDecorator={this.getFieldDecorator}
            getFieldsValue={this.getFieldsValue}
            getFieldValue={this.getFieldValue}
            validateFields={this.validateFields}
          />
        </div>
      );
    }
  };
}
```



# 弹窗组件

```jsx
import React, {Component} from "react";
import {createPortal} from "react-dom";

export default class Dialog extends Component {
  constructor(props) {
    super(props);
    const doc = window.document;
    this.node = doc.createElement("div");
    doc.body.appendChild(this.node);
  }

  componentWillUnmount() {
    window.document.body.removeChild(this.node);
  }

  render() {
    return createPortal(
      <div className="dialog">
        <h3>Dialog</h3>
        {this.props.children}
      </div>,
      this.node
    );
  }
}

```

# 

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

# 虚拟Dom

​    操作真实 DOM 会引起重排和重绘，会影响到性能，因此要尽量减少 DOM 操作。其次DOM的数据结构也非常复杂。对于 DOM 的任何操作都映射为内存中对象的操作，在必要时才重新渲染页面。

# Diff算法

​	找不同的过程，就是调和过程Reconciliation。

## 三个假设

1. 相同的组件有着相同的 DOM 结构，不同的组件有着不同的 DOM 结构。
2. 位于同一层次的一组子节点，它们之间可以通过唯一的 id 进行区分
3. DOM 结构中，跨层级的节点操作非常少，可以忽略不计

## 过程

​	首先逐层对比，当对比两棵树时，`diff` 算法会优先比较两棵树的根节点，如果它们的类型不同，比如说之前是 div，现在变成 p 了，那么就认为这两棵树完全不同，这是两个完全不同的组件。因此也没有必要再往下再比对子节点了，直接把 div 删掉，重建为 p。也就是卸载旧组件、挂载新组件。若根节点类型相同，React 才会认为“你没变，你还是那个组件”。接下来，在保留这个组件的基础上，检查其属性的变化，然后根据属性变化的情况去更新组件。

​	数组动态生成的组件，可以用key

【练习 1】你觉得 Virtual DOM 好在哪

​	以前使用模板的时候

```php+HTML
<table>
  {% students.forEach(function(student){ %}
  <tr>
    <td>{% student.name %}</td>
    <td>{% student.age %}</td>
    </tr>
  {% }); %}
</table>
```

​	模板会帮我们做什么呢？它会把你的 students 这个数据源读进去，塞到上面这段 template 代码里，把它们融合在一起，吐出一段目标 HTML 给你。然后这段 HTML 代码就可以直接被拿去渲染到页面上，成为 DOM。

​	这个过程差不多是这样：

```js
// 数据和模板融合出 HTML 代码
var targetDOM = template({data: students})
// 添加到页面中去
document.body.appendChild(targetDOM)
```

​	本来我只是想改 Maria 的名字，现在整个表格都需要被重新渲染。DOM 操作的范围，从小小的一个表格字段位，扩大到了整个表格。这不合理。

​	相比之下，虚拟 DOM 方案每次只更新必要的 DOM，虽然它增加了 diff 过程。但增加的是 js 计算，换来的可是 DOM 开销，这可是杠杆撬地球一般的操作。。。所以说，这种情况下，我们会认为虚拟 DOM 从性能上来讲会更合适。（并不是比直接操作 DOM 更快）。不用操心 DOM 细节

⚠️ 以下是其他资料

基于以上三个假设，React 分别对 tree diff、component diff 以及 element diff 进行算法优化。

## tree diff

​	由于跨节点层级的移动操作特别少到可以 忽略不计，针对这一点，React 通过对两个树的同一层的节点进行比较，当发现节点已经不存在时，则该节点及其子节点会被完全删除掉，不会用于进一步的比较。这样只需要对树进行一次遍历，便能完成整个 DOM 树的比较。所以，官方建议不要进行DOM节点的跨层级操作，因为它会整个移除并重新插入。在实现自己的组件时，保持稳定的 DOM 结构会有助于性能的提升。例如，我们有时可以通过 CSS 隐藏或显示某些节点，而不是真的移除或添加 DOM 节点。

## component diff

React 是基于组件构建应用的，对于组件间的比较所采取的策略也是简洁高效。

1. 如果是同一类型的组件，按照原策略继续比较虚拟 DOM Tree ；
2. 如果不是，则将该组件判断为 dirty component，从而替换整个组件下的所有子节点；
3. 对于同一类型的组件，有可能其虚拟 DOM 没有任何变化，如果能够确切的知道这点那可以节省大量的 diff 运算时间，因此 React 允许用户通过 shouldComponentUpdate() 来判断该组件是否需要进行 diff 

## element diff

​	当节点处于同一层级（比如`ul`下面一组`li`）时，React Diff 提供了三种节点操作，分别为：INSERT_MARKUP（插入）、MOVE_EXISTING（移动）和 REMOVE_NODE（删除）

1. **INSERT_MARKUP**：新的 component 类型不在老集合里， 即是全新的节点，需要对新节点执行插入操作；
2. **MOVE_EXISTING**：在老集合有新 component 类型，且 element 是可更新的类型，generateComponentChildren 已调用 receiveComponent，这种情况下 prevChild=nextChild，就需要做移动操作，可以复用以前的 DOM 节点；
3. **REMOVE_NODE**：老 component 类型，在新集合里也有，但对应的 element 不同则不能直接复用和更新，需要执行删除操作，或者老 component 不在新集合里的，也需要执行删除操作

​    有种情况：老集合中包含节点：A、B、C、D，更新后的新集合中包含节点：B、A、D、C，两个集合的顺序发生了一些 改变。此时新集合和老集进行 diff 差异化对比时，发现 B != A，则创建并插入 B 至新集合，删除老集合 A；以此类推，创建并插入 A、D 和 C，删除 B、C 和 D。

​    针对这一现象，React 提出优化策略：允许开发者对同一层级的同组子节点，添加唯一 key 进行区分，虽然只是小小的改动，性能上却发生了翻天覆地的变化！

⚠️16 - 那么，如此高效的 diff 到底是如何运作的呢？让我们通过源码进行详细分析。

# Fiber

​	Fiber 是 React16 引入的一种新的调和引擎。

​	由于调和阶段的生命周期逻辑是单纯的 js 计算，所有的工作都在内存里进行，不涉及真实 DOM 操作，也就是说你打断执行也好、重复执行也罢，用户都是不感知的，只要你最后能 commit 出正确的 DOM 更新就好。这里硬要说的话，有一个编码层面的坑需要注意一下：在调和阶段的生命周期里，不要尝试写入一些你期望它只执行一次的逻辑，它可不保证到底会给你执行多少次。

​	JS单线程 - 如果前面有一个任务长期霸占 CPU ，那后面的什么事情都干不了 - 浏览器会呈现卡死的状态 - 对于我们前端框架而言，解决这种问题通常有三个方向：

1. 优化每个任务，提高它的运行速度，挤压 CPU 运算量；
2. 快速响应用户，让用户觉得快，不阻塞用户的交互；
3. 尝试 Worker 多线程。

​    Vue 选择了第一种，而 React 选择了第二种

​	React V15 在渲染时，会递归比对 VirtualDOM 树，找出需要变动的节点，然后同步更新它们， 一气呵成。这个过程 React 称为 Reconcilation 。在 Reconcilation 期间， React 会占据浏览器资源，会导致用户触发的事件得不到响应，并且会导致掉帧，导致用户感觉到卡顿。

​	为了给用户制造一种应用很快的“假象”，我们不能让一个任务长期霸占着资源。 你可以将浏览器的渲染、布局、绘制、资源加载(例如 HTML 解析)、事件响应、脚本执行视作操作系统的“进程”，我们需要通过某些调度策略合理地分配 CPU 资源，从而提高浏览器的用户响应速率, 同时兼顾任务执行效率。

​	所以 React 通过Fiber 架构，让自己的 Reconcilation 过程变成可被中断。“适时”地让出 CPU 执行权，除了可以让浏览器及时地响应用户的交互，还有其他好处:

1. 与其一次性操作大量 DOM 节点相比, 分批延时对DOM进行操作，可以得到更好的用户体验；
2. 给浏览器一点喘息的机会，他会对代码进行编译优化（JIT）及进行热代码优化，或者对 reflow 进行修正。

​    渲染的过程可以被中断，可以将控制权交回浏览器，让位给高优先级的任务，浏览器空闲后再恢复渲染。

​	通常屏幕的帧率为60Hz，也就是每秒60张画面。所以每一张画面的时间为1000/60 = 16ms，所以我们的代码执行实现少于16ms，就能看到流畅的画面。

⚠️ 17

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

为什么这样设计呢？简单来说，由于 render 阶段的操作对用户来说其实是“不可见”的，所以就算打断再重启，对用户来说也是零感知。而 commit 阶段的操作则涉及真实 DOM 的渲染，再狂的框架也不敢在用户眼皮子底下胡乱更改视图，所以这个过程必须用同步渲染来求稳。

# React原理

​	虚拟DOM到真实DOM的同步过程，成为协调。

**初始化**

```jsx
import React from './components/KReact'
import ReactDOM from './components/KReact/ReactDOM'

const jsx = <div className="border">app</div>

ReactDOM.render(jsx, document.getElementById('root'))
```

KReact.js

```js
function createElement(type, props, children) {}

export default {
  createElement
}
```

KReactDOM.js

```js
function render(vnode, container) {}

export default {
  render
}
```

​	这样就不报错了。接下来实现细节了

​	**createElement**接收参数(babel会自动调用这个函数并且把参数都放到对应的位置上)，返回虚拟dom - vnode，return出来的值就给ReactDOM.render函数了,为了方便把children全部传下去，要给children加...，同时为了建立树状图，需要把children放到props里,因为children只有一个文本的时候与其他节点的格式不太像，所以转换一下createTextNode

```js
function createElement(type, props, ...children) {
  return {
    type: type,
    props: {
      ...props,
      children: children.map(child => 
        typeof child === 'object' ? child : createTextNode(child)
      )
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
```

​	**render**就是把vnode变成node，并把node更新到container。第一件事是生成一个node（createNode）根据谁来创建呢，我们的节点类型很多，如文本节点，html节点，（class，function）肯定是不同的函数来进行处理。通过type值就可以判断节点类型。第一步只能处理最外层，如果要处理children则必须递归，children是一个数组，写个reconcilerChildren。我们要先遍历它的children,对每个子元素再做一次render。现在可以渲染出节点了，但是还没有塞内容，如className等。这些内容都在props里，遍历props里的值把它放到node里我们写一个函数叫updateNode做这件事。 现在处理function类型组件，

```js
function render(vnode, container) {
  const node = createNode(vnode)
  container.appendChild(node )
}

function createNode(vnode) {
  const { type, props } = vnode
  let node
  if(type === "TEXT") {
    node = createTextNode()
  } else if(typeof type === 'function') {
    node = type.isReactComponent ? updateClassComponent : updateFunctionComponent(vnode)
  } else if(type){
    node = document.createElement(type)
  } else {
    node = document.createDocumentFragment()
  }

  updateNode(node, props)
  reconcilerChildren(props.children, node)
  return node
}

function updateFunctionComponent(vnode) {
  const { type, props } = vnode // type就是函数的名字
  const vvnode = type(props) // 得到虚拟节点
  const node = createNode(vvnode)
  return node
}

function updateClassComponent(vnode) {
  const { type, props } = vnode
  const cmp = new type(props)
  const vvnode = cmp.render()
  const node = createNode()
  return node
}

class Component {
  static isReactComponent = {}
	constructor(props) {
    this.props = props
  }
}

function updateNode(node, nextValue) {
  Object.keys(nextValue)
    .filter(k => k !== "children")
  	.forEach(k => { 
    	if(k.slice(0,2)==='on') {
        let eventName = k.slice(2).toLowerCase()
        node.addEventListener(eventName, nextValue[k]) 
      } else {
        node[k] = nextValue[k]
      }
  	})
}

function reconcilerChildren(children, node) {
  for(let i=0; i<children.length; i++) {
    let child = children[i]
    if(Array.isArray(child)) {
      for(let j=0; j<child.length; j++ ) {
        render(child[j], node)
      }
    }
    render(children[i], node)
  }
}
```







# Dva

​	dva首先是一个基于redux和redux-saga的数据流方案，然后为了简化开发体验，dva还额外内置了react-router和fetch，所以也可理解为一个轻量级框架。

​	dva = React-router + Redux + Redux-saga

```js
import React from 'react';
import dva, {connect} from 'dva'
import { Dispatch } from 'redux';
import {Router, Route} from 'dva/router'
import {Link} from 'react-router-dom'

let app = dva()
interface Counter1State {
  number: number
}
type Counter1Props = Counter1State & {
  dispatch: Dispatch<Object>
}
interface Counter2State {
  number: number
}
type Counter2Props = Counter2State & {
  dispatch: Dispatch<Object>
}
app.model({
  namespace: 'counter1', // model可以定义很多个，每个都有自己的命名空间
  state: {number: 0},
  reducers: {
    increment(state) {// 函数名actionTypes, state老的状态, return新的状态
      return {
        number: state.number + 1
      }
    },
    decrement(state) {
      return {
        number: state.number - 1
      }
    }
  },
  effects: {
    *asyncIncrement(action, {call, put}) {// redux-saga/effects
      // yield call(delay, 1000)
      yield call(delay)
      yield put({type: 'increment'});
    }
  },
  // 在系统启动的时候把所有订阅函数都执行一遍，执行一次
  subscriptions: {
    changeTitle({history, dispatch}) {
      history.listen(({pathname}) => {
        document.title = pathname
      })
    }
  }
})
const delay = ()=>new Promise(function(res, rej){
  setTimeout(function() {
    res()
  },1000)
})

app.model({
  namespace: 'counter2',
  state: {
    number: 0
  },
  reducers: {
    increment(state) {
      return {
        number: state.number + 1
      }
    },
    decrement(state) {
      return {
        number: state.number - 1
      }
    }
  }
})

const Counter1 = (props: Counter1Props) => (
  <div>
    <p>{props.number}</p>
    <button onClick={() => props.dispatch({type:'counter1/increment'})}>+</button>
    <button onClick={() => props.dispatch({type:'counter1/decrement'})}>-</button>
    <button onClick={() => props.dispatch({type:'counter1/asyncIncrement'})}>async+</button>
  </div>
)
const mapStateToProps1 = (state) => state.counter1
const ConnectedCounter1 = connect(mapStateToProps1)(Counter1)

const Counter2 = (props: Counter2Props) => (
  <div>
    <p>{props.number}</p>
    <button onClick={() => props.dispatch({type:'counter2/increment'})}>+</button>
    <button onClick={() => props.dispatch({type:'counter2/decrement'})}>-</button>
  </div>
)
const mapStateToProps2 = (state) => state.counter2
const ConnectedCounter2 = connect(mapStateToProps2)(Counter2)
app.router((props)=>{
  let {history, app} = props
  return (
    <Router history={history}>
    <>
      <Link to="/counter1">counter1</Link>
      <Link to="/counter2">counter2</Link>
      <Route path="/counter1" component={ConnectedCounter1}/>
      <Route path="/counter2" component={ConnectedCounter2}/>    
    </>
    </Router>
  )
})

app.start('#root')
```

# Umi

​	早先先出的是dva，然后是umi，umi一般就包含了dva

​	每次新建项目的时候重新做那套项目的架构问题很大，每次我都要把这些库引进来，然后注册来注册去的，后面出了个dva，把好多东西都整合到了一起，实现开箱即用

​	创建项目后就是一个类似于后台管理系统的初始页面



**基本使用**

1. npm init -y
2. 创建pages目录
3. 建立pages文件夹下的单页面   =>   umi g page about
4. 建立文件夹channel   =>   umi g page channel/index



创建完路由会跟着更新

umi g page channel/index –less

​	写页面没有什么区别

​	写接口的时候一般在models里面写，新建一个channel2.js

```js
const Model = {
  nameSpace:'channel',
  state:{
    data:[]
  },
  effect:{
    // 用来获取列表
    *getChannelData({payload},{call, put}) {
        const res = yield call(fetch(channelData, payload))
        yield put({
          type: 'channelData', 
          payload: res
        })
    },
    *getChannelDataBySearch(){
      const res = yield call(fetch(channelDataBySearch, payload))
      yield put({
        type: 'channelDataBySearch', 
        payload: res
      })
    }
  },
  reducers: {
    channelData(state, { payload }) {
        return {…state, data:[...payload.data]}
    }
  }
}
export default Model
```



那model和组件怎么关联起来呢？还是connect

```js
import {connect} from 'dva'
export default connect(
   // mapStateToProps
   ({channel}) => {
      return { …channel }
   }
   // mapDispatchToProps
   {
     getChannelData:()=>({
         type:'channel/getChannelData'
     }),
     getChannelDataBySearch: (name)=>({
         type:'channel/getChannelData', name
     }),
   }
   )(class Channel extends Component {
    componentDidMount() {
        this.props.getChannelData()
    }
       
   Button.onClick=this.props.getChannelDataBySearch(name)
       render(){
           const {data} = this.props
           return (
               Input
               Button
               List
           )
       }
   }
)
```

# 服务端渲染

​	它可以在请求时，在服务端直接生成一个完整结构的 HTML 文件，返回给浏览器展示。这样减少了首次加载白屏的时间，同时解决了 SEO 不友好的缺点。

```js
ReactDOMServer.renderToString(element)
ReactDOMServer.renderToStaticMarkup(element)
```

⚠️18



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