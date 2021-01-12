# 基础

🍇 router/index.js

```js
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)

const routes = [
  {
    path: 'login',
    name: 'Login',
    component: Login
  },
  {
    path: '/',
    component: 'Layout',
    children: [
      {
        path: '',
        name: 'Index',
        component: Index
      },
      {
        path: 'detail/:id',
        name: 'Detail',
        props: true, // 组件通过props即可获取URL参数
        component: () => import('../views/Detail.vue')
      }    
    ]
  },
  {
    path: '*',
    component: 404
  }

]

const router = new VueRouter({
  routes
})

export default router
```

- **Vue.use**注册vue组件，它会调用传入对象的install方法。如果是函数，会直接调用。

- **懒加载**，当用户访问时才会被加载

- **嵌套路由**，会先加载Layout组件，然后加载内部的组件，比如我们需要相同的header和footer时可以这样。它们的path会自动拼接到一起

🍇 main.js

```js
import router from './router'

new Vue({
  router,
  render: h => h(App)
}).$mount("#app")
```

​	把router注册到vue实例的作用是什么，它会将`$router`和`$route`添加到vue实例对象上。

​	有时候在某些插件上不方便拿到`$route`，这时候就可以通过`$router`里面的`currentRoute`获取



🍇 编程式导航

```js
this.$router.push('/')
this.$router.push({
  name: 'Index',
  params: {
    id: 1
  }
})

this.$router.replace('/') // 不会记录本次历史

this.$router.go(-2)
```



# hash和history

- hash是基于锚点，以及onhashchange事件

  url中#后面的内容作为路径地址

  监听hashchange事件

  根据当前路由地址找到对应组件进行渲染

- history是基于HTML5中的History API，`history.pushState(), replaceState()`，`history.push`方法会向服务器发送请求，而前者不会。（但是注意，刷新的时候会向服务器发送请求，所以还是要配置）

  pushState方法仅仅只会改变地址栏，不会发送请求。监听popstate事件可以监测到浏览器历史操作的变化，可以记录改变后的地址，pushState和replaceState的时候不会触发该事件，当点击浏览器的前进或后退的时候，或者调用history'的back和forward的时候，该事件才会被触发

🍇

​    node直接用express，`express().use(history())`即可配置成功不发送请求

​	ngix的话打开nginx.config配置文件，找到server

```nginx
server {
  location / {
    root html；
    index index.html iindex.htm；
    try_files $uri $uri/ /index.html；
  }
}
```

​	主要就是加一个try_files



# 实现

🍇 vuerouter/index.js

```js
let _Vue = null

export default class VueRouter {
  // install方法
  static install(Vue) {
    if(VueRouter.install.installed) {
      return
    }
    VueRouter.install.installed = true
    _Vue = Vue
    _Vue.mixin({
      beforeCreate() {
        if(this.$options.router) {
          _Vue.prototype.$router = this.$options.router    
          this.$options.router.init()
        }
      }
    })
  }
  // 构造函数
  constructor(options) {
    this.options = options
    this.routeMap = {}
    this.data = _Vue.observable({
      current: '/'
    })
  }
  // createRouteMap
  createRouteMap() {
    this.options.routes.forEach(route => {
      this.routeMap[route.path] = route.component
    })
  }
  // initComponents
  initComponents(Vue) {
  	Vue.component('router-link', {
      props: {
        to: String
      },
      template: '<a :href="to"><slot></slot></a>'
    })
    const self = this
    Vue.component('router-view', {
      render(h) {
        const component = self.routeMap[self.data.current]
        return h(component)
      }
    })
	}
  // initEvent
  initEvent() {
    window.addEventListener('popstate', () => {
      this.data.current = window.location.pathname
    })
  }
  
  // init
  init() {
    this.createRouteMap()
    this.initComponents(_Vue)
    this.initEvent()
  }
}
```

 **install方法**

1. 判断当前插件是否被安装

   静态方法也是方法，方法也是对象，对象就可以有属性，所以可以给install添加installed属性

2. 把Vue构造函数记录到全局变量

3. 把创建Vue实例时传入的router对象注入到Vue实例上

   想让所有Vue实例共享，可以用prototype挂到构造函数上。我们可以获取Vue实例时就可以用this，所以要在beforeCreate里写。但是我们只需要执行一次，只需要在Vue实例创建时执行就好了，如果是组件就不让它执行，而只有实例中有`this.$options.router`

**构造函数**

1. 记录options
2. routeMap里健放路由地址，值放对应的组件
3. data需要设置为响应式，存放current当前路由

**createRouteMap**

1. 遍历所有路由规则，解析成健值对形式

**initComponents**

1. 实现router-link和router-view

**initEvent**

1. 实现完以上以后跳转基本没问题，但是点浏览器的后退，前进就不行，因为没有实现这些方法，需要popState，它在历史发生变化的时候会被触发，pushState，replaceState不会触发



# Vue的构建版本

运行时版：不支持template模板，需要打包的时候提前编译

完整版：包含运行时和编译器，体积比运行时版大10K左右，程序运行的时候把模版转换成render函数

vue-cli默认使用的是运行时版本，因为运行效率更高

如何把运行时转为完整版，如下

🍇 vue.config,js

```js
module.exports {
  runtimeCompiler: true
}
```



​	我们写单文件组件的时候不一直也在写模版吗，这是因为在打包的过程中把单文件的template编译成了render函数，这叫做预编译

​	此外，如果是运行时不支持template，我们也可以直接使用render函数

```js
Vue.component('router-link', {
  props: {
    to: String
  },
  // template: '<a :href="to"><slot></slot></a>'
  render(h) {
    return h('a', {
      attrs: {
        href: this.to
      },
      on: {
        click: this.clickHandler
      }
    }, [this.$slots.default])
	},
  methods: {
    clickHandler(e) {
      history.pushState({}, '', this.to)
      this.$router.data.current = this.to
      e.preventDefault()
    }
  }           
})
```

1. 接收h，h函数作用是帮我们创建虚拟dom
2. 设置属性告诉它是dom属性，所以用attrs
3. `this.$slots.default` 通过代码获取默认插槽内的东西











