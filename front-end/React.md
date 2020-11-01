# Fiber

​	Fiber 会使原本同步的渲染过程变成异步的。

​	在 React 16 之前，每当我们触发一次组件的更新，React 都会构建一棵新的虚拟 DOM 树，通过与上一次的虚拟 DOM 树进行 diff，实现对 DOM 的定向更新。这个过程，是一个递归的过程。

​	同步渲染的递归调用栈是非常深的，只有最底层的调用返回了，整个渲染过程才会开始逐层返回。这个漫长且不可打断的更新过程，将会带来用户体验层面的巨大风险：同步渲染一旦开始，便会牢牢抓住主线程不放，直到递归彻底完成。在这个过程中，浏览器没有办法处理任何渲染之外的事情，会进入一种无法处理用户交互的状态。因此若渲染时间稍微长一点，页面就会面临卡顿甚至卡死的风险。（也就是16ms内还在执行逻辑运算）

​	而 React 16 引入的 Fiber 架构，恰好能够解决掉这个风险：Fiber 会将一个大的更新任务拆解为许多个小任务。每当执行完一个小任务时，渲染线程都会把主线程交回去，看看有没有优先级更高的工作要处理，确保不会出现其他任务被“饿死”的情况，进而避免同步渲染带来的卡顿。在这个过程中，渲染线程不再“一去不回头”，而是可以被打断的，这就是所谓的“异步渲染”

​	Fiber 架构的重要特征就是可以被打断的异步渲染模式。但这个“打断”是有原则的，根据“能否被打断”这一标准，React 16 的生命周期被划分为了 render 和 commit 两个阶段，而 commit 阶段又被细分为了 pre-commit 和 commit。

- render 阶段：纯净且没有副作用，可能会被 React 暂停、终止或重新启动。
  - constructor - getDerived - shouldComp - render

- pre-commit 阶段：可以读取 DOM。
  - getSnap

- commit 阶段：可以使用 DOM，运行副作用，安排更新。
  - didMount - didUpdate - willUnMount

​    总的来说，render 阶段在执行过程中允许被打断，而 commit 阶段则总是同步执行的。

​    为什么这样设计呢？简单来说，由于 render 阶段的操作对用户来说其实是“不可见”的，所以就算打断再重启，对用户来说也是零感知。而 commit 阶段的操作则涉及真实 DOM 的渲染，再狂的框架也不敢在用户眼皮子底下胡乱更改视图，所以这个过程必须用同步渲染来求稳。

# React实现EventEmitter

发布订阅模式

```js
class myEventEmitter {
  constructor() {
    // eventMap 用来存储事件和监听函数之间的关系
    this.eventMap = {};
  }
  // type 这里就代表事件的名称
  on(type, handler) {
    // hanlder 必须是一个函数，如果不是直接报错
    if (!(handler instanceof Function)) {
      throw new Error("哥 你错了 请传一个函数");
    }
    // 判断 type 事件对应的队列是否存在
    if (!this.eventMap[type]) {
      // 若不存在，新建该队列
      this.eventMap[type] = [];
    }
    // 若存在，直接往队列里推入 handler
    this.eventMap[type].push(handler);
  }
  // 别忘了我们前面说过触发时是可以携带数据的，params 就是数据的载体
  emit(type, params) {
    // 假设该事件是有订阅的（对应的事件队列存在）
    if (this.eventMap[type]) {
      // 将事件队列里的 handler 依次执行出队
      this.eventMap[type].forEach((handler, index) => {
        // 注意别忘了读取 params
        handler(params);
      });
    }
  }
  off(type, handler) {
    if (this.eventMap[type]) {
      this.eventMap[type].splice(this.eventMap[type].indexOf(handler) >>> 0, 1);
    }
  }
}
```

# React

​	Context API 是 React 官方提供的一种组件树全局通信的方式。

​	在 React 16.3 之前，Context API 由于存在种种局限性，并不被 React 官方提倡使用，开发者更多的是把它作为一个概念来探讨。而从 v 16.3.0 开始，React 对 Context API 进行了改进，新的 Context API 具备更强的可用性。

​	Context API 有 3 个关键的要素：React.createContext、Provider、Consumer。

​	**React.createContext**，作用是创建一个 context 对象。下面是一个简单的用法示范

```js
const AppContext = React.createContext()
```

​	注意，在创建的过程中，我们可以选择性地传入一个 defaultValue：

```js
const AppContext = React.createContext(defaultValue)
```

​	从创建出的 context 对象中，我们可以读取到 Provider 和 Consumer：

```js
const { Provider, Consumer } = AppContext
```

​	我们使用 **Provider** 对组件树中的根组件进行包裹，然后传入名为“value”的属性，这个 value 就是后续在组件树中流动的“数据”，它可以被 Consumer 消费。使用示例如下：

```jsx
<Provider value={title: this.state.title, content: this.state.content}>
  <Title />
  <Content />
</Provider>
```

​	**Consumer**，顾名思义就是“数据的消费者”，它可以读取 Provider 下发下来的数据。

​	其特点是需要接收一个函数作为子元素，这个函数需要返回一个组件。像这样：

```jsx
<Consumer>
  {value => <div>{value.title}</div>}
</Consumer>
```

​	注意，当 Consumer 没有对应的 Provider 时，value 参数会直接取创建 context 时传递给 createContext 的 defaultValue。

**以前的context：**

​	首先映入眼帘的第一个问题是**代码不够优雅**：一眼望去，你很难迅速辨别出谁是 Provider、谁是 Consumer。同时这琐碎的属性设置和 API 编写过程，也足够我们写代码的时候“喝一壶了”。总而言之，从编码形态的角度来说，“过时的” Context API 和新 Context API 相去甚远。

​	如果组件提供的一个Context发生了变化，而中间父组件的 shouldComponentUpdate 返回 false，**那么使用到该值的后代组件不会进行更新**。使用了 Context 的组件则完全失控，所以基本上没有办法能够可靠的更新 Context。[这篇博客文章](https://medium.com/@mweststrate/how-to-safely-use-react-context-b7e343eff076)很好地解释了为何会出现此类问题，以及你该如何规避它。  ——React 官方

​	新的 Context API 改进了这一点：**即便组件的 shouldComponentUpdate 返回 false，它仍然可以“穿透”组件继续向后代组件进行传播**，**进而确保了数据生产者和数据消费者之间数据的一致性**。再加上更加“好看”的语义化的声明式写法，新版 Context API 终于顺利地摘掉了“试验性 API”的帽子，成了一种确实可行的 React 组件间通信解决方案。

​	对于简单的跨层级组件通信，我们可以使用发布-订阅模式或者 Context API 来搞定。但是随着应用的复杂度不断提升，需要维护的状态越来越多，组件间的关系也越来越难以处理的时候，我们就需要请出 Redux 来帮忙了。

# Redux

​	Redux 主要由三部分组成：**store、reducer 和 action**。我们先来看看它们各自代表什么：

​	**在 Redux 的整个工作过程中，数据流是严格单向的**。这一点一定一定要背下来，面试的时候也一定一定要记得说——不管面试官问的是 Redux 的设计思想还是工作流还是别的什么概念性的知识，开局先放这句话，准没错。

**1. 使用 createStore 来完成 store 对象的创建**

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

​	这其中一般来说，只有 reducer 是你不得不传的。下面我们就看看 reducer 的编码形态是什么样的。

**2. reducer 的作用是将新的 state 返回给 store**

​	一个 reducer 一定是一个纯函数，它可以有各种各样的内在逻辑，但它最终一定要返回一个 state：

```js
const reducer = (state, action) => {
    // 此处是各种样的 state处理逻辑
    return new_state
}
```

​	当我们基于某个 reducer 去创建 store 的时候，其实就是给这个 store 指定了一套更新规则：

**3. action 的作用是通知 reducer “让改变发生”**

**4. 派发 action，靠的是 dispatch**

# React Hooks

​	动笔写 React-Hooks 之前，我发现许多人对这块的知识非常不自信，至少在面试场景下，几乎没有几个人在聊到 React-Hooks 的时候，能像聊 Diff 算法、Fiber 架构一样滔滔不绝、言之有物。后来我仔细反思了一下，认为问题应该出在学习姿势上。

​	提起 React-Hooks，可能很多人的第一反应，都会是 useState、useEffect、useContext 这些琐碎且繁多的 API。似乎 React-Hooks 就是一坨没有感情的工具性代码，压根没有啥玄妙的东西在里面，那些大厂面试官天天让咱聊 React-Hooks，到底是想听啥呢？

​	React-Hooks 自 React 16.8 以来才真正被推而广之，对我们每一个老 React 开发来说，它都是一个新事物。如果在认知它的过程当中，我们能够遵循“Why→What→How”这样的一个学习法则，并且以此为线索，梳理出属于自己的完整知识链路。那么我相信，面对再刁钻的面试官，你都可以做到心中有数、对答如流。

​	开篇我们先来聊“Why”。React-Hooks 这个东西比较特别，它是 React 团队在真刀真枪的 React 组件开发实践中，逐渐认知到的一个改进点，这背后其实涉及对**类组件**和**函数组件**两种组件形式的思考和侧重。

**函数组件与类组件的对比：无关“优劣”，只谈“不同”**

- 类组件中可以获取到实例化后的 this，并基于这个 this 做各种各样的事情，而函数组件不可以；
- Balabala...

​    有一个非常容易被大家忽视、也极少有人能真正理解到的知识点，我在这里要着重讲一下。这个知识点缘起于 React 作者 Dan 早期特意为类组件和函数组件写过的[一篇非常棒的对比文章](https://overreacted.io/how-are-function-components-different-from-classes/)，这篇文章很长，但是通篇都在论证这一句话：

> **函数组件会捕获 render 内部的状态，这是两类组件最大的不同。**

​	**类组件和函数组件之间，纵有千差万别，但最不能够被我们忽视掉的，是心智模式层面的差异**，是面向对象和函数式编程这两套不同的设计思想之间的差异。说得更具体一点，**函数组件更加契合 React 框架的设计理念**。何出此言？不要忘了这个赫赫有名的 React 公式：

​	`UI = render(data)`

​	不夸张地说，**React 组件本身的定位就是函数，一个吃进数据、吐出 UI 的函数**。	

**Hooks 的本质：一套能够使函数组件更强大、更灵活的“钩子”**

​	**如果说函数组件是一台轻巧的快艇，那么 React-Hooks 就是一个内容丰富的零部件箱**。“重装战舰”所预置的那些设备，这个箱子里基本全都有，同时它还不强制你全都要，而是**允许你自由地选择和使用你需要的那些能力**，然后将这些能力以 Hook（钩子）的形式“钩”进你的组件里，从而定制出一个最适合你的“专属战舰”。



# CSS

## 选择器

**元素选择符，通用选择符，类选择符，ID选择符**

### 群组选择符

我想让多个元素都显示为红色

```css
h1, p {
  color: red;
}
```

⚠️注：有些元素在有些旧的浏览器上无法识别，如ie8不识别`<main>`元素，我们可以在dom上创建元素，让浏览器知道元素的存在，`document.createElement('main')`

```css
p.warning.help {
  color: red;
}
```

表示匹配类名为warning且help的p元素

### 属性选择符

```less
h1[class] {
  color: red;
}
// 具有class属性的h1标签
```

### 后代选择符

```less
h1 p {
  color: red;
}
```

### 子元素选择符

```less
h1 > p {
	color: red;
}
```

### 紧邻兄弟选择符

```less
h1 + p {
  color: red;
}
// p和h1有相同的父元素，且p紧挨在h1后面，也就是只选择后面这个
// p的颜色红了
```

### 兄弟选择符

```scss
h1 ~ p {
  // ...
}
// h1后面只要是兄弟的，且是p元素的
// p的颜色红了
```

### 伪类选择符

```scss
a:link:hover {
  color: red;
}
```

​	**伪类始终指代所依附的元素**，比如我想选择我的第一个孩子

```scss
#hyh > :first-child { }

// 大部分人会写成这样
#hyh:first-child
// 这个的意思是我，且我是我父级的第一个子集，它选中的是我我我
```



```less
// 选择空元素
p:empty {
  display: none; // 禁止显示空段落
}

// 选择唯一的子代
img:only-child // 只有标签可以，试了一下类名不香
```

# 期约与异步函数

​	

