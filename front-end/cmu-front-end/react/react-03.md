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





















--------------------------------------------------------

​	使用函数式组件的时候，应该尽量减少在函数中声明子函数，否则，组件每次更新时都会重新创建这个函数。