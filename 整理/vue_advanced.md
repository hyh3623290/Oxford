# 响应式原理

- 给Vue实例新增一个成员是否是响应式的 - 否
- 给属性重新赋值成对象，是否是响应式的 - 是


​    实例化时才会让data成为响应式，所以新增一个成员不是响应式

```js
Vue.set(vm.someObject, 'b', 2)

// this.$set(this.someObject, 'b', 2)
```

​	猜测只不过是调用了defineReactive，把这个属性转换成了getter和setter



## Object.defineProperty

```js
let data = {
  msg: 'hello'
}

let vm = {}

Object.defineProperty(vm, 'msg', {
  enumerable: true,
  configurable: true,// 是否可delete，通过defineProperty重新定义
  get() {
    return data.msg
  },
  set(newValue) {
    if(newValue === data.msg) {
      return
    }
    data.msg = newValue
    document.querySelector('#app').textContent = data.msg
  }
})
```

​	如果监听多个属性就需要遍历，`Object.keys(data).forEach(key => ...)`



## Proxy

​	直接监听对象，而非属性，不需要遍历了，简洁多了。性能更高，由浏览器进行性能优化。

```js
let vm = new Proxy(data, {
  get(target, key) {
    return target[key]
  },
  set(target, key, newValue) {
    if(target[key] === newValue) {
      return
    }
    target[key] = newValue
    document.querySelector('#app').textContent = data.msg
  }
})
```



## 发布订阅模式

- 订阅者
- 发布者
- 信号中心

​    我们规定，有一个信号中心，某个任务执行完成，就向信号中心发布一个信号，其他任务可以向信号中心订阅这个信号，从而知道什么时候自己可以开始执行。

🍇 Vue的自定义事件基于发布订阅模式

```js
let vm = new Vue()

vm.$on('dataChange', () => {
  console.log('datachange')
})

vm.$emit('dataChange')
```



🍇 兄弟组件通信过程

```js
// eventBus.js 事件中心
let eventHub = new Vue()

// ComponentA.vue 发布者
addTodo: function() {
  eventHub.$emit('add-todo', { text: 'xxx' })
}

// ComponentB.vue 订阅者
created: function() {
  eventHub.$on('add-todo', this.addTodo)
}
```



🍇 模拟Vue自定义事件的实现

```js
class EventEmitter {
  constructor() {
    // { 'click': [fn1, fn2] }
    this.subs = Object.create(null)
  }
  
  $on(eventType, handler) {
    this.subs[eventType] = this.subs[eventType] || [] // 有可能本身就注册过
    this.subs[eventType].push(handler)
  }
  
  $emit(eventType) {
    if(this.subs[eventType]) {
      this.subs[eventType].forEach(handler => {
        handler()
      })
    }
  }
}
```

​	Object.create需要一个参数，用来设置对象的原型





## 观察者模式

​	响应式机制中使用了观察者模式，区别是没有事件中心

- 观察者 -- Watcher
  1. `update()`: 当事件发生时，具体要做的事情
- 目标 -- Dep
  1. `subs` 数组：存储所有的观察者
  2. `addSub`：添加观察者
  3. `notify`：当事件发生，调用所有观察者的update方法

```js
class Dep {
  constructor() {
    this.subs = []
  }
  
  addSub(sub) {
    if(sub && sub.update) {
      this.subs.push(sub)
    }
  }
  
  notify() {
    this.subs.forEach(sub => {
      sub.update()
    })
  }
}

class Watcher {
  update() {
    console.log('update')
  }
}
```

```js
let dep = new Dep()
let watcher = new Watcher()

dep.addSub(watcher)
dep.notify()
```

- 观察者模式是由具体目标调度，比如事件触发，Dep就会调用观察者的方法，所以观察者模式的订阅者和发布者之间是存在依赖的
- 发布订阅模式由统一的调度中心调用，因此发布者和订阅者无需知道对方的存在





## 响应式原理模拟

### Vue

- 负责接收初始化的参数
- 负责把data中的属性注入到vue实例，转换成getter/setter
- 负责调用observer监听对象中所有属性的变化
- 负责调用compiler解析指令/差值表达式

```js
class Vue {
  constructor(options) {
    this.$options = options || {}
    this.$data = options.data || {}
    this.$el = typeof options.el === 'string' ? document.querySelector(options.el) : options.el
 		this._proxyData(this.$data)// 注入到Vue实例
    new Observer(this.$data)// 把data的所有属性转换成getter，setter
    new Compiler(this)
  }
  
  _proxyData(data) {
    Object.keys(data).forEach(key => {
      Object.defineProperty(this, key, {
          enumerable: true,
  				configurable: true,
        	get() {
            return data[key]
          },
        	set(newValue) {
            if(newValue === data[key]) {
              return
            }
            data[key] = newValue
          }
      })
    })
  }
}

```



### Observer

- 负责把data选项中的属性转换成响应式数据
- data中的某个属性也是对象，把该属性转换成响应式数据
- 数据变化发送通知

```js
class Observer {
  constructor(data) {
    this.walk(data)
  }
  
  walk(data) {
    if(!data || typeof data !== 'object') {
      return
    }
    Object.keys(data).forEach(key => {
      this.defineReactive(data, key, data[key])
    })
  }
  
  defineReactive(obj, key, value) {
    let that = this // 指向Observer实例
    let dep = new Dep()
    this.walk(value)
    Object.defineProperty(obj, key, {
      enumerable: true,
  		configurable: true,
      get() {
        Dep.target && dep.addSub(Dep.target)
        return value // 这里如果返回obj[key]会死递归
      },
      set(newValue) {
        if(newValue === value) {
          return
        }
        value = newValue
        that.walk(newValue)
        dep.notify()
      }
    })
  }
}

```



### Compiler

- 负责编译模版，解析指令/差值表达式
- 负责页面的首次渲染
- 当数据变化后重新渲染页面

```js
class Compiler {
  constructor(vm) {
    this.el = vm.$el
    this.vm = vm
    this.compile(this.el)
  }
  
  compile(el) {
    let childNodes = el.childNodes
    Array.from(childNodes).forEach(node => {
      if(this.isTextNode(node)) {
        this.compileText(node)
      } else if(this.isElementNode(node)) {
        this.compileElement(node)
      }
      if(node.childNodes && node.childNodes.length) {
        this.compile(node)
      }
    })
  }
  
  compileElement(node) {
    Array.from(node.attributes).forEach(attr => {
      let attrName = attr.name // name:v-text,value:'aaa'
      if(this.isDirective(attrName)) {
        // v-text --> text
        attrName = attrName.substr(2)
        let key = attr.value
        this.update(node, key, attrName)
      }
    })
  }
  update(node, key, attrName) {
    let updateFn = this[attrName + 'Updater']
    updateFn && updateFn.call(this, node, this.vm[key], key)
  }
  textUpdater(node, value, key) { // 处理v-text
    node.textContent = value
    new Watcher(this.vm, key, (newValue) => {
      node.textContent = newValue
    })
  }
  modelUpdater(node, value, key) {
    node.value = value
    new Watcher(this.vm, key, (newValue) => {
      node.value = newValue
    })  
  }
  
  compileText(node) { // 编译文本，处理差值表达式
    let reg = /\{\{(.+?)\}\}/
    let value = node.textContent
    if(reg.test(value)) {
      let key = RegExp.$1.trim()
      node.textContent = value.replace(reg, this.vm[key])
    }
    new Watcher(this.vm, key, (newValue) => {
      node.textContent = newValue
    })
  }
  
  isDirective(attrName) { // 是否是指令
    return attrName.startWith('v-')
  }
  
  isTextNode(node) { // 是否文本
    return node.nodeType === 3
  }
  
  isElementNode(node) { // 是否元素节点
    return node.nodeType === 1
  }
}
```



### 响应式

#### Dep

- 收集依赖，添加观察者

  每一个响应式属性将来都会创建一个对应的Dep对象，它负责收集所有依赖于该属性的地方，所有依赖该属性的地方都会创建一个watcher对象（getter - 收集依赖，setter - 通知依赖）

- 通知所有观察者

```js
class Dep {
  constructor() {
    this.subs = [] // 存储所有观察者
  }
  addSub(sub) { // 添加观察者
    if(sub && sub.update) {
      this.subs.push(sub)
    }
  }
  notify() { // 发送通知
    this.subs.forEach(sub => {
      sub.update()
    })
  }
}
```



#### Watcher

- 当数据变化时触发依赖，dep通知所有的watcher实例更新视图
- 自身实例化的时候往dep对象中添加自己

```js
class Watcher {
  constructor(vm, key, cb) {
    this.vm = vm
    this.key = key // data中的属性名称
    this.cb = cb
    Dep.target = this // 记录到Dep的静态属性
    this.oldValue = vm[key] // 获取属性可以触发get方法，getter将收集依赖
    Dep.target = null // 防止重复添加
  }
  
  update() {
    let newValue = this.vm[this.key]
    if(newValue === this.oldValue) {
      return
    }
    this.cb(newValue)
  }
}
```



# --

51.54

11.05























