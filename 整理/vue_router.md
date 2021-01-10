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
- history是基于HTML5中的History API，`history.pushState(), replaceState()`，`history.push`方法会向服务器发送请求，而前者不会。（但是注意，刷新的时候会向服务器发送请求，所以还是要配置）

​    node直接用express，`express().use(history())`即可配置成功

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



41.06



















