[TOC]



# CSS Module

​	实现样式的模块化，需使用 `[name].module.scss` 方式命名

```jsx
import styles from './index.module.scss'

export default function App() {
  return (
    <div className={styles.title}>hello world</div>
  )
}
```



# JSX

## 出现原因

​	虚拟 DOM 也可以通过 JavaScript 来创建

```js
const ele = React.createElement('div', null, 'hello, world')
```

​	但这种代码可读性比较差，于是就有了 JSX 

​	Babel 会把 JSX 转译成一个名为 `React.createElement()` 的函数调用

# 函数式组件

​	函数式创建的组件代码简洁，专注于 render，且组件不需要被实例化，整体渲染性能得到了提升，且视图和数据解耦分离，输出只取决于输入。但是，它无法拥有自己的 state，只能通过 props 获取属性内容并实现组件的更新，无生命周期。



























