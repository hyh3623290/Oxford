# Form

⽤Form.create()的⽅式实现： 

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



# redux

## 预备

有如下函数， 聚合成一个函数，并把第一个函数的返 回值传递给下一个函数，如何处理

```js
function f1(arg) {
	console.log("f1", arg);
	return arg;
}
function f2(arg) {
	console.log("f2", arg);
	return arg;
}
function f3(arg) {
	console.log("f3", arg);
	return arg;
}
```

```js
// 1. 
let res = f1(f2(f3("omg")))
console.log(res)

// 2. 效果一样
compose(f1, f2, f3)("omg")
function compose(...funcs) {
  if(funcs.length === 0) {
    return arg => arg
  }
  if(funcs.length === 1) {
    return funcs[0]
  }
  return funcs.reduce((f, fn) => (...args) => f(fn(...args)))
}
// 当你遇到一个累加，传递的操作时就可以想到reduce了
```



## 效果

```jsx
import React, {Component} from "react";
import store from '../components/kredux'

export default class ReduxPage extends Component {
  constructor(props) {
    super(props);
    this.state = {
      
    };
  }
  add = () => {
    store.dispatch({type: "ADD"})
  }
  minus = () => {
    store.dispatch({type: "MINUS"})
  }
  thunkFn = () => {
    store.dispatch(() => {
      setTimeout(() => {store.dispatch({type: "ADD"})}, 1000)
    })
  }
  componentDidMount() {
    store.subscribe(() => {
      this.forceUpdate()
    })
  }
  render() {
    // store.subscribe(()=>{console.log(store.getState())})
    return (
      <div>
        <h3>ReduxPage</h3>
        <div>{store.getState()}</div>
        <button onClick={this.add}>增加</button>
        <button onClick={this.minus}>减少</button>
        <button onClick={this.thunkFn}>thunk</button>
      </div>
    );
  }
}
```



## 手写redux 

```js
export function createStore(reducer) {
  let currentState = undefined

  let currentListener = []

  function getState() {
    return currentState
  }

  function dispatch(action) {
    currentState = reducer(currentState, action)
    currentListener.map(v => v())
  }

  function subscribe(cb) {
    currentListener.push(cb)
  }
  dispatch({ type: '38hecb48h4c4-redux' }) // 使redux产生一个初始值, 因为reducer默认有个state
  return {
    getState,
    dispatch,
    subscribe
  }
}
```



## 手写applyMiddleware

​	没有中间件，action就直接到了reducer。中间件就是一个函数，对store.dispatch方法进⾏改造， 在发出 Action 和执⾏ Reducer 这两步之间，添加了其他功能。经过处理之后，他会先经过所有中间件按顺序执行完毕以后才可以达到store，再交给reducer，

​	applyMiddleWare是让我们redux支持中间件的一个机制，作为createStore的第二个参数，并且是立即执行的，里面的参数的话是中间件的列表，因为它是有执行顺序的。中间件都是函数

```js
export function createStore(reducer, enhancer) {
  if(enhancer) {
    return enhancer(createStore)(reducer)
  }
  // ...
}

export function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    const store = createStore(...args)
    let dispatch = store.dispatch
    const reduxApi = {
      dispatch: (...args) => dispatch(...args),
      getState: store.getState,
    }
    const chain = middlewares.map(mw => mw(reduxApi))
    dispatch = compose(...chain)(store.dispatch)
    return {
      ...store,
      dispatch
    }
  }
}

export function compose(...funcs) {
  return funcs.reduce((left, right) => (...args) => right(left(...args))) // 6
}
```





## 手写中间件

```js
// logger
function logger() {
  return dispatch => action => {
    console.log(action.type + '执行了')
    return dispatch(action)
  }
}

// thunk
function thunk() {
  return dispatch => action => {
    if(typeof action === "function") {
      return action(dispatch, getState)
    } else {
      return dispatch(action) 
    }
  }
}


```

# react-redux

## 预备

```js
export default connect()(
  class Cmp extends Component {
    render() {
      console.log(this.props)
      // ...
    }
  }
)
```

可以看到输出的里面就有个dispatch属性了，那如何拿到state呢

```js
export default connect(
  state => ({ count: state.count })
)()

// 可以接收两个参数，第二个为该组件本身的props,就是没有被高阶组件修饰前的props
// 但是这个ownProps谨慎使用,因为它变化的话mapState会重新执行,state也会被重新计算,影响性能
export default connect(
  (state, ownProps) => {
    console.log(ownProps)
    return {
      count: state.count
    }
  }
)()
```

​	你再看输出的props发现里面连count都有了，接下来看mapDispatchToProps，它是一个**对象**，如果不指定它的话默认dispatch注入props，所以我一般就只写mapState，不写mapDispatch

```js
export default connect(
  state => ({ count: state.count }),
  {
    add: () => ({ type: 'Add' })
  }
)()
```

这会儿再打印props，发现dispatch的位置上放的是add，就可以这样。但是我又想要dispatch和add都有怎么办呢。其实mapDispatch即可以是对象，也可以是函数，如果想这样就用函数的写法然后一起返回

```js
export default connect(
  state => ({ count: state.count }),
  dispatch => {
    let res = {
      add: () => dispatch({ type: 'ADD' })
      minus: () => dispatch({ type: 'MINUS' })
  	}
    return {
      dispatch,
      ...res
    }
  }
)()
```

如果说我当前的res对象来自另外一个文件夹，这时候dispatch我是不是还得传过去呀，太麻烦了，可不可以不用dispatch，拿一个函数统一处理action呀，库也已经实现了`bindActionCreators`，

```js
import { bindActionCreators } from 'redux'

export default connect(
  state => ({ count: state.count }),
  dispatch => { // 这里第二个参数也可以接收ownProps
    let res = {
      add: () => ({ type: 'Add' }),
      minus: () => ({ type: "Minus" })
  	}
    res = bindActionCreators(res, dispatch)
    return {
      dispatch,
      ...res
    }
  }
)()
```

怎么实现呢

```js
// let res = {
//   add: () => ({ type: 'Add' }),
//   minus: () => ({ type: "Minus" })
// }

function bindActionCreator(creator, dispatch) {
  return (...args) => dispatch(creator(...args))
}

export function bindActionCreators(creators, dispatch) {
  const obj = {}
  for(const key in creators) {
    obj[key] = bindActionCreator(creators[key], dispatch)
  }
  return obj
}
```



## 效果

```jsx

```





## 手写react-redux

要有provider和connect，还要传store，怎么传呢，就是context

```jsx
import React, {Component} from "react";
// import {bindActionCreators} from "redux";

const ValueContext = React.createContext();

export const connect = (
  mapStateToProps = state => state,
  mapDispatchToProps
) => WrappedComponent => {
  return class extends Component {
    // 此时组件的所有生命周期都能获得this.context
    static contextType = ValueContext;
    constructor(props) {
      super(props);
      this.state = {
        props: {}
      };
    }
    componentDidMount() {
      const {subscribe} = this.context;
      this.update();
      // 订阅
      subscribe(() => {
        this.update();
      });
    }

    update = () => {
      const {getState, dispatch, subscribe} = this.context;
      //  getState获取当前store的state
      let stateProps = mapStateToProps(getState());
      let dispatchProps;
      // mapDispatchToProps Object/Function
      if (typeof mapDispatchToProps === "object") {
        dispatchProps = bindActionCreators(mapDispatchToProps, dispatch);
      } else if (typeof mapDispatchToProps === "function") {
        dispatchProps = mapDispatchToProps(dispatch, this.props);
      } else {
        // 默认
        dispatchProps = {dispatch};
      }
      this.setState({
        props: {
          ...stateProps,
          ...dispatchProps
        }
      });
    };
    render() {
      console.log("this.context", this.context); //sy-log
      return <WrappedComponent {...this.state.props} />;
    }
  };
};

export class Provider extends Component {
  render() {
    return (
      <ValueContext.Provider value={this.props.store}>
        {this.props.children}
      </ValueContext.Provider>
    );
  }
}

function bindActionCreator(creator, dispatch) {
  return (...args) => dispatch(creator(...args));
}

// {
//   add: () => ({type: "ADD"})
// }
export function bindActionCreators(creators, dispatch) {
  const obj = {};
  for (const key in creators) {
    obj[key] = bindActionCreator(creators[key], dispatch);
  }
  return obj;
}

```

















---------------------------------------------------

- 接收一个组件，返回一个组件

```js
const App = Cmp => props => {
  
}
```

​	`props => {}`代表返回的组件，返回的组件当然要接收props

​	不要再render里使用高阶组件（如下），React的diff算法使用组件标识来确定他是应该更新现有子树还是将其丢弃并挂载新子树。在render里，每次执行都会创建一个新的组件，会导致原有组件重新卸载和挂载

```js
render() {
  const Hoc = App(Cmp)
  return <Hoc />
}
```

两个箭头的写法：首先App(Cmp) 返回的是一个函数，那App(Cmp)(k)，k就是props





- for in遍历的是数组的索引（即键名），而for of遍历的是数组元素值。

  

- reduce

  从术语名称来看，Reduce方法是指将多个值缩减为一个。然而，笔者发现，与“减少”相比，把它想成是“积累”更容易操作。.reduce()方法的回调需要两个参数：累加器和当前值，比如写个相加的方法

  ```js
  let arr = [1, 2, 3, 4, 5]
  let total = arr.reduce((total, item) => {
    return total + item
  }, 0)
  ```

  ​	接收两个参数，回调和初始值，回调一样只是多加个total，没有初始值的话默认初始值使用数组的第一个参数，并且从第二个开始，total也是第一个参数











































----------------------------------------------------

- jsx需要babel编译才可以使用

- **插值表达式**：`{}`	

  如果是布尔值，undefined，null则啥都不显示，如果是数组去掉逗号全部输出，如果是对象则不能解析，但可以这样

`Object.values(data)`输出value

- **条件渲染**：`{true && "成立输出"}`	`{false || "不成立则输出"}`

- class要写作className，style接收的是一个对象

```html
let style = {
  width: "20px",
  height: "20px"
}
ReactDom.render(
  <div className="box" style={style}>
  	<p>hello react</p>
  </div>,
  document.querySelector("#root"),
)
```

​	此外所有标签名小写

​	如果不希望父元素被渲染，可以用`React.Fragment`

- React事件里面是没有this的，输出是undefined

```jsx
<div onClick={fucntion(){ console.log(this) }}></div>
```



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

为什么这样设计呢？简单来说，由于 render 阶段的操作对用户来说其实是“不可见”的，所以就算打断再重启，对用户来说也是零感知。而 commit 阶段的操作则涉及真实 DOM 的渲染，再狂的框架也不敢在用户眼皮子底下胡乱更改视图，所以这个过程必须用同步渲染来求稳。





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























--------------------------------------------------------

​	使用函数式组件的时候，应该尽量减少在函数中声明子函数，否则，组件每次更新时都会重新创建这个函数。