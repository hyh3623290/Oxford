# Redux

​	Redux 主要由三部分组成：**store、reducer 和 action**。**在 Redux 的整个工作过程中，数据流是严格单向的**。这一点一定一定要背下来，面试的时候也一定一定要记得说——不管面试官问的是 Redux 的设计思想还是工作流还是别的什么概念性的知识，开局先放这句话，准没错。

**combineReducer**

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

**此外subscribe也有解除监听的方法**

```js
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
)

unsubscribe()
```



# Provider

```jsx
import { Provider } from 'react-redux'
ReactDOM.render(
	<Provider store={store}>
  	<App />
  </Provider>,
  document.getElementById('root')  
)
```



# connect

```js
import { connect } from 'react-redux'

class Cmp extends Component {
  render() {
    console.log(this.props) // 此时只有dispatch
    ...
  }
}

export default connect()(Cmp)
```

​	通过props即可获取传入的东西

## 第一个参数

​	第一个参数是一个函数

```js
connect((...args) => { console.log(args);return {} })(Cmp)
```

​	打印上面args得到两个数组，第一个是store里的值，第二个是父组件传的props，所以也可以这么写

```js
connect((state, props) => { return {} })(Cmp)
```

​	允许对store和props做一个合并处理，返回值就是传递给组件的数据

```js
connect((state, props) => { 
  return {
    ...props,
    foo: state.foo
  } 
})(Cmp)

// es6写法
connect(
  state => ({ count: state.count })
})(Cmp)
```



## 第二个参数

​	它可以是一个对象，也可以是函数。如果不指定它的话默认dispatch注入props

​	对象的情况如下：

```js
export default connect(
  state => ({ count: state.count }),
  {
    add: () => ({ type: 'Add' })
  }
)(Cmp)
```

​	这会儿再打印props，发现dispatch的位置上放的是add

​	如果想要dispatch和add一起就用函数的方式写

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

​	如果res对象写在了另外一个文件中，这样写就不需要把dispatch也传到那个文件中

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













