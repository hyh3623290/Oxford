# React-Router

## Route

​	当 URL 和 Route 匹配时，Route 会创建一个 match 对象作为 props 中的一个属性传递给被渲染的组件。

### 渲染方式

1. 渲染优先级：children>component>render。 
2. component需要传一个组件，render的时候，你调⽤的只是个函数。两者只在当location匹配的时候渲染
3. 当不匹配的时候，children的match为 null。不管location是否匹配，你都需要渲染⼀些内容，这时候你可以⽤children。其它⼯作⽅法与 render完全⼀样。

⚠️ component其实也可以给一个函数，但这样会导致每次render都会产生重复的卸载和挂载。因为渲染component的时候会调用React.createElement，如果使用匿名函数的形式，每次都会生成一个新的匿名的函数，导致生成的组件的type总是不相同

### 嵌套

路由也是可以嵌套的，页面里写个`<Route />`即可

### 动态路由	

**动态路由**也是可以嵌套的 

```jsx
<Route path="/search/:id" component={Search} />
<Route path={"/search:" + id + "/detail"} component={ Detail } />
```



## props

### 属性

​	路由中的属性会通过props传递，有三个东西，打印出来可以看到是history，location，match。

​	route要写在Router里面，因为要用到history，location这些都来自router

```js
function Index(props) {
  console.log(props)
  return (
    ...
  )
}
```

1. history：历史记录及路由给我们的一些操作

   - `history.length`
   - `history.go()` 正数的话向前跳，负数的话向后跳
   - `history.push()` 修改当前的url
   - `history.replace()` 同上，不过不会多加一条历史记录

   如果用window.location.href的话会刷新页面

   

2. location：获取当前url的一些信息，如`localhost:3000/?id=12 `

   - `hash: "";`
   - `pathname: '/'; `
   - `search: '?id=12';`
   - `state: 'hehe';   // history.push('index', hehe)`

   

3. match：路由本身匹配的信息

   - isExact
   - params：动态路由传过来的参数
   - path
   - url

但是如果用之前的render方法则不会有这些属性的，这时候想用路由参数的话会稍微有些麻烦，

```jsx
<Route path="/index" exact render={(props) => {
  console.log(props)
  return <Index {...props}/>
}}/>
```



那普通的组件（不是路由组件）怎么获取到这些信息呢

### withRouter

​	如果一个组件不是路由绑定组件，那么该组件的props中是没有路由相关对象的，虽然我们可以通过传参的方式传入，但是如果结构复杂，这样做会特别繁琐。幸好，我们可以通过withRouter高阶组件这个方法来注入路由对象。

```js
import { withRouter } from 'react-router-dom'

function Index(props) {
  console.log(props)
  ...
}
    
export default withRouter(Index)
```



### hooks

- useHistory
- useLocation
- useParams
- useRouteMatch

```js
import { useHistory } from 'react-router-dom'

let { goBack } = useHistory()
```



## Redirect

```jsx
<Route path="/" exact render={(props) => {
  console.log(props)
  return <Redirect to="/list/1"/> // #1
}}/>
```

1. 当我们访问/这个路径时就会被重定向到列表页



## Browser和Hash

1. HashRouter简单，不需要服务器端渲染，靠浏览器的# 的来区分path就可以，BrowserRouter需要服务器端对不 同的URL返回不不同的HTML，后端配置可参考。BrowserHistory切换路由的时候每次会刷新一下，是不是相当于给后端发了一个请求，如果后台没有做处理的话是不是会404

2. 使⽤用 HTML5 提供的 history API (pushState, replaceState 和 popstate 事件) 来保持 UI 和 URL 的同步。

3. 使⽤用 URL 的 hash 部分（即 window.location.hash）来保持 UI 和 URL 的同步。

   注意：hash history 不支持 location.key 和 location.state。在以前的版本中，我们曾尝试 shim 这种行为，但是仍有一些边缘问题无法解决。因此任何依赖此行为的代码或插件都将无法正常使⽤用。由于该技术仅 ⽤用于支持旧浏览器，因此我们鼓励大家使用 <BrowserHistory> 

4. Hash history不需要服务器任何配置就可以运行

**额外：**MemoryRouter 将URL的历史记录保存在内存中的`<Router>`，不读取，也不写入地址栏，在测试和非浏览器环境中很有用。如React Native





## basename

​	所有URL的base值。如果你的应⽤用程序部署在服务器器的子⽬录，则需要将其设置为⼦目录。basename 的格式是前⾯面有 一个/，尾部没有/

​	其实就是加一个前缀嘛

```jsx
<BrowserRouter basename="/kkb">  
    <Link to="/user" /> 
</BrowserRouter>
```











## 路由守卫

**路由守卫 - 去某个页面如果没登陆就去登录 - 登录完去到对应的页面**

```jsx
<PrivateRoute path="/user" component={UserPage} />

export default class PrivateRoute extends Component {
  const {isLogin, path, component} = this.props
	if (isLogin) {
      return <Route path={path} component= {component} />
    } else {
      return <Redirect to={{pathname: '/login', state: {redirect: path}}}/>
    }
}

export default class LoginPage extends Component {  
  render() {    
    const { isLogin, location } = this.props
    const { redirect = '/' } = location.state || {} // 给redirect一个默认值
    if(isLogin) {
      return <Redirect to={redirect}/>
    } else {
      return <h1>LoginPage</h1>
    }
  } 
}
```

​	路由守卫，也是一个高阶组件，因为你会接收到path和component，如果登录就跳到对应的页面，否则就重定向到登录页并带上路由的信息，因为你还要跳回来。登录页逻辑，当你重定向到登录页的时候依旧会带上参数，没有的话就给个默认值，登录成功返回原有路由，否则即在登录页做操作

​	路由守卫，拿到路由信息以后判断是否登录从而判断是否要去登录页面，

​	登录页，依旧拿到路由信息判断是否登录从而判断是否要去重定向页面





## 手写Router

```jsx
<div>
  <BrowserRouter>
    <Link to="/user">用户  </Link>
    <Link to="/home">首页  </Link>
    <Link to="/login">登录  </Link>
    <Link to="/children">children方法  </Link>
    <Link to="/render">render方法  </Link>
    <Switch>
      <Route path='/user' component={User} />
      <Route path='/home' component={Home} />
      <Route path='/login' component={Login} />
      <Route path='/children' children={() => <div>children</div>} />
      <Route path='/render' render={() => <div>render</div>} />
    </Switch>
  </BrowserRouter>
</div>
```



### BrowserRouter

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



### Route

```jsx
import React, {Component, Children} from "react";
import {RouterContext} from "./RouterContext";
import matchPath from "./matchPath";

// 这里的children不管是否匹配match都可以存在，这里能不能直接返回，就不判断了
// match 匹配 children是function或者是节点
// 不match 不匹配  children是function

export default class Route extends Component {
  render() {
    return (
      <RouterContext.Consumer>
        {context => {
          const {path, computedMatch, children, component, render} = this.props;
          // const match = context.location.pathname === path;
          const location = this.props.location || context.location;
          const match = computedMatch
            ? computedMatch
            : path
            ? matchPath(location.pathname, this.props)
            : context.match;
          const props = {
            ...context,
            location,
            match
          };
          //  children, component, render 能接收到(history, location match),
          // 所以我们定义在props，传下去

          // match 渲染children, component, render 或者null
          // match的时候如果children存在：function或者children本身
          // 不match children 或者 null
          // children是和匹配无关

          //这里只是简单处理 ，所以呢 我们还是不要自己去创建element了，还是用createElement
          // let element;
          // if (match && component) {
          //   console.log(
          //     "component",
          //     component,
          //     React.isValidElement(component)
          //   ); //sy-log
          //   // 如果这里想要用cloneElement，首先得有个element
          //   if (typeof component === "function") {
          //     // class function
          //     // 怎么判断class组件和function组件
          //     if (component.prototype.isReactComponent) {
          //       // class组件
          //       const cmp = new component(component.props);
          //       element = cmp.render();
          //     } else {
          //       //function组件
          //       element = component(props);
          //     }
          //   } else {
          //     // 对象
          //     const cmp = new component.WrappedComponent({user: {}, ...props});
          //     element = cmp.render();
          //   }
          // }
          return (
            <RouterContext.Provider value={props}>
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
            </RouterContext.Provider>
          );

          // return match ? React.createElement(component, this.props) : null;
        }}
      </RouterContext.Consumer>
    );
  }
}


```



### Link

```react
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

​		NavLink 是特殊的 Link，可以为与当前 URL 匹配的 Link 元素增加样式属性。唉，其实就是加一些特殊样式的Link

### Switch

​	Switch 通常被用来包裹 Route，用于渲染与路径匹配的第一个子 `<Route>` 或 `<Redirect>`，它里面不能放其他元素。

```jsx
import React, {Component} from "react";
import {RouterContext} from "./RouterContext";
import matchPath from "./matchPath";

export default class Switch extends Component {
  render() {
    return (
      <RouterContext.Consumer>
        {context => {
          // 找出渲染的，第一个符合匹配的元素，存在element
          // const {location} = context;
          // 优先用props上的location
          const location = this.props.location || context.location;
          let element,
            match = null;
          let {children} = this.props;
          // children = [children];
          // // this.props.children  数组形式、对象
          // for (let i = 0; i < children.length; i++) {
          //   let child = children[i];
          //   if (match === null && React.isValidElement(child)) {
          //     element = child;
          //     const path = child.props.path;
          //     match = path
          //       ? matchPath(location.pathname, {...child.props, path})
          //       : context.match;
          //   }
          // }
          React.Children.forEach(children, child => {
            if (match === null && React.isValidElement(child)) {
              element = child;
              const path = child.props.path;
              match = path
                ? matchPath(location.pathname, {
                    ...child.props,
                    path
                  })
                : context.match;
            }
          });
          // console.log("element", element, React.isValidElement(element)); //sy-log
          // createElement(type, props)
          // return match
          //   ? React.createElement(element.type, {
          //       ...element.props,
          //       location,
          //       computedMatch: match
          //     })
          //   : null;
          // cloneElement(element, otherProps)
          return match
            ? React.cloneElement(element, {
                location,
                computedMatch: match
              })
            : null;
        }}
      </RouterContext.Consumer>
    );
  }
}

```

### Redirect

```jsx
import React, {Component} from "react";
import {RouterContext} from "./RouterContext";

export default class Redirect extends Component {
  render() {
    return (
      <RouterContext.Consumer>
        {context => {
          const {history} = context;
          const {to} = this.props;
          // history.push(to)
          return <LifeCycle onMount={() => history.push(to)} />;
        }}
      </RouterContext.Consumer>
    );
  }
}

class LifeCycle extends Component {
  componentDidMount() {
    if (this.props.onMount) {
      this.props.onMount();
    }
  }
  render() {
    return null;
  }
}

```





**BrowserRouter**

​	利用context提供history，渲染children

**Route**

​	当location.pathname与当前path相同时渲染component（createElement）

**Link**

​	返回a标签，href是就是to，同时阻止默认事件并通过history.push去对应的页面。渲染children

















# Redux

​	react只是非常简单的一个视图层框架，配合redux可以完成大型项目

​	Redux = Reducer + Flux

​	Redux 主要由三部分组成：**store、reducer 和 action**。**在 Redux 的整个工作过程中，数据流是严格单向的**。这一点一定一定要背下来，面试的时候也一定一定要记得说——不管面试官问的是 Redux 的设计思想还是工作流还是别的什么概念性的知识，开局先放这句话，准没错。

## store

​	使用 createStore 来完成 store 对象的创建

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

## reducer

​	reducer 的作用是将新的 state 返回给 store，state 的计算过程就称为 reducer。一个 reducer 一定是一个纯函数，它可以有各种各样的内在逻辑，但它最终一定要返回一个 state：

```js
const reducer = (state, action) => {
  // 此处是各种样的 state处理逻辑，如下
  switch (action.type) {
    case "ADD":
      return {...state, count: state.count+1}
    case "MINUS":
      return {...state, count: state.count-1}
    default:
      return state;
  }
}
```

​	当我们基于某个 reducer 去创建 store 的时候，其实就是给这个 store 指定了一套更新规则：

### combineReducer

```js
import { combineReducer } from 'redux'

function index = (state={info: "首页数据"}, action) {
  return state
}
function list = (state={info: "列表数据"}, action) {
  return state
}

let reducer = combineReducer({
  index,
  list
})

createStore(reducer)
// store.getState().index
```

这样以后其他方法跟平常一样使用

## action

​	View 会发送多种消息，这就需要定义多种 Action，如果每个都手写，会重复工作。我们可以定义一个函数来生成 Action，这个函数就称为 Action Creator。

```js
const ADD_TODO = 'ADD_TODO'

function addTodo(type, text) {
    return {
        type,
        payload: text
    }
}

const action = addTodo(ADD_TODO, 'Learn Redux')
```

## dispatch

​	派发 action，靠的是 dispatch

## subscribe

​	Store 允许使用 `store.subscribe()` 方法设置监听函数，State 发生变化时会自动执行这个函数。`store.subscribe` 方法返回一个函数，调用这个函数就可以解除监听。	

```js
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
)

unsubscribe()
```





## 中间件与异步

​	Action 发出后，Reducer 会立刻算出 State，称为同步。如果 Action 发出后，过一段时间再执行 Reducer，怎样让 Reducer 在异步操作结束之后自动执行呢？这时候就需要用到新的工具：中间件（middleware）。

​	中间件其实就是一个函数，它对 `store.dispatch()` 方法进行了改造，在发出 Action 和执行 Reducer 这两步之间，添加了其他功能。

⚠️11 稍后

```js
componentDidMount() {
  axios.get('./list.json').then((res) => {
    const data = res.data
    const action = initListAction(data)
    store.dispatch(action)
  })
}
```



### redux-saga

优点：利用generator，处理复杂逻辑时及其强大，action依然是对象，不想thunk变成一个函数，这样程序的一致性就更强劲了，易管理，执行，测试，失败处理

缺点：用起来稍微繁琐一些

建立sagas.js理解为任务清单

#### store/index.js

```js
import  { createStore, applyMiddleWare }  from  'redux'
import  createSagaMiddleware  from  'redux-saga'
import reducer from  './reducer.js' 
import  todoSagas  from './sagas.js'

const sagaMiddleware = createSagaMiddleware()
const store = createStore(reducer, applyMiddleWare(sagaMiddleware))
sagaMiddleware.run(todoSagas)
export default store
```



再saga里我们会把异步逻辑放到单独的一个文件去管理，创建sagas.js文件 , 这个文件也可以接收到action



#### sagas.js

```js
import { takeEvery , put , } from 'redux-saga/effects'
function* getInitList () {
  try {
    const res = yield axios.get( '/list.json' )
    const action = initListAction(res.data)
    yield put(action)   // store.dispatch
  } catch(e) {
    alert('fail')
  }
}
function* mySaga() {
  yield takeEvery(GET_INIT_LIST，getInitList)
}
export default mySaga
```

call：调用异步操作

put：状态更新

takeEvery：做saga监听



#### store/actionCreators

```js
export const getTodoList = () => ({
  type:  GET_INIT_LIST
})
```



#### TodoList.js

```js
componentDidMount() {
  const action = getTodoList()
  store.dispatch(action)
}
```



## reduce

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
// reduce接收一个函数，函数接收两个参数，一个是累计值，一个是当前值
// 函数返回到结果会给到累计值里,刚开始时为数组前两个
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
    dispatch = compose(...chain)(dispatch) // thunk(logger(reduxApi)(dispatch))
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
function thunk({dispatch, getState}) {
  return dispatch => action => {
    if(typeof action === "function") {
      return action(dispatch, getState)
    } else {
      return dispatch(action) 
    }
  }
}


```

# React-Redux

## 预备

```jsx
import { Provider } from 'react-redux'
ReactDOM.render(
	<Provider store={tore}>
  	<App />
  </Provider>,
  document.getElementById('root')  
)
```



```js
export default connect(
	state => ({ count: state.count })// es6写法，相当于直接return这个对象
)(
  class Cmp extends Component {
    render() {
      console.log(this.props)
      // ...
    }
  }
)
```



```js
// mapState可以接收两个参数，第二个为该组件本身的props,就是没有被高阶组件修饰前的props
// 但是这个ownProps谨慎使用,因为它变化的话mapState这个函数会重新执行,state也会被重新计算,影响性能
// 假设1处那个获取state的计算比较复杂的话
export default connect(
  (state, ownProps) => {
    console.log(ownProps)
    return {
      count: state.count // 1
    }
  }
)()
// mapStateToProps 函数
// mapDispatchToProps 对象/函数
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

​	这会儿再打印props，发现dispatch的位置上放的是add，就可以这样。但是我又想要dispatch和add都有怎么办呢。其实mapDispatch即可以是对象，也可以是函数，如果想这样就用函数的写法然后一起返回

```js
export default connect(
  state => ({ count: state.count }),
  dispatch => {// 这里第二个参数也可以接收ownProps
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
  dispatch => { 
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

此外其实还有一个参数

`mergeProps(stateProps,dispatchProps,ownProps){`

​		`return {omg:'omg', ...stateProps ...}`

`}`

```js
(stateProps, dispatchProps, ownProps) => {
  console.log(stateProps) // connect里的state
  console.log(dispatchProps) // connect里的dispatch
  console.log(ownProps) // 自己的props
  return { ...stateProps, ...dispatchProps, ...ownProps, omg: "omg" }
}
```



## 手写react-redux

​	react-redux其实就是connect和Provider，Provider简单，创建一个context，从props里拿到store直接传下去就好，并且它只需要渲染this.props.children。

​	connect就是一个高阶函数，它接收两个参数，接收一个组件并返回这个组件。只需要把store里的state和dispatch传给这个组件即可。如何拿到呢，就是consumer。记得要把state传给mapStateToProps函数的参数，dispatch要给默认值。如果mapDispatchToProps是函数和对象的时候要给不同的操作。因为每次state变化子组件要重新渲染，所以你必须把需要传的数据放到state里去

​	**总结：**其实没有什么东西，就是高阶组件connect从context拿值传给子组件

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
      // 传函数就可以用函数形式接收，参数就是传的函数的参数
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



- React事件里面是没有this的，输出是undefined

```jsx
<div onClick={fucntion(){ console.log(this) }}></div>
```

























-----------------------------------------------------------















































































