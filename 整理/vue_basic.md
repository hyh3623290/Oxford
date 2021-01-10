# 生命周期

​	在创建vue实例（`new Vue`）的过程中会发生很多事情。

​	首先是初始化。帮我们初始化事件，生命周期相关的成员，包括h函数

​	在 `beforeCreate` 钩子函数调用的时候，是获取不到 `props` 或者 `data` 中的数据的，因为这些数据的初始化都在 `initState` 中。

​	然后会执行 `created` 钩子函数，在这一步的时候已经可以访问到之前不能访问到的数据，但是这时候组件还没被挂载，所以是看不到的。

​	接下来会先执行 `beforeMount` 钩子函数，开始创建 VDOM，最后执行 `mounted` 钩子，并将 VDOM 渲染为真实 DOM 并且渲染数据。组件中如果有子组件的话，会递归挂载子组件，只有当所有子组件全部挂载完毕，才会执行根组件的挂载钩子。

​	接下来是数据更新时会调用的钩子函数 `beforeUpdate` 和 `updated`，这两个钩子函数没什么好说的，就是分别在数据更新前和更新后会调用。

​	另外还有 `keep-alive` 独有的生命周期，分别为 `activated` 和 `deactivated` 。用 `keep-alive` 包裹的组件在切换时不会进行销毁，而是缓存到内存中并执行 `deactivated` 钩子函数，命中缓存渲染后会执行 `actived` 钩子函数。

​	最后就是销毁组件的钩子函数 `beforeDestroy` 和 `destroyed`。前者适合移除事件、定时器等等，否则可能会引起内存泄露的问题。然后进行一系列的销毁操作，如果有子组件的话，也会递归销毁子组件，所有子组件都销毁完毕后才会执行根组件的 `destroyed` 钩子函数。



⚠️ `beforeCreate`执行完后是初始化注入的操作，会把props，data，methods等等成员注入到vue的实例中。

⚠️ `created`执行完成。到此，vue创建完毕。下面做的事情是帮我们把**模版编译成render函数**。首先判断选项中是否设置了el选项，如果没有就调用$mount()方法，$mount实际就是帮我们把el转换成template，然后判断是否设置了template，如果没有的话将el外部的html作为template模版，否则将template编译到render函数中

​	接下来准备挂载dom。执行beforeMount

⚠️ 调用`vm.$destroy`执行销毁阶段