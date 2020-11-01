# Vue

组件通信

## $attrs/$listeners

​	包含了父作用域中不作为 prop 被识别 (且获取) 的特性绑定 ( class 和 style 除外)。当一个组件没有 声明任何 prop 时，这里会包含所有父作用域的绑定 ( class 和 style 除外)，并且可以通过 vbind="$attrs" 传入内部组件——在创建高级别的组件时非常有用。

​	凡是子组件里没有通过props的方式去声明，你还通过老爹的方式传进来了，它就会被收纳到这个attrs里头

​	老爹写了个事件，挂在了子组件上。子组件就可以用listeners监听

```jsx
// parent
<Cmp @click="onClick"/>
onClick() {
  console.log('parent func')
}

// child
<p v-on="$listeners"></p> // 事件会被展开
```

## 插槽

```jsx
// 匿名
<template>
  <slot></slot>
</template>
<Cmp>
  <p>content</p>
</Cmp>

// 具名
<template>
  <slot name="foo"></slot>
</template>
<Cmp>
  <p v-slot:foo>content</p>
</Cmp>

// 作用域 - 我定义slot的时候还想把里面的数据bar往外传
<template>
  <slot :foo="bar"></slot>
</template>
<Cmp>
  <p v-slot:k="ctx">content:{{ctx.foo}}</p>
</Cmp>
```

## 通用表单组件设计

​	收集数据、校验数据并提交。

**需求分析**

​	实现KForm ：指定数据、校验规则 

​	KformItem ：执行校验 显示错误信息 

​	KInput ：维护数据

**设计**

​	把数据，校验规则之类放到最外层，方便进行一个全局的校验，

​	最外层form应依次执行formItem的校验方法，同时把自身整个传下去。这样item可以拿到整个form实例，进一步获取属于自己的数据并校验，最里面input负责派发事件。

### index.vue

在index里提供model，rules，并执行form的validate方法

```html
<template>
  <div>
    <h3>表单</h3>
    <hr />
    <KForm :model="model" :rules="rules" ref="loginForm">
      <KFormItem label="用户名" prop="username">
        <KInput v-model="model.username" placeholder="请输入用户名"></KInput>
      </KFormItem>
      <KFormItem label="密码" prop="password">
        <KInput type="password" v-model="model.password" placeholder="请输入密码"></KInput>
      </KFormItem>
      <KFormItem>
        <button @click="onLogin">登录</button>
      </KFormItem>
    </KForm>
  </div>
</template>

<script>
import KInput from "@/components/form/KInput.vue";
import KFormItem from "@/components/form/KFormItem.vue";
import KForm from "@/components/form/KForm.vue";
import Notice from "@/components/Notice.vue";
import create from "@/utils/create";

export default {
  components: {
    KInput,
    KFormItem,
    KForm
  },
  data() {
    return {
      model: { username: "tom", password: "" },
      rules: {
        username: [{ required: true, message: "请输入用户名" }],
        password: [{ required: true, message: "请输入密码" }]
      }
    };
  },
  methods: {
    onLogin() {
      this.$refs["loginForm"].validate(isValid => {
        const notice = create(Notice, {
          title: "杨哥喊你来搬砖",
          message: isValid ? "登录去！" : "有错改去！"
        });
        notice.show();
        setTimeout(() => {
          notice.hide();
        }, 2000);
        
      });
    }
  }
};
</script>

<style lang="scss" scoped>
</style>
```



### Form.vue

1. 需要一个插槽
2. provide让子组件可以获取这个表单
3. 接收model和rules
4. 提供校验方法执行孩子的formItem的校验

```html
<template>
  <div>
    <slot></slot>
  </div>
</template>

<script>
export default {
  provide() {
    return {
      form: this
    };
  },
  props: {
    model: {
      type: Object,
      required: true
    },
    rules: { type: Object }
  },
  methods: {
    validate(cb) { // reject
      // 全局校验
      // 1.不是所有项都需要校验
      //   tasks是promise数组
      const tasks = this.$children
        .filter(item => item.prop)
        .map(item => item.validate());
      //   所有必须全通过
      Promise.all(tasks)
        .then(() => cb(true))
        .catch(() => cb(false));
    }
  }
};
</script>

<style lang="scss" scoped>
</style>
```



### FormItem

1. 显示label
2. 插槽放input
3. 需要提示错误信息
4. 需要接收form拿到它的model和rules
5. 监听input的validate触发自身校验方法
6. 通过schema以及传入数据和规则来进行校验

监听子组件的validate事件触发自身的 validate方法，使用schema，通过prop知道来校验哪个属性

```html
<template>
  <div>
    <!-- label -->
    <label v-if="label">{{label}}</label>
    <slot></slot>
    <!-- 校验信息 -->
    <p v-if="errorMessage">{{errorMessage}}</p>
  </div>
</template>

<script>
// npm i async-validator -S
import Schema from "async-validator";
export default {
  inject: ["form"],
  data() {
    return {
      errorMessage: ""
    };
  },
  props: {
    label: {
      type: String,
      default: ""
    },
    prop: {
      // 用于获取指定字段值和校验规则
      type: String,
      default: ""
    }
  },
  mounted() {
    this.$on("validate", () => {
      this.validate();
    });
  },
  methods: {
    validate() {
      // 1.获取值和校验规则
      const rules = this.form.rules[this.prop];
      const value = this.form.model[this.prop];
      // 2.创建Schema实例 {username: rules}
      const schema = new Schema({ [this.prop]: rules });
      // 3.执行校验，校验对象,回调函数
      // validate返回校验结果Promise
      return schema.validate({ [this.prop]: value }, errors => {
        if (errors) {
          // 显示错误
          this.errorMessage = errors[0].message;
        } else {
          this.errorMessage = "";
        }
      });
    }
  }
};
</script>
```



### input

接收父级的value并且在触发input的事件时传入value给父级，触发父级的validate方法

```html
<template>
  <div>
    <!-- 双绑：:value @input -->
    <!-- v-bind会展开$attrs对象 -->
    <input :value="value" @input="onInput" v-bind="$attrs">
  </div>
</template>

<script>
export default {
  // 1.双绑
  // 2.通知校验
  inheritAttrs: false,
  props: {
    value: {
      type: String,
      default: ""
    }
  },
  mounted() {
    console.log(this.$attrs);
  },
  methods: {
    onInput(e) {
      // 仅派发事件
      this.$emit("input", e.target.value);

      // 通知校验
      this.$parent.$emit("validate");
    }
  }
};
</script>

<style lang="scss" scoped>
</style>
```

## 弹窗组件设计

​	弹窗这类组件的特点是它们在当前vue实例之外独立存在，通常挂载于body；它们是通过JS动态创建 的，不需要在任何组件中声明。常见使用姿势：

```js
import Notice from '../create/Notice'
import create from '../create/create'
const notice = this.$create(Notice, {
  title: '来搬砖',
  message: '提示信息',
  duration: 1000
})
notice.show();
```



```js
import Vue from 'vue'

function create(Cmp, props) {
  const vm = new Vue({
    render(h) {
      return h(Cmp, {props})
    }
  }).$mount()

  document.body.appendChild(vm.$el)
  const comp = vm.$children[0]
  comp.remove = () => {
    document.body.removeChild(vm.$el);
    vm.$destroy();
  }
  return comp
}

export default create
```

Homework - 下一节 - 待了解（render，$mount，Vue.use）

Element 源码 - pakeages文件夹很重要几乎所有源码都在这里，src其实只是暴露一些接口的地方。

emitter做广播，冒泡来代替之前表单的$parent,$children。为了程序的鲁棒性。（dispatch方法）

## VueRouter

**需求分析** 

- 作为一个插件存在：实现VueRouter类和install方法 
- 实现两个全局组件：router-view用于显示匹配组件内容，router-link用于跳转 
- 监控url变化：监听hashchange或popstate事件 
- 响应最新url：创建一个响应式的属性current，当它改变时获取对应组件并显示

​    使用VueRouter第一步就是`Vue.use(VueRouter)`（应用插件）第二步是创建Router实例，第三步就是将它装到实例里面

**VueRouter使用方法**

router.js

```js
import Vue from 'vue'
import VueRouter from './kvue-router'
import Home from '../views/Home.vue'

// 1.应用插件
Vue.use(VueRouter)

const routes = [
  {
    path: '/',
    name: 'home',
    component: Home
  },
  {
    path: '/about',
    name: 'about',
    component: () => import(/* webpackChunkName: "about" */ '../views/About.vue')
  }
]

// 2.创建实例
const router = new VueRouter({
  mode: 'history',
  base: process.env.BASE_URL,
  routes
})

export default router
```

main.js

```js
// 3.引入，挂载这个组件其实是为了可以在所有实例中都能使用到这个router
import router from './router'
new Vue({
  store,
  render: h => h(App)
}).$mount('#app')
```

39.36

**实现router**

```js
import Link from './krouter-link'
import View from './krouter-view'

let Vue;

// 1.实现一个插件：挂载$router

class KVueRouter {
  constructor(options) {
    this.$options = options
    console.log(this.$options);
    

    // 需要创建响应式的current属性
    // 利用Vue提供的defineReactive做响应化
    // 这样将来current变化的时候，依赖的组件会重新render
    Vue.util.defineReactive(this, 'current', '/')

    // this.app = new Vue({
    //   data() {
    //     return {
    //       current: '/'
    //     }
    //   }
    // })
    
    // 监控url变化
    window.addEventListener('hashchange', this.onHashChange.bind(this))
    window.addEventListener('load', this.onHashChange.bind(this))


    // 创建一个路由映射表
    this.routeMap = {}
    options.routes.forEach(route => {
      this.routeMap[route.path] = route
    })
  }

  onHashChange() {
    console.log(window.location.hash);

    this.current = window.location.hash.slice(1)
  }
}

KVueRouter.install = function (_Vue) {
  // 保存构造函数，在KVueRouter里面使用
  Vue = _Vue;

  // 挂载$router
  // 怎么获取根实例中的router选项
  Vue.mixin({
    beforeCreate() {
      // 确保根实例的时候才执行
      if (this.$options.router) {
        Vue.prototype.$router = this.$options.router
      }

    }
  })


  // 任务2：实现两个全局组件router-link和router-view
  Vue.component('router-link', Link)
  Vue.component('router-view', View)
}

export default KVueRouter
```



# React

## jsx

​	JSX 是 JavaScript 的一种语法扩展，它和模板语言很接近，但是它充分具备 JavaScript 的能力。

​	Facebook 公司给 JSX 的定位是 JavaScript 的“扩展”，而非 JavaScript 的“某个版本”，这就直接决定了浏览器并不会像天然支持 JavaScript 一样地支持 JSX。那么，JSX 的语法是如何在 JavaScript 中生效的呢？[React 官网](https://reactjs.org/docs/glossary.html#jsx)其实早已给过我们线索：JSX 会被编译为 React.createElement()， React.createElement() 将返回一个叫作“React Element”的 JS 对象

​	Babel 是一个工具链，主要用于将 ECMAScript 2015+ 版本的代码转换为向后兼容的 JavaScript 语法，以便能够运行在当前和旧版本的浏览器或其他环境中。—— Babel 官网

**JSX 是如何映射为 DOM 的：起底 createElement 源码**

​	在分析开始之前，你可以先尝试阅读我追加进源码中的逐行代码解析，大致理解 createElement 中每一行代码的作用：

```js
/**
 101. React的创建元素方法
 */
export function createElement(type, config, children) {
  // propName 变量用于储存后面需要用到的元素属性
  let propName; 
  // props 变量用于储存元素属性的键值对集合
  const props = {}; 
  // key、ref、self、source 均为 React 元素的属性，此处不必深究
  let key = null;
  let ref = null; 
  let self = null; 
  let source = null; 
  // config 对象中存储的是元素的属性
  if (config != null) { 
    // 进来之后做的第一件事，是依次对 ref、key、self 和 source 属性赋值
    if (hasValidRef(config)) {
      ref = config.ref;
    }
    // 此处将 key 值字符串化

    if (hasValidKey(config)) {

      key = '' + config.key; 
    }
    self = config.__self === undefined ? null : config.__self;
    source = config.__source === undefined ? null : config.__source;
    // 接着就是要把 config 里面的属性都一个一个挪到 props 这个之前声明好的对象里面
    for (propName in config) {
      if (
        // 筛选出可以提进 props 对象里的属性
        hasOwnProperty.call(config, propName) &&  /
        !RESERVED_PROPS.hasOwnProperty(propName) 
      ) {
        props[propName] = config[propName]; 
      }
    }
  }
  // childrenLength 指的是当前元素的子元素的个数，减去的 2 是 type 和 config 两个参数占用的长度
  const childrenLength = arguments.length - 2; 
  // 如果抛去type和config，就只剩下一个参数，一般意味着文本节点出现了
  if (childrenLength === 1) { 
    // 直接把这个参数的值赋给props.children
    props.children = children; 
    // 处理嵌套多个子元素的情况
  } else if (childrenLength > 1) { 
    // 声明一个子元素数组
    const childArray = Array(childrenLength); 
    // 把子元素推进数组里
    for (let i = 0; i < childrenLength; i++) { 
      childArray[i] = arguments[i + 2];
    }
    // 最后把这个数组赋值给props.children
    props.children = childArray; 
  } 
  // 处理 defaultProps
  if (type && type.defaultProps) {
    const defaultProps = type.defaultProps;
    for (propName in defaultProps) { 
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }
  // 最后返回一个调用ReactElement执行方法，并传入刚才处理过的参数
  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
}
```

​	createElement 执行到最后会 return 一个针对 ReactElement 的调用。这里关于 ReactElement，我依然先给出源码 + 注释形式的解析：

```js
const ReactElement = function(type, key, ref, self, source, owner, props) 
  const element = {
    // REACT_ELEMENT_TYPE是一个常量，用来标识该对象是一个ReactElement
    $$typeof: REACT_ELEMENT_TYPE,
    // 内置属性赋值
    type: type,
    key: key,
    ref: ref,
    props: props,
    // 记录创造该元素的组件
    _owner: owner,
  };
  // 
  if (__DEV__) {
    // 这里是一些针对 __DEV__ 环境下的处理，对于大家理解主要逻辑意义不大，此处我直接省略掉，以免混淆视听
  }
  return element;
};
```

​	ReactElement 的代码出乎意料的简短，从逻辑上我们可以看出，ReactElement 其实只做了一件事情，那就是“**创建**”，说得更精确一点，是“**组装**”：ReactElement 把传入的参数按照一定的规范，“组装”进了 element 对象里，并把它返回给了 React.createElement，最终 React.createElement 又把它交回到了开发者手中。

​	既然是“虚拟 DOM”，那就意味着和渲染到页面上的真实 DOM 之间还有一些距离，这个“距离”，就是由大家喜闻乐见的**ReactDOM.render**方法来填补的。

​	在每一个 React 项目的入口文件中，都少不了对 React.render 函数的调用。下面我简单介绍下 ReactDOM.render 方法的入参规则：

```jsx
ReactDOM.render(
    // 需要渲染的元素（ReactElement）
    element, 
    // 元素挂载的目标容器（一个真实DOM）
    container,
    // 回调函数，可选参数，可以用来处理渲染结束后的逻辑
    [callback]
)
```





## 数据通信

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









------------------------------------