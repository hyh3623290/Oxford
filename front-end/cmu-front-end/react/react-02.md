# React-Router



## props

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



## withRouter

​	如果一个组件不是路由绑定组件，那么该组件的props中是没有路由相关对象的，虽然我们可以通过传参的方式传入，但是如果结构复杂，这样做会特别繁琐。幸好，我们可以通过withRouter高阶组件这个方法来注入路由对象。

```js
import { withRouter } from 'react-router-dom'

function Index(props) {
  console.log(props)
  ...
}
    
export default withRouter(Index)
```



如果是hooks就简单了

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

1. HashRouter简单，不不需要服务器器端渲染，靠浏览器器的# 的来区分path就可以，BrowserRouter需要服务器器端对不不 同的URL返回不不同的HTML，后端配置可参考。BrowserHistory切换路由的时候每次会刷新一下，是不是相当于给后端发了一个请求，如果后台没有做处理的话是不是会404

2. 使⽤用 HTML5 提供的 history API (pushState, replaceState 和 popstate 事件) 来保持 UI 和 URL 的同步。

3. 使⽤用 URL 的 hash 部分（即 window.location.hash）来保持 UI 和 URL 的同步。

   注意：hash history 不支持 location.key 和 location.state。在以前的版本中，我们曾尝试 shim 这种行为，但是仍有一些边缘问题无法解决。因此任何依赖此行为的代码或插件都将无法正常使⽤用。由于该技术仅 ⽤用于支持旧浏览器，因此我们鼓励大家使用 <BrowserHistory> 

4. Hash history不需要服务器任何配置就可以运行

**额外：**MemoryRouter 将URL的历史记录保存在内存中的`<Router>`，不读取，也不写入地址栏，在测试和非浏览器环境中很有用。如React Native





## basename

​	所有URL的base值。如果你的应⽤用程序部署在服务器器的⼦子⽬目 录，则需要将其设置为⼦子⽬目录。basename 的格式是前⾯面有 一个/，尾部没有/

​	其实就是加一个前缀嘛

```jsx
<BrowserRouter basename="/kkb">  
    <Link to="/user" /> 
</BrowserRouter>
```





## Route渲染方式

​	**Route渲染优先级**：children>component>render。这三种方式互斥，你只能用⼀种。三者能接收到同样的[route props]，包括match, location and history，但是当不匹配的时候，children的match为 null

​	**children**和**render**使用方式一样，只不过children不管有没有匹配到都显示

​	**component**的话其实也可以用函数的方式写，但是这样不好`<Route component={()=><child />}>`，这样的话组件更新的时候会不停的挂载卸载。因为渲染component的时候会调用React.createElement，如果使用下面这种匿名函数的形式，每次都会生成一个新的匿名的函数，导致生成的组件的type总是不相同，这个时候会产生重复的卸载和挂载。为什么type值不同，因为它是匿名函数，两个匿名函数怎么能相同呢

​	当你⽤component的时候，Router会用你指定的组件和 React.createElement创建一个新的[React element]。这意 味着当你提供的是一个内联函数的时候，每次render都会创建一个新的组件。这会导致不再更更新已经现有组件，而是直接卸载然后再去挂载一个新的组件。因此，当⽤用到内联函数的内联渲染时，请使⽤用render或者children。



## 嵌套

路由也是可以嵌套的，页面里写个`<Route />`即可



## 动态路由	

**动态路由**也是可以嵌套的 

```jsx
<Route path="/search/:id" component={Search} />
<Route path={"/search:" + id + "/detail"} component={ Detail } />
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

### Switch

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



react只是非常简单的一个视图层框架，配合redux可以完成大型项目

Redux = Reducer + Flux



## 基本使用

## combineReducer

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
```

这样以后其他方法跟平常一样使用



## 异步

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







## Dva

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



## Umi

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





























-----------------------------------------------------------















































































