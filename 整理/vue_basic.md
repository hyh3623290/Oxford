#   组件

## 根组件

​	应用的最顶层组件，一般一个独立应用有一个根组件

```jsx
<div id="#app"></div>

let app = new Vue({
  // 配置 options
  template:`<div></div>`
})

app.$mount('#app')
```

​	注意，根组件不能挂到html或者body上，因为他会替换

## 可复用的功能组件

```jsx
Vue.component('tab', {
  template: `<div>选项卡</div>`
})
```

​	就可以直接复用了

```jsx
<tab />
```



## 组件内容渲染

- template选项
- render选项

​    根组件如果不用$mount，就也可以用el选项，用el选项就是在构建的时候同时进行挂载，$mount就是可以提供延迟挂载的功能，挂载的时机可以自己选

```js
let app = new Vue({
  // 配置 options
  el: '#app'
  template:`<div></div>`
})

// app.$mount('#app')
```

​	如果提供了`el`，而没有提供`template`，那么会自动把el的`innerHTML`作为`template`

```jsx
<div id="#app">
  <h1>123</h1>
  <tab />
</div>

let app = new Vue({
  el: '#app'
})
```

​	render方法会少一个模版到虚拟dom的过程



## data

```js
data: {
  title: '123'
}
app.title = 111 // 视图即可变化 或者
app.$data.title = 111
```

使用defineProperty拦截监听的一些问题

- 属性新增属性

  只能拦截单个数据，对于对象新增属性无法拦截，因为只有初始化`new Vue`的时候会对data做defineProperty

- 数组方法

- 数组新增值：`[]`

- 数组length属性

​    以上操作不会触发监听拦截

```js
组件中必须这样使用, 不用函数的话会互相影响
data() {
  return {
    a: 1
  }
}
```





Vue的话属性新增是不可以的，其他的自己做了一些处理可以解决（比如数组做了一个重新包装）

```js
// 属性新增不行
data: {
  title: '123'
  user: {
    name: 'zs'
  }
}
app.age = 2

// 解决 - 这样操作之后就是响应式了
Vue.set('age', 2)
app.$set('age', 2)
Vue.set(app.user, 'gender', 'male')

set其实就是又做了一层defineProperty
```

# 生命周期

​	在创建vue实例（`new Vue`）的过程中会发生很多事情。

​	首先是初始化。帮我们初始化事件，生命周期相关的成员，包括h函数

​	首先是beforeCreate，实例初始化后被调用，在 `beforeCreate` 钩子函数调用的时候，是获取不到 `props` 或者 `data` 中的数据的。（此时data的响应式追踪，event/watcher都还没有被设置，也就是说不能访问到data，props，computed，watch，methods上的方法和数据）

​	然后会执行 `created` 钩子函数，在这一步的时候已经可以访问到之前不能访问到的数据，但是这时候组件还没被挂载，所以是看不到的。

​	接下来会先执行 `beforeMount` 钩子函数，开始创建 VDOM，最后执行 `mounted` 钩子，并将 VDOM 渲染为真实 DOM 并且渲染数据。组件中如果有子组件的话，会递归挂载子组件，只有当所有子组件全部挂载完毕，才会执行根组件的挂载钩子。

​	接下来是数据更新时会调用的钩子函数 `beforeUpdate` 和 `updated`，这两个钩子函数没什么好说的，就是分别在数据更新前和更新后会调用。

​	另外还有 `keep-alive` 独有的生命周期，分别为 `activated` 和 `deactivated` 。用 `keep-alive` 包裹的组件在切换时不会进行销毁，而是缓存到内存中并执行 `deactivated` 钩子函数，命中缓存渲染后会执行 `actived` 钩子函数。

​	最后就是销毁组件的钩子函数 `beforeDestroy` 和 `destroyed`。前者适合移除事件、定时器等等，否则可能会引起内存泄露的问题。然后进行一系列的销毁操作，如果有子组件的话，也会递归销毁子组件，所有子组件都销毁完毕后才会执行根组件的 `destroyed` 钩子函数。



⚠️ `beforeCreate`执行完后是初始化注入的操作，会把props，data，methods等等成员注入到vue的实例中。

⚠️ `created`执行完成。到此，vue创建完毕。下面做的事情是帮我们把**模版编译成render函数**。首先判断选项中是否设置了el选项，如果没有就调用$mount()方法，$mount实际就是帮我们把el转换成template，然后判断是否设置了template，如果没有的话将el外部的html作为template模版，否则将template编译到render函数中

​	接下来准备挂载dom。执行beforeMount

⚠️ 调用`vm.$destroy`执行销毁阶段



# 指令

## 基本指令

```js
// v-text
{{  }} 这个的问题，有时候内容比较多的话会先显示{{ title }}这种样子，就可以用v-text解决

// v-cloak
也可以解决上面的问题，不过需要配合css属性选择器
[v-cloak]: {
  display: none;
}
当渲染完后会自动将选择器里面的值设置为 display: block

// v-html
渲染html

// v-once
只渲染一次，后期的更新不再渲染

// v-pre
忽略这个组件以及它的子元素的编译，比如你就想输出{{}}这个东西

// v-show v-if

// v-for 
v-for也可以遍历object

// v-bind
v-bind:id="myId" 等价于 :id="myId"
:style="'width: 200px'"
:class="'box1 box2'"
:class="{'box1': isActive}"

// v-model
```

​	attribute是html标签上的属性，property是dom（对象）的属性



## 指令修饰符

```js
.lazy // 懒
.trim	// 去除首尾空格
.number // 转数字

v-model.lazy="val"
```



## 自定义指令

### 注册指令

**全局指令**

```js
Vue.directive('test', {
  
})
```

**局部指令**

​	只在当前组件中可以使用

```js
new Vue({
  directive: {
    name: {
      
    }
  }
})
```

​	v-name即可使用

### 指令生命周期

- bind：第一次绑定到元素的时候调用，只调用一次，可以做一些初始化，但是这时候页面可能还没有插入这个元素
- inserted：被绑定元素被插入父节点时使用
- update：所在组件更新时调用
- componentUpdated：所在组件更新完成时调用
- unbind：解绑时调用，只调用一次

```js
// 会传入el和binding
el就是所在的元素
binding是指令配置细节，参数，修饰符，值，名称等

// 此外还有vnode和oldVnode
```



### 实现获取焦点指令

```jsx
Vue.directive('focus', {
  inserted(el, binding) {
    binding.isFocus && el.focus()
  }
});

<input v-focus="isFocus" />
```

​	第四期在这里讲了一个实现拖拽指令



## 组件的v-model与.sync

```jsx
<div v-model="val"></div>

Vue.component('kkb', {
  model: {
    prop: 'checkedValue',// checkedValue和val进行双绑
    event: 'click'
  }
})

this.$emit('click', this.value)
```

还有.sync 在高级四期Vue（四）

# 事件

```jsx
<div @click="fn"></div>

fn(e) {
  第一个参数就是事件对象
}
// ---
<div @click="fn('1')"></div>
// 或者 <div @click="fn('1', $event)"></div>
fn(1, e) {
  
}
```

 也有修饰符

- .stop：阻止冒泡
- .prevent：默认行为
- .capture：是否捕获
- .self：只有本身触发才执行
- .once：只绑一次
- .passive
- .keyCode
- .enter
- .down
- .exact
- .native：指定监听原生事件，而不是组件自定义事件

```js
@click.stop
@keyup.13 // 13某个keyCode，当按下键是13的时候
```



# watch

​	计算属性不接受异步

```js
watch: {
  keyWord() {
    setTimeout => this.showUsers = this.users.filter(...)
  },
  a: {
    handler: function() {},
    deep: true // 深度监听
  }                                                  
}
// 也可以多层监听
watch: {
  a.b.c: function() {}
}
```



# 过滤器

filters 可以直接用 | 来使用

# 插槽

高级四期，Vue（四）

# ref和$refs

高级四期，Vue（四）





















