# 1

# Vue生命周期

​	new Vue实例之后，刚开始只有默认的一些生命周期函数和默认的事件，其他东西都未创建

​	然后执行的是beforeCreate，表示实例被创建出来之前，这个时候是获取不到props或者data中的数据的，然后执行created函数（实例初始化完了），到这一步就可以访问到之前不能访问的数据了，但是这个时候组件还未被挂载，所以是看不到的。

​	接下来是beforeMount钩子函数，开始创建虚拟dom，最后执行mounted钩子函数，并将虚拟dom渲染为真实dom并且渲染数据。组件中如果有子组件，会递归挂载子组件，只有当所有子组件都挂载完后才会执行根组件的挂载钩子

​	接下来就是数据更新前的beforeUpdate和更新完后的updated，这两个钩子函数没什么好说的，就是分别在数据更新前和更新后会调用。

​	最后就是beforeDestroy和destroyed，前者适合移除一些事件、定时器等等，否则可能会引起内存泄露的问题。然后进行一系列的销毁操作，如果有子组件的话，也会递归销毁子组件，所有子组件都销毁完毕后才会执行根组件的 `destroyed` 钩子函数



# Vue组件通信

​	首先最简单的就是父传子，用props，子传父的话就是$emit触发一个事件给父组件传递数据。此外我们还可以用$parent和$children来访问组件实例里面的方法和数据，所以也可以传递数据，ref也跟这个方法差不多，也可以获取组件实例来传递数据。

​	兄弟组件直接通信的话其实可以用他们共同的父组件$parent来触发和监听事件来传递数据

​	有时候需要跨级的话可以用provide和inject，给祖先的话可以这样

```js
function dispatch(eventName, data) {
  let parent = this.$parent
  while(parent) {
    if(parent) {
      parent.$emit(eventName, data)
      parent = parent.$parent
    } else {
      break
    }
  }
}
// 这样任何一个关心某事件的祖辈全部都能听到此事件
// 某后代组件
<h1 @click="dispatch('hello', 'hello world')"/>xxx<h1>
// App.vue
this.$on('hello', this.sayHello)
```

​	如果是任意两个组件的话可以用事件总线，或者是vuex都可以



# React组件通信

​	首先是父子通信，父组件通过 `props` 传递数据给子组件，子组件通过调用父组件传来的函数传递数据给父组件，这两种方式是最常用的父子通信实现办法。

​	如果是兄弟组件通信，对于这种情况可以通过共同的父组件来管理状态和事件函数。比如说其中一个兄弟组件调用父组件传递过来的事件函数修改父组件中的状态，然后父组件将状态传递给另一个兄弟组件。

​	如果是跨多层级的话可以用Context

```js
// context.js
import React, { createContext } from 'react'
let context = createContext()
let { Consumer, Provider } = context

export default context;
export { Consumer, Provider }

// App.js
import { Provider } from './context'
class App extends Component{
  render() {
    return (
      <Provider value={{info: "999"}}>
        <Child />
      <Provider>
    )
  }
}

// Child1.js
import { Consumer } from './context'
class Child extends Component{
  render() {
    return (
      <div>
        <Consumer>
          { // 需要写在插值里
            (value) => { 
              console.log(value);
              return value.info
            }
          }
        <Consumer>
      <div>
    )
  }
}

// Child2.js
import context, { Consumer } from './context'
class Child extends Component{
  static contextType = context // #2
  render() {
    console.log(this.context) // 拿到数据
    return (
      <div>
        
      <div>
    )
  }
}
// Child.contextType = context #1 #3
```

1. class.contextType = Context
2. static contextType = Context
3. 这个时候打印该组件会发现组件多了一个context属性，就可以拿到该数据了
4. context一般不用在开发里，一般组件库需要用

# Redux流程

​	通过reducer创建一个store，`let store = createStore(reducerName)`

​	通过subscribe实现订阅，`store.subscribe(() => {console.log(store.getState)})`

​	通过dispatch来修改状态，`store.dispatch({ type: 'ADD' })`，之后就会被`reducer`接收，通过`actionType`来做处理，返回一个`newState`

**react-redux**

​	Provider

​	connect

​	mapStateToProps，mapDispatchToProps

```js
connect()(componentName) // connect是一个高阶组件
```



# 2

# Vue路由导航守卫

## 全局

```js
const router = ({
  ...
})
    
router.beforeEach(function(to, from, next) {
  ...
  next()
})
router.afterEach(function(to, from) {
  
})
```



## 路由

路由独享的守卫

```js
const router = new VueRouter({
  routes:[
    {
      path: '/',
      component: Home,
      beforeEnter: (to, from, next) => {
        ...
      }
    }
  ]
})
```

## 组件

组件内使用

```js
beforeRouteEnter(to, from, next) {
  ...  
}
beforeRouteUpdate(to, from, next) { // #1
  ...
}
beforeRouteLeave(to, from, next) {
  ...
}
```

1. 在当前路由改变，但是该组件被复用时调用

   举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用



# Vuex的mapState

使用mapState辅助函数帮助我们生成计算属性

```js
computed: {
  title() {
    return this.$store.state.title
  },
  content() {
    return this.$store.state.content
  }
}
// 等价于
computed: {
  mapState: ['title', 'content']
}

// 实际开发这么写
computed: {
  ...mapState({
    title: 'title',
    content: 'content'
  })
  // 其他的内部的computed
}
```



# 垂直居中

```html
<div>
  <span>文字</span>
</div>
```

- **padding**

```css
div {
  width: 200px;
  padding-top: 50px;
  padding-bottom: 50px;
}
```

缺点：父元素不能设置固定高度



- **line-height**

```css
span {
  line-height: 100px; /* #1 */
}

p {
  line-height: 100px;
  margin: 0;  /* #2 */
}
```

1. 设置为父级的高度
2. p元素有默认高度
3. 一般用于单行文字



- **table**

```css
ul {
  display: table;
}
li {
  display: table-cell;
  vertical-align: middle
}
```



- **grid**

```css
ul {
  display: grid;
  grid-template-columns: repeat(6, 1fr)
  align-items: center;
  justify-content: center;
}
```



- **absolute**

```css
i {
  height: 50px;
  width: 50px;
  position: absolute;
  top: 50%;
  margin-top: -25px; /* #1 */
}
```

1. 如果不知道高度，`transform: translateY(-50%)`
2. 缺点时脱离文档流

- **inline-block**

```css
div.search {
  display: inline-block;
  vertical-align: middle;
}
div::after { /* #1 */
  content: '';
  display: inline-block;
  height: 100%;
  width: 0;
  vertical-align: middle;
}
```

1. 没有其他单元格可以做对比，将另一个元素高度设为100%，这一行的对齐暂时就要听它的

# 水平居中

- **text-align**

```css
h1 {
  text-align: center;
}
p {
  text-align: center;
}
span {
  text-align: center;
}
```

1. 前两者生效，后者不生效，后者为行内元素，需要在父级设置text-align才可以

块级元素居中

- **margin：0 auto**

- **脱离文档流时**

  如浮动和绝对定位，方式同垂直居中，`left: 50%; transform: translateX(50%)`

  此外：

  ```css
  span {
    position: absolute
    left: 0; /* #1 */
    right: 0;
    margin: 0 auto;
  }
  ```

  1. left和right设为0，让文档知道我们并不想左右定位，如果行内元素设置绝对定位，我们就可以想块级元素一样设置宽度和高度，可以用普通流块级元素的方法来进行居中

















