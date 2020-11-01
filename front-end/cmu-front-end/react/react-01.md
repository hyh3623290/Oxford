# 与Vue的异同

1. React用JSX拥抱js，Vue使用模板拥抱html

   开发体验的话Vue更像html的结构，上面是一个模板，然后中间是js代码，下面是style。React的话全是以点js结尾的，写Vue的话是以点Vue结尾的，所以写react就是写js，只不过jsx就是个语法糖而已，本质就是createElement函数。

2. react是函数式编程，Vue是声明式编程

   Vue变量得先声明好，你去修改data的值，我去监听，每个操作都是声明式的操作，data.a=100之类。React每一次修改都是setState修改

3. react更多需要自力更生，Vue把想要的都给你

   

   

   

   

   jsx较简洁，vue的话还有动态表达式：，jsx的话变量全是大括号，字符串的话就是一个双引号。就感觉react的动态静态的逻辑非常简单，只要动态就是大括号，静态就是字符串。还有jsx直接if，else就可以了，vue钟的话还v-if之类的



# ReactDOM

ReactDom提供了与浏览器交互的DOM功能，如：dom渲染。

```js
ReactDom.render(
  "新年好", // #1
  document.querySelector("#root"),
  () => {
    console.log("渲染完回调")
  }
)

```

1. 这里可以写react元素，如何创建一个react元素

   `React.createElement("h1", null, "hello react")` （type，props，children）



1. 

# className

```jsx
<p className="title"></p>
```

# style

```jsx
const styleData = { fontSize: '30px', color: 'red' }
<p style={styleData}></p>

<p style={{ fontSize: '30px', color: 'red' }}></p>

<p style="font-size: 30px;"></p>
// 静态style也非常好写，就跟html的一样
```



# 原生html

```jsx
const rawHtml = <div>123</div>
const rawHtmlData = {
  __html: rawHtml
}

<div dangeriousSetInnerHtml={rawHtmlData}></div>
// 必须要这么写
```



# 条件判断

```jsx
// if else
    
// ? :

// theme === 'black' && blackBtn 
```



# 渲染列表

```js
// map key
// map 返回一个新的数组，原数组没有变
```





# 事件

​	`bind(this)`的时候最好在constructor上bind，因为这样只会执行一次，虽然这样也可以`onClick={this.handleClick.bind(this)}`，但是这样的话每次点击都会执行一次。但其实最好还是箭头函数写

```jsx
<p onClick={this.handleClick}></p>

handleClick = (event) => {// #1
  event.preventDefault() // 阻止默认行为
  event.stopPropagation() // 阻止冒泡
  console.log(event)
  console.log(event.target) // target：当前元素触发
  console.log(event.currentTarget) // currentTarget：当前元素绑定
  console.log(event.nativeEvent.target)
  console.log(event.nativeEvent.currentTarget) // #3
}
```

1. 最推荐这种方法绑定this。参数的话如果什么都不传，第一个参数就是event参数，传的话最后一个参数是event

2. 虽然也正常输出当前元素，但是是假象，这个event不是原生的event，想获取原生的event可以通过`event.nativeEvent`。

   target：哪个元素触发的事件

   currentTarget：哪个元素绑定的事件

3. 会输出document

​    JSX 上写的事件并没有绑定在对应的真实 DOM 上，而是通过事件代理的方式，将所有的事件都统一绑定在了 `document` 上。这样的方式不仅减少了内存消耗，还能在组件挂载销毁时统一订阅和移除事件。

​	合成事件首先抹平了浏览器之间的兼容问题，另外这是一个跨浏览器原生事件包装器，赋予了跨浏览器开发的能力。

​	对于原生浏览器事件对象，浏览器会给每个监听器分配一个事件对象，如果有很多的事件监听，那么就会分配很多的事件对象，造成高额的内存分配问题。但是对于合成事件来说，有一个事件池专门管理它们的创建和销毁，当事件需要使用时，就会从池子里复用对象，事件回调结束后就会销毁事件对象上的属性，从而便于下次复用事件对象。

总结：

- 最佳绑定this
- 合成事件，原生事件
- target，currentTarget
- react事件机制及优点

## 合成事件过程

​	比如事件已经挂载完了，现在div触发事件，最终都会冒泡到document上。document会生成一个统一的react event。也就是合成事件的对象，然后它会再去派发事件。在合成事件这一层，它会再去派发事件，派发事件就是说我们能通过target事件知道谁触发了对吧，比如一个a标签，它所在的组件是哪一个我们就知道了，然后这个组件里面到底哪个绑定了onClick事件通过这种方式我们已经知道了。所以说通过判断它的target，判断target所在的组件，及它的事件绑定关系。我们就能触发到所有绑定了这个click事件的函数，然后再把这个合成事件的event传递进来去执行这个函数

## 合成事件的优点

1. 更好的兼容性和跨平台

   做了合成事件的机制以后因为是自己实现的，就摆脱了dom事件的那层逻辑（当然不是完全的摆脱），可以自己去实现一套逻辑，就可以更好的去保证兼容性和跨平台。如果是在网页上完全用dom事件的话，那假设现在要嵌套在移动平台上，那还要重写一套事件机制 

2. 全挂载到document上，减少内存消耗，避免频繁解绑

   首先事件挂载的越多，内存消耗越高（事件代理），如果有那么多组件的话可想而之。其次组件销毁的时候也没必要去解绑这个dom事件。所以减少和dom事件的依赖，以及减少和dom事件的绑定解绑的次数

3. 方便事件的统一管理



# 受控组件 - 表单

```jsx
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

  handleCheckboxChange = () => {
    this.setState({
      flag: !this.state.flag
    })
  }

  handleRadioChange = (e) => {
    this.setState({
      gender: e.target.value
    })
  }



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
```



# props

props类型检查

```js
import PropTypes from 'prop-types'
class MyComponent extends Component { 
    //... 
}

MyComponent.propTypes = {
  list: PropTypes.arrayOf(PropTypes.object).isRequired
  // 这个list必须得是一个数组, 数组每一项是个对象, 必传
}
```



## 传数据

## 传函数

```jsx
function Parent() {
  onFunc = (name) => {
    // ... 
  }
  return {
    <Child func={this.onFunc}/>  
  }
}

function Child(props) {
    handleClick = (123) => {
      this.props.func(123)
    }
    return <button onClick={this.handleClick}></button>
}

```



# setState

- **不可变值**

如何操作数组呢

```js
// 追加
let { array } = this.state
array: array.concat(10) // 不会影响原来的array
array: [...array, 10]

// 截取
array: array.slice(0, 3)

// 筛选
array: array.filter(k => k > 10)

// 其他操作
const arrayCoppy = array.slice() // 生成副本
arrayCoppy.balabaka // 修改完后再setState

```

对象呢

```js
obj: { ...this.state.obj, a: 10 }
obj: Object.assign({}, this.state.obj, {a: 10})
```



- **可能是异步的**

setTimeout里同步，自己定义的dom事件同步

- **可能会被合并**

```js
this.setState({
  count: this.state.count + 1
})
this.setState({
  count: this.state.count + 1
})
this.setState({
  count: this.state.count + 1
})
// 相当于
Object.assign(  
  {},
  { count: this.state.count + 1 },
  { count: this.state.count + 1 },
  { count: this.state.count + 1 },
)

// 每次setState的时候会做一个浅合并
this.state = Object.asign({
  name: 'kkb', // #1
  age: 10
}, {
  age: 18
})

// 传对象的话就会被合并，如果传入函数的话不会被合并, 肯定呀，函数怎么合并呀
this.setState((prevState) => ({ count: prevState.count + 1 }))
this.setState((prevState) => ({ count: prevState.count + 1 }))
this.setState((prevState) => ({ count: prevState.count + 1 }))
```

**如何拿到最新的state呢**

1. 传一个函数

```js
this.setState((state,props) => { this.setState(count: state.count + 1) })
// 这样就不合并了
```

2. setTimeout

```js
setTimeout(() => {
  console.log(this.state.count)
}, 0)
```

3. 原生事件中修改状态

```js
componentDidMount() {
  document.body.addEventListener('click', this.changeValue, false)
}
changeValue = () => {
  this.setState({count: this.state.count + 1})
  console.log(this.state.count)
}
```

异步（普通调用）- 同步（setTimeout，DOM事件）

合并（对象） - 不合并（函数）



**setState主流程**

​	setState后，newState会被存入一个pending队列，然后判断是否处于batch update这个机制中，是则保存组件于dirtyComponent中。否则遍历所有的dirtyCompoennts，调用updateComponent，然后去执行更新。所以setTimeout的时候被判断没有在那个batchUpdate中，就走了同步的方式去更新

​	首先在class里的所有函数，进入的时候都会遇到一个变量叫isBatchingUpdates，刚开始为true，函数执行完就赋值为false

​	dirtyCompoennt就是说现在state已经被更新的组件

**transaction事物机制**



# 非受控组件

ref - defaultValue，defaultChecked -  手动操作dom元素

```jsx
constructor() {
  let wrap = createRef()
}
<div ref={this.wrap}><div>
```

必须操作dom时使用，如file文件，必须通过input拿到

# createPortal

```jsx
import { createPortal } from 'react-dom'

export default class PortalTest extends React.Component {
  render() {
    return createPortal(
      <div className="modal">{this.props.children}</div>,
      document.body
    )
  }
}
// modal这个css不影响
```



# Context

```jsx
const ThemeContext = React.createContext('light')
// 当父组件的theme变化时，子组件就会变化
return (
  <ThemeContext.Provider value={this.state.theme}>
  	<Child />
  </ThemeContext.Provider>
)
```

```jsx
Child.contextType = ThemeContext // 静态属性

render() {
  const theme = this.context
  return (
    <div>{theme}</div>
  )
}
```

​	一般会单独创建一个context文件，再导出它，然后在需要的地方引入。如果是函数组件的话无法使用类的静态属性，此时只能用Consumer。此外如果是多个context的话也要用Consumer，只不过它会获取最近的consumer。

⚠️注意事项

以下这种方式，当每一次Provider重渲染时，里面的所有consumer都会重渲染，因为value总是被赋予新的对象

```jsx
<Provider value={{ a: 'a' }}>
	...
</Provider>
// {a: 'a'} === {a: 'a'} 返回false
```

为防止这种现象，应把value的状态放到state中

```jsx
this.state = {
  value: { a: 'a' }
}
<Provider value={this.state.value}>
	...
</Provider>
// 这样value没改变的话就不重新渲染
```



# 异步组件

组件比较大的时候，需要懒加载，也是性能优化的方案之一

```jsx
const Component = React.lazy(() => import('./Component'));

render() {
  return (
    <div>
      <p>xxx<p>
      <React.Suspense fallback={<div>loading...</div>}>
        <Component />
      </React.Suspense>          
    </div>
  )
}

```

# 性能优化

## shouldCompomentUpdates

SCU

```js
shouldCompomentUpdate(nextProps, nextState) {
  if(nextProps.count !== this.props.count) {
    return true // 可以渲染
  }
  return false // 不重复渲染
}
// _.isEqual(nextProps.list, this.props.list)
// lodash可以做深度比较，一次递归到底，耗费性能
```

为什么要用这个SCU呢？

​	首先父组件有更新，则子组件不管状态有没有变化都会无条件会更新（会执行didUpdate），这时候如果做一些判断会比较好。SCU默认会返回true。Vue就没有这个问题

**注意：不可变值**

​	当state直接改变值会怎么样呢 `count: ++this.state.count`，左右两边的count一样了，所以在SCU里不好判断。react为了避免有些人的这种不规范的写法，它就没有默认自己在SCU里做比较，如果默认自己比较的话就会有问题了。所以SCU必须要配合不可变值来写





## PureComponent和memo

​	在SCU中它会对state和props做一个浅比较，判断是否更新，浅比较适用大部分情况，尽量不要深度比较，写state的时候别写太深。深度比较的话一次递归到底，耗费性能。Vue的defineProperty也是递归到底的，所以data也不要太深（性能优化）



## immutable.js

​	彻底拥抱不可变值，基于共享数据（不是深拷贝），速度好

```js
const map1 = Immutable.Map({ a:1, b:2 })
const map2 = map1.set('b', 50) // 修改b
map1.get('b') // 2
map2.get('b') // 50
```



## 额外

- 异步组件



# 公共组件的抽离

## HOC

​	高阶组件就是一个工厂函数，参数为组件，返回值为新的组件的函数

​	为了提高组件的复用性，可测试性，就要保证组件功能单一性，但若要满足复杂需求就要扩展功能单一的组件，在React里就有了HOC的概念

```jsx
const withMouse = (Component) => {
  class withMouseComponent extends Component {
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
          <Component {...this.props} mouse={this.state}/>
        </div>
        
      )
    }
  }
  return withMouseComponent
}


const App = (props) => {
  const {x, y} = this.props
  const other = this.props.other
  return (
    <div>
      <h1>鼠标的位置是: ({x}, {y})</h1>
      <p>其他属性: {other}</p>
    </div>
  )
}

export default withMouse(App)
// Vue如何实现高阶组件,高阶组件不是一个功能，是一个设计模式
```

​	其实就是对一些数据的公共操作放到外面了，然后传给子组件

​	组件的开发的哲学就是一定要把UI抽象成最小的颗粒度，然后再通过高阶组件去拓展，这就是react开发的一个哲学。开发任何一个组件都是从里往外，从小到大。抽象的过程中想怎么架构，怎么组织。



### 链式调用

```js
const withLog = Comp => {
  return class extends React.Component {
    componentDidMount() {
      console.log('has mounted', this.props)
    }
    render() {
      return (
        <Comp />
      )
    }
  }
}
let P = withLog(withContent(Lesson))
```



​	组件的开发的哲学就是一定要把UI抽象成最小的颗粒度，然后再通过高阶组件去拓展，这就是react开发的一个哲学。开发任何一个组件都是从里往外，从小到大。抽象的过程中想怎么架构，怎么组织。

### 装饰器写法

```js
@withLog
@withContent
class Lesson2 extends React.Componen { // #1
  render() {
    return {
      <div>
        {props.stage} - {props.title}
      <div>       
    }
  }
}
```

注：不要在render里使用高阶组件的写法，

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

# key

​	组件每次更新时，会生成一个虚拟dom和原有的虚拟dom进行对比。

​	如果是批量生成一组元素，那就会根据key值进行对比，比如key为1的和key为1的进行对比，需要给它一个唯一标识。如果是index的话对性能不好，因为index是会变的，比如你删除了第一个，那其他的也都会变了，就会全部重新渲染一遍。所以如果列表中发生顺序等操作变化，key一定要用数据的id



# JSX本质

```jsx
React.createElement('tag', null, child1, child2, child3)
// child也是React.createElement(...)
// 返回vNode, tag即可以是html标签，也可以是组件名称
// 如何区分组件还是标签呢，组件名必须是大写
```

# 组件渲染过程和更新过程

React原理最后两集





# 组件复合

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

作用域插槽和hooks没看。



 



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

























































































































7-8























