

组件通信

# $attrs/$listeners

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

# 插槽

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

# 通用表单组件设计

​	收集数据、校验数据并提交。

**需求分析**

​	实现KForm ：指定数据、校验规则 

​	KformItem ：执行校验 显示错误信息 

​	KInput ：维护数据

**设计**

​	把数据，校验规则之类放到最外层，方便进行一个全局的校验，

​	最外层form应依次执行formItem的校验方法，同时把自身整个传下去。这样item可以拿到整个form实例，进一步获取属于自己的数据并校验，最里面input负责派发事件。

## index.vue

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

## Form.vue

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

## FormItem

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

## input

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

# 弹窗组件设计

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

# VueRouter

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

1.13.10

# 