# 微信小程序

​	开发者工具右上角详情可以更改APPID，最下面有个不校验合法域名...，在没有配置域名的情况下点击这个也可以进行本地开发

app.json 可以设置pages，window，tabbar（底部的那个）



主包8M，分包2M

云开发后台2个g 日流量5万次

微信公众平台 - 登录 - 扫码 - 

1. 左栏有个开发 - 上栏有个开发设置能看到APPID
2. 左栏有个管理 - 成员管理 - 可以添加权限
3. 开发者工具 - 右上角 >>符号可以上传之类的 - 上传后可以在版本管理里看到 

app.json - 全局配置，pages下放在最上面的页面会默认加载这个页面

如果是局部的配置的话默认就是只能操作window下面的配置。

## 不同点

`view` - div

`text` - span

`rpx` - 相当于做适配, 类似rem，比如选了iphon6的尺寸，可以看到（375 * 667 | Dpr2），如果你想要个自适应宽度100%的话可以这样写 `width: 750rpx`（375 * Dpr）

`src`- / 这个就是相当于我们文件的根目录



`bindtap` - tap是轻敲的意思，相当于onclick，对应的方法直接写在js的Pages里面就行

`catchtap` - 也是点击，不同点是阻止冒泡，两者的都不能带括号`bindtap = fn`，如果要传参数则如下

```html
<button data-name="ming" bindtap="fn">{{msg}}</button>

<!-- js文件 -->
fn(e) {
  console.log(e.currentTarget.dataset.name) // "ming"
  this.setData({
	msg: "123"
  })
}

```



`{{ }}` 类似vue

`wx:if = "{{true}}" ` - 类似 v-if，此外还有`wx:else`   `wx:elseif`

`wx:for = "{{array}}"` `wx:for-item = "itemName"`  `wx:for-index = idx`  `wx:key`

`setData` - 类似react - setState

## 引用模板

```html
// templates/a.wxml
<template name="a">
  <view>
  	1234  
  </view>
</template>
<view>
  2222
</view>

// 我要引入上面这个模板
<import src="/pages/templates/a.wxml" />
<template is="a"></template>
// 引如模板之外的那个2222
<include src="/pages/template/a.wxml"> <!-- 2222 -->

```

## 网络请求

小程序只能请求https的域名，域名一定要在后端配置的地方配置一下（开发 - 服务器域名 - 开始配置）

没有跨域问题

```js
wx.request({
  url: '',
  success: res => {
    console.log(res.data)
  }
})
```

## 生命周期

onLoad - 监听页面加载 - 可以去发请求

## 路由跳转

`wx.navigateTo` - 保留当前页面，跳转到应用内的某个页面。`wx.navigateBack`可以返回原页面

## 组件

​	小程序也有组件，一般在components文件夹下，通过右键，新建组件就可以，不同的是组件的js文件是`Component（{}）`组件的方法一般写在methods里，不像页面随便写。json文件里要声明`component: true`。

```js
// 引用组件
// json
{
  "usingComponent": {
    "my-component": "/components/my-component/my-component"
  }
}
```



```js
properties: { // 类似于props, 不过小程序可以修改这个值
  myvalue: {
    type: "string",
    value: "",
    observer(newValue, oldValue) { // 可以观察这个值
      console.log(newValue, oldValue)
    }
  }
},
    
methods: {
  goDad () {
    this.triggerEvent("myevent", "参数") // 类似$emit
  }
}

// 在父级中可以通过bind:myevent来接收, 类似@myevent
<my-component
  bind:myevent="fn"
>
<my-component>
```

## 插槽

​	类似vue

## websocket

```js
wx.connectSocket({
  url: 'ws://',
})
wx.onSocketOpen(() => {
  console.log("链接成功")
})
wx.sendSocketMessage({
  data: "some value..."
})
wx.onSocketMessage(function(res) {
  console.log(res)
})
```



## 3秒点击游戏

5期 => 小程序2集 => 2集 `1：14：58` - 3集 `42：41` => `!important` => 授权

## 云开发

​	文档有单独罗列的云开发文档，微信开发者工具上面有个云开发按钮

​	相当于一个人解决前后端，云开发后面有数据库之类的，也可以保存文件，然后在小程序里获取文件之类的。

​	在`project.config.json`里配置使用云开发也可以

​	有时候有些问题可能是权限问题，点击云开发 - 数据库 - 有个权限管理

```js
// 在pages/index.js里
const db = wx.cloud.database() // 得先实例化一下
// 增
addData() {
  db.collection("test").add({ // 集合名称test, 也即是一个表
    data: {
      name: "张三",
      age: 20
    }
  }).then(res => {
    console.log(res)
  })
}
// 删
delData() {
  db.collection("test").doc('d3d8h3bnhbv')// doc是id
    .remove().then(res => {
      console.log(res.data)
  	})
}
// 改
const _ = db.command
updateData() {
  db.collection("test").doc('d3d8h3bnhbv')
    .update({
      data: {
        age: _.gt(21) // 大于21的改成李四
        name: "李四"
      }
  	}).then(res => {
      console.log(res)
  	})
}
// 查
getData() {
  db.collection("test").where({
      age: 18
  	})
    .get().then(res => {
      console.log(res)  
  	})
}
```

如果是较复杂的操作要借助云函数进行, 如删除多条记录，更新等

在cloudFunctions目录下右键 - 新建Nodejs云函数 - updateFn。建好后文件夹里面有index.js和package.json

index.js里面其实就是写node（里面会默认先帮你写好一些东西），写完以后点击右键上传并部署

```js
// 如何调云函数呢
wx.cloud.callFunction({ // 可以理解为ajax
  name: "updateFn",// 像接口名称一样
  data: {
    a: 10,
    b: 20
  }
}).then(res => {
  console.log(res)
})
```

1：45：32

































-------------------------------------------------------------------

openid是啥意思





