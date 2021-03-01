# 2.25

## instanceof 的内部机制是什么

​	instanceof内部机制是通过原型链判断的

## 实现一个通用的判断类型的方法

```js
function getType(obj){
  let type  = typeof obj;
  if (type !== "object") {    // 先进行typeof判断，如果是基础数据类型，直接返回
    return type;
  }
  // 对于typeof返回结果是object的，再进行如下的判断，正则返回结果
  return Object.prototype.toString.call(obj).replace(/^\[object (\S+)\]$/, '$1');  // 注意正则中间有个空格
}
```

​	

## 闭包的定义是什么

​	闭包的定义其实很简单：函数 A 内部有一个函数 B，函数 B 可以访问到函数 A 中的变量，那么函数 B 就是闭包。



## 常用的深浅拷贝方法及某个深拷贝方法的缺点

​	Object.assign，`...`

​	`JSON.parse(JSON.stringify(object))`

- 拷贝的对象的值中如果有函数、undefined、symbol 这几种类型，经过 JSON.stringify 序列化之后的字符串中这个键值对会消失；

- 无法拷贝对象的循环应用，即对象成环 (obj[key] = obj)。

## 手写浅拷贝

```js
const shallowClone = (target) => {
  if (typeof target === 'object' && target !== null) {
    const cloneTarget = Array.isArray(target) ? []: {};
    for (let prop in target) {
      if (target.hasOwnProperty(prop)) {
          cloneTarget[prop] = target[prop];
      }
    }
    return cloneTarget;
  } else {
    return target;
  }
}
```





## 手写深拷贝

```js
function deepClone(obj) {
  function isObject(o) {
    return (typeof o === 'object' || typeof o === 'function') && o !== null
  }

  if (!isObject(obj)) {
    throw new Error('非对象')
  }

  let isArray = Array.isArray(obj)
  let newObj = isArray ? [...obj] : { ...obj }
  Reflect.ownKeys(newObj).forEach(key => {
    newObj[key] = isObject(obj[key]) ? deepClone(obj[key]) : obj[key]
  })

  return newObj
}
```



## __proto__ 和 prototype有什么关系

​	打印obj，你发现`obj.__proto__.constructor.prototype === obj.__proto__`

## ES5继承的方法

## map, filter, reduce

​	map 是对每个数组做操作返回

​	filter 是只返回为true的

​	reduce 可以将数组中的元素通过回调函数最终转换为一个值，接受两个参数，分别是回调函数和初始值

```js
const sum = arr.reduce((acc, current) => acc + current, 0)
```



## 说一下Generator

​	通过 * 来声明generator函数，执行它普通函数不同，会返回一个迭代器，要执行迭代器的next才会继续往下执行，第一次执行时，会暂停在第一个yield处，并向外返回一个值。如果传参数的话，会代替上一个yield的返回值

## 定时器有什么问题吗

注意点

- 因为 JS 是单线程执行的，如果前面的代码影响了性能，就会导致 `setTimeout` 不会按期执行
- 如果定时器 `setInterval` 执行过程中出现了耗时操作，多个回调函数会在耗时操作结束以后同时执行

## 手写 call apply bind

```js
Function.prototype.myCall = function(context, ...args) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  context = context || window
  context.fn = this
  const result = context.fn(...args)
  delete context.fn
  return result
}
```

```js
Function.prototype.myApply = function(context) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  context = context || window
  context.fn = this
  let result
  // 处理参数和 call 有区别
  if (arguments[1]) {
    result = context.fn(...arguments[1])
  } else {
    result = context.fn()
  }
  delete context.fn
  return result
}
```

```js
Function.prototype.myBind = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  const _this = this
  const args = [...arguments].slice(1)
  // 返回一个函数
  return function F() {
    // 因为返回了一个函数，我们可以 new F()，所以需要判断
    if (this instanceof F) {
      return new _this(...args, ...arguments)
    }
    return _this.apply(context, args.concat(...arguments))
  }
}
```

​	1

## 手写 new

在调用 `new` 的过程中会发生以上四件事情：

1. 新生成了一个对象
2. 链接到原型
3. 绑定 this
4. 返回新对象

```js
function create() {
  let obj = {}
  let Con = [].shift.call(arguments)
  obj.__proto__ = Con.prototype
  let result = Con.apply(obj, arguments)
  return result instanceof Object ? result : obj
}
```

​	1

## 手写 instanceof

```js
function myInstanceof(left, right) {
  let prototype = right.prototype
  left = left.__proto__
  while (true) {
    if (left === null || left === undefined)
      return false
    if (prototype === left)
      return true
    left = left.__proto__
  }
}
```

​	1



# 2.27

## Fiber阅读





# 2.28

## Redux

### 如何创建一个store

```js
const store = createStore(reducer)
```

### reducer 函数的形式及其参数

```js
function reducer(state = initialState, action) {
  switch(action.type) {
    case 'increment':
      return { count: state.count + 1 }
    default:
      return state
  }
  return state
}
```



### action怎么派发，形式是什么样的

```js
store.dispatch({ type: 'increment' })
```



## React + Redux

### mapStateToProps 接收的参数以及返回值的形式

### mapDispatchToProps 的实质

```js
const mapStateToProps = state => ({
  // 要返回一个对象，会映射到props上，不管写啥都可以，比如随便写个a:'123',也可以通过props.a拿到
  count: state.count
})

const mapDispatchToProps = dispatch => ({
  // 这个方法主要帮我们简化代码，实质就是把返回的对象映射到props属性当中
  // props.increment
  increment() {
    dispatch({ type: 'increment' })
  },
  decrement() {
    dispatch({ type: 'decrement' })
  }
})
export default connect(mapStateToProps, mapDispatchToProps)(Counter)
```



## bindActionCreators 用法及好处

```js
const mapDispatchToProps = dispatch => ({
  ...bindActionCreators({
    increment() {
      return { type: 'increment' }
    }
  }, dispatch)
})
```

​	好处其实就是不用写dispatch了

## 中间件及如何开发，注册

本质上就是一个函数，通过这个函数拓展和增强redux应用程序。体现在对action的处理能力上，以前是直接给reducer处理，现在action会优先被中间件处理。

```js
export default store => next => action => {}

createStore(reducer, applyMiddleware(logger, test))
// 注册：执行顺序是从左往右
```



# 3.1

## redux-saga有什么好处，怎么用

更加的强大：可以将异步操作从 Action Creator 文件中抽离出来，放在一个单独的文件中，这样项目代码就更加的可维护了。

```js
import createSagaMiddleware from 'redux-saga'
const sagaMiddleware = createSagaMiddleware()

createStore(reducer, applyMiddleware(sagaMiddleware))

// 接下来引入下面这个saga文件，把名字传入run方法里
sagaMiddleware.run(sagaName)
```

​	可以创建一个单独的文件来写异步代码了

```js
import { takeEvery, put } from 'redux-saga/effects'

function *increment_async_fn(action) {
  const { data } = yield axios.get('api')
  yield put(increment(action.payload)) // 执行完异步在这里派发一个同步的action
}

export default function *postSaga() {
  // 接收action
  yield takeEvery(INCREMENT_ASYNC, increment_async_fn)
}
```

1. takeEvery 这个方法是用来接收action的
2. put 用来触发另一个 action 的，就跟 dispatch 一样
3. 要求我们默认导出一个generator函数





​	拆分saga，在sagas文件夹下新建root.saga.js

```js
import { all } from 'redux-saga/effects'
import counterSaga from './counter.saga.js'
import modalSaga from './modal.saga.js'

// 依旧需要导出一个generator函数
export default function *rootSaga() {
  
  yield all([
    counterSaga(),
    modalSaga()
  ])
}
```

​	然后需要引入的就是这个saga文件，再把引入的内容放到 `sagaMiddleware.run()` 参数里

## redux-actions

​	帮助我们简化actions代码和reducer代码，因为redux流程中大量的样板式代码读写很痛苦，比如创建大量的 action creator 函数，比如我们把action的type抽象成独立的常量，比如ruducer中通过switch case处理action

```js
// counter.action.js
import { createAction } from 'redux-actions'

// 帮助我们创建action creator函数
export const increment = createAction('increment') // type值, 顺便也不需要常量了
export const decrement = createAction('decrement')
```



```jsx
// Component.js
<Button onClick={props.increment}/>
  
<Button onClick={() => { props.increment(10) }}/>
```



```js
// reducers/counter.reducer.js
import { handleActions as createReducer } from 'redux-actions'

function handleIncrement(state, action) {
  return {
    count: state.count + action.payload // 中间件自动加，不用写在createAction里
  }
}

function handleDecrement(state, action) {
  return {
    count: state.count - 1
  }
}

export default createReducer({
  [increment]: handleIncrement,
  [decrement]: handleDecrement
}, initialState)
```

## Hooks

​	16.8 新增，对函数型组件进行增强，让函数型组件可以存储状态，能够处理副作用

​	只要不是把数据转换为视图对就是副作用，比如获取dom元素，为dom元素添加事件，设置定时器及发送Ajax请求

### 类组件的不足

1. 缺少逻辑复用机制

   渲染属性和高阶组件代码比较复杂，虽然复用逻辑，但都是在原有组件的外层又包裹了一层组件，又没有实际的渲染效果，层级过深变得十分臃肿，增加调试过程的难度，也降低了组件运行的效率

2. 类组件经常会变的很复杂，难以维护

   体现在生命周期函数当中，将一组相干的逻辑拆分到不同的生命周期函数中（比如挂载之后做个什么事，更新之后做个什么事），或者在一个生命周期中存在多个不相干的逻辑

3. 不能够保证this指向的正确性

   当我们给一个元素绑定事件，在事件处理函数当中我们去更改状态的时候，通常要更正函数内部的this指向，否则就会指向undefined，也使得类组件难以维护，而且代码看起来比较复杂

### useState

```jsx
const [ count, setCount ] = useState(0);

// 如果初始值是动态的也可以传一个函数，如,适用于只需要组件第一次执行的时候
// 因为里面的函数只会被执行一次，后续重新渲染的时候不会执行
const [ count, setCount ] = useState(() => {
  return props.count || 0
});

<button onClick={() => { setCount(count + 1) }}></button>

// 也可以传回调，因为是异步的，如果依赖这个新的值就要放到回调中
setCount(count => {
  return count + 1
})
```



# ^_^

周末可以看手写代码的题，如fiber，diff等，周内可以看简单的，也可以看interview的