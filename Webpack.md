# 模块化

## CommonJs

```js
require('./moduleA')
var m = require('./moduleB')
console.log(m)

// moduleA
var m = 1
module.exports = m
```

​	commonJs是同步的，然后有了AMD，是异步的模块化规范

## AMD

```js
require(['moduleA', 'moduleB'], function(moduleA, moduleB) {
  console.log(moduleB) // 通过回调的方式
})

// moduleA
define(function(require) {
  var m = require('moduleB')
	setTimeout(() => { console.log(m) })
})
```

​	AMD在node环境下还需要一个加载器脚本，常见的就是require.js

⚠️【爪哇 - 49】

# entry

​	可以接收字符串，数组，对象

```js
entry: "./src/index.js"

entry: ["./src/index.js", "./src/other.js"]

// 多入口
entry: {
  main: "./src/index.js",
  other: "./src/index.js",
}
```

# output

​	多入口对应输出文件 `[name].js`，会输出两个文件( bundle )。一个输入文件对应一个chunk，对应一个bundle

​	占位符 - hash，chunkhash，name，id

```js
output: {
  path: path.resolve(__dirname, "./dist")
  filename: "[name]-[hash:6].js"
}
```

hash的作用就是缓存

# loader

​	模块解析，webpack默认只认识json和js ，其他的就需要loader了

## **css-loader**

​	是把css的东西放到js中去，css in js方式，所以还需要style-loader，把js里的内容提取出来，在html中创建style标签并放入内容

```js
module: {
  rules: [
    {
      test: '/\.css$/',
      use: ['style-loader','css-loader']
    }
  ]
}
```

## **css modules**

```js
module: {
  rules: [
    {
      test: '/\.css$/',
      use: [
       	'style-loader',
        {
          loader: 'css-loader',
          options: {
            modules: true
          }
        }
      ]
    }
  ]
}
```

##  **postcss-loader**

​	css后处理器，可以帮忙加前缀，帮我们处理css3之类的东西

​	`npm install postcss-loader autoprefixer`

```js
module: {
  rules: [
    {
      test: '/\.css$/',
      use: [
       	'style-loader',
        {
          loader: 'css-loader',
          options: {
            modules: true
          }
        },
        'postcss-loader',
        'less-loader'
      ]
    }
  ]
}
```

​	然后创建postcss.config.js文件

```js
const autoprefixer = require("autoprefixer")
module.exports = {
  plugins: {
   [
    autoprefixer({
      overrideBrowserslist: ["last 2 versions", ">1%"] 
    })
    // autoprefixer("IE 10")
   ]
  }
}
```

## **file-loader**

​	处理图片，第三方字体，txt呀，word之类

```js
module: {
  rules: [
    {
      test: '/\.(png|gif|jpeg)$/',
      use: [
        {
          loader: 'file-loader',
          options: {
            name: "[name][hash:6].[ext]",
           	outputPath: "images/"
          }
        }
      ]
    }
  ]
}
```

⚠️ cdn 其实就是把打包后的文件上传到了cdn服务器 - 第一次访问cdn是快于普通的 - 有缓存以后 就一样了

​	处理字体

```less
@font-face {
  font-family: 'webfont';
  font-display: 'swap';
  src: url("webfont.woff2") format("woff2");
}

body {
  font-family: "webfont" !important;
}

// webpack.config.js
{
  test: /\.(eot|woff2)$/,
  use: "file-loader"
}
```

## url-loader

​	包含file-loader的全部功能，此外可以通过limit把图片用base64表示，小体积建议base64

## babel-loader

​	用法跟其他loader，此外又个exclude属性，可以排除node_modules下的内容。其实babel-loader只是babel和webpack的一个通信桥梁，它本身是不做语法转换的，所以需要额外安装。@babel/preset-env做语法转换，@babel/core做语法解析

```js
npm i @babel/core @babel/preset-env -D

use: {
  loader: "babel-loader",
  options: {
    // 语法转换插件 preset-env
    presets: [
      [
        "@babel/preset-env",
        {
          targets: {
            edge: "17",
            chrome: "67"
            safari: "11.1",
            firefox: "60"
          },
          corejs: 2,
          useBuiltIns: "usage" // 按需引入
        }        
      ],
  		"@babel/preset-react"
    ]
  }
}
```

​	你发现promise没转换，不认识promise的浏览器怎么办 - polyfill  是ES6+的ECMA规范库

```js
npm i @babel/polyfill -S // 需要生产依赖！
```

useBuiltIns有三个参数

- entry：需要在webpack的入口文件`import @babel/polyfill`引入一次，babel会自动按需引入
- usage：不需要引入，会自动全局检测，需要安装@babel/polyfill，（试验阶段）
- false：全部引入

​    ⚠️polyfill的缺点是会污染全局对象，因为把一些api挂到了window上。但是这个对开发也不算是缺点，真正的是对一些开源库，UI库，组件库，工具库有影响。所以推荐@babel/plugin-transform-runtime，这个是闭包方式，不会污染全局

⚠️**对react，jsx做处理**

```js
npm install --save-dev @babel/preset-react // 官网
```

⚠️**.babelrc**

​	只需要把options的value值拿出来放进去就好了，然后options就可以 不要了，只留一个babel-loader

# Plugin

​	插件，对webpack的补充

## clean-webpack-plugin

​	实际就是删了重建打包文件，做清理工作，把冗余文件清除掉

```js
// 1.引入 - require
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
// 2.使用 - new
plugins: [new CleanWebpackPlugin()]
```



## html-webpack-plugin

​	首先还是需要创建一个模板在src，然后把它输出到dist。最后会自动帮我们引入bundle文件

```js
plugins: [new HtmlWebpackPlugin({ // 传参
  template: "./src/index.html",
  filename: "index.html",
  // minify压缩 favicon设置icon
  // title
})]
```

​	title要想设置成功，因为这个插件默认支持ejs模板语法

```php+HTML
<title><%= htmlWebpackPlugin.options.title %></title>
```

## mini-css-extract-plugin

```js
plugins: [
  new MiniCssExtractPlugin({
    filename: "css/[name][chunkhash:8].css" // css目录下
  })
]

module: {
  rules: [
    {
      test: '/\.css$/',
      use: [
        MiniCssExtractPlugin.loader,
        'css-loader'
      ]
    }
  ]
}
```

​	把css提取成一个独立的文件，并且注意到是就不用style-loader了，而是这个插件自带的loader，用完会发现它还会帮我们自动引入css文件在html中

# sourceMap

```js
devtool: "cheap-module-eval-source-map" 
// none
// inline 没有map文件
// cheap 不包含列信息
// module 第三方模块，包含loader的sourceMap，如jsx，babel等
```

​	设置sourceMap后会多出来一个.map文件，表示与bundle代码的一个映射



# webpackDevServer

```js
"script": {
  "dev": "webpack-dev-server"
}
// npm run dev
```

​	这样即可实现热更新，发生改变，通知浏览器去更新。这样我改代码一保存，浏览器就刷新了。同时它是可以设置的，如下：

```js
devServer: {
  contentBase: path.resolve(__dirname, "./dist"), // 设置根目录
  port: 8080,
  open: true // 自动打开浏览器窗口
}
```

## mock

​	先本地创建server.js，模拟服务器，npm i express -D

```js
// server.js
const express = require("express")
const app = express()

app.get("/api/info", (req, res) => {
  res.json({
    hello: "express"
  })
})

app.listen("9092")
// node server.js 启动服务
// localhost:9092/api/info
```

```js
axios.get("/api/info").then()
```

​	现在这么请求会跨域，端口号都不一样。那现在devServer怎么帮我们解决这个问题，那就是代理，当我们本地请求这个服务的话我们代理到哪里去，代理到目标服务器上去，目标服务器就是我们的这个9092，打破同源策略

```js
devServer: {
  proxy: {
    "/api": {
      target: "localhost:9092"
    }
  },
  before(app, server) {
    app.get("/api/info", (req, res) => {
      res.json({
        hello: "express"
      })
    })  
  },
  after() {}
}
```

​	当我们带上api这个字段的时候，就会来做一个转发，代理到target。

​	第二种mock数据的方式是通过before钩子，就不用开启那个node服务了

## HMR 热模块替换

​	代码生效但是操作保留，不要刷新浏览器

```js
const webpack = require("webpack")

devServer: {
  hot: true,
  hotOnly: true // 即便hmr没有生效，浏览器也不要自动刷新
}

plugins: [
  new webpack.HotModuleReplacementPlugin()
]
```

​	但是这个MiniCssExtractPlugin.loader对hmr支持非常不友好，需要把他换成style-loader，所以开发环境下不要用这个抽离css的MiniCssExtractPlugin。但这个只搞定css，对js还不行，需要如下这么做

```js
import number from './number.js'
number() // 需要监测的js

if(module.hot) {
  module.hot.accept("./number.js", function() {
    document.body.removeChild("那个dom")
    number()
  })
}
```

​	以上是原理，实际上不需要这样，直接安装react或者vue对应的hmr插件loader，如vue-loader，它自身就对.vue文件支持hmr，react的话对jsx 

# 性能优化

 1. 构建时间越来越长

 2. 优化输出质量

    生产环境代码体积尽可能的小，打开速度会更快

## 缩小文件范围

​	loader是消耗性能的大户，缩小loader的查找范围，`exclude`，`include`

```js
{
  test: '/\.css$/',
  include: path.resolve(__dirname, "./src") // 推荐
  use: ['style-loader','css-loader']
}

// excludes: path.resolve(__dirname, "./nodule_modules")
```



## Resolve

### modules

```js
resolve: {
  modules: [path.resolve(__dirname, "./nodule_modules")]
}
plugins: [
  // ...
]
```

​	这样就只会在当前的nodule_modules找第三方模块（依赖），不然的话它找不到还会去全局，耗费性能

### alias

​	减少查找过程，起别名

```js
resolve: {
  alias: {
    "@": path.resolve(__dirname, "./src/images")
    react: "./nodule_modules/react/umd/react.production.min.js"
    "react-dom": "./nodule_modules/react-dom/umd/react.production.min.js"
  }
}
// 这样import react的时候就被定位到那个路径了，不用消耗找的时间
```

⚠️1. 使用绝对路径也比较好，因为相对路径也会被转为绝对路径。2. react下面有个cjs和umd，cjs是基于commonjs的代码，umd是已经打包好的代码，可以直接拿过来用的代码。

### extensions

​	后缀匹配，建议不使用，还是写的时候都加上，比如index.jsx

 ```js
extensions: ['.js', '.json', '.jsx']
 ```

### externals

​	优化cdn的静态资源，比如我们用了cdn的链接，然后比如在代码里`import jquery`，这时候是不需要把它打包到bundle文件里面的。就可以在本地放心大胆的使用cdn方法。

```js
externals: {
  jQuery: "jQuery"
}
```

​	百度地图，echarts

​	其实不引入也可以，直接使用cdn暴露 的全局方法也可以，比如window.jquery

## publicPath

```js
output: {
  path: path.resolve(__dirname, "./dist")
  filename: "[name]-[hash:6].js"
  publicPath: "https://cdn.kaikeba.com/assets"
}

<script src="https://cdn.kaikeba.com/assets/main.js"><>
// 自动加前缀，然后记得把main.js上传到cdn服务器上
```



## 压缩css

optimise-css-assets-webpack-plugin

```js
plugins: [
  new OptimiseCssAssetsWebpackPlugin({
    cssProcessor: require("cssnano"),
    cssProcessorOptions: {
      discardComments: { removeAll: true }
		}
  })
]
// cssnano不用安装， postcss的一个依赖，安装的时候就撞上了
```

## 压缩HTML

html-webpack-plugin

```js
plugins: [new HtmlWebpackPlugin({ // 传参
  template: "./src/index.html",
  filename: "index.html",
  minify: {
    removeComments: true, // 移除注释
    collapeWhitespace: true, // 移除空白符与换行符
    minifyCss: true // 压缩内联css
  }
})]
```



## 区分开发和生产

`npm i webpack-merge -D `

1.33.01 kkb 5-4-2

5-5缺失了，差treeshaking之类的。





# ---

⚠️ mode会把process.env设置成对应的，webpack相关的好像都是开发依赖，比如各种loader



【参考文献】

1. kkb 16期