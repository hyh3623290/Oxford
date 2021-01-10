# JSX

## 自动展开数组

```jsx
const arr = [<p>111</p>, <p>111</p>, <p>111</p>]
const element = (
  <div>{arr}</div>
);

// 解析后
<div>
	<p>111</p>
  <p>111</p>
  <p>111</p>
</div>
```

## 事件传参

```js
onClick={e => this.handleClick('参数', e)}
```

## 样式

​	我们希望这个样式只用于这个组件，所以样式文件的名字是要有一些约定的，比如这个文件叫button.js，则这个css叫Button.module.css。

```js
// 1. 外链样式
import styles from './Button.module.css'

// Button.js
className={styles.error}

// 2. 全局样式
import './styles.css'
```

## ref

🍇 createRef方法

```jsx
// constructor
this.inputRef = React.createRef()

// render
<input ref={this.inputRef}>
```

​	这样`this.inputRef`就会有一个current属性，也就是对应的dom对象

​	它其实也可以获取组件实例，然后可以调用组件实例里面的方法

🍇 函数参数

```js
// render
<input ref={input => {this.input = input}}>
```

​	这个参数里的input就是对应的当前这个input元素

# 组件

## props默认值

```js
class App extends Component {
  static defaultProps = { a: '1' }
}

function App() {}
App.defaultProps = {}
```

 单向数据流的优点：数据流动变得可预测，使得定位程序错误变得简单



## setState

​	直接修改state的话对应的dom不会被更新

​	双向数据绑定就是表单元素和state之间



## Context

```js
// UserContext.js
const UserContext = React.createContext('default value')
const UserProvider = UserContext.Provider
const UserConsumer = UserContext.Consumer

export { UserProvider, UserConsumer }
```



```jsx
// App.js
import { UserProvider } from './UserContext.js'
render() {
  return (
  	<UserProvider value="hello context">
      <Component />
    <UserProvider>
  )
}
```



```jsx
// Component.js
import { UserConsumer } from './UserContext.js'
render() {
  return (
  	<UserConsumer>
      {
        (username) => {
          return <div>{username}</div>
        }
      }
    <UserConsumer>
  )
};
      
// 另一种用法
class Component {
  static contextType = UserContext
}
```



## 表单

1. 受控表单

   就是跟state进行双向绑定

2. 非受控表单

   就是通过ref获取

























