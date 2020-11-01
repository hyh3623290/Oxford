# webpack

​	执行`webpack ./src/a.js`命令，这样就会打包a.js文件，并且会将它的依赖也一起打包进去，嫌麻烦的话可以在package.json里写命令配置，直接执行npm run xxx。

​	还可以写个webpack.config.js配置文件，不用每次都在package.json里改，因为可以要加参数啊什么的，如果配好了入口之类的可以直接运行webpack。一般像vue这种配置单页面一般只需要一个入口。

​	如果是多入口

```js
entry: {
  'index': './src/index.js',
  'list': './src/list.js'
}
// 打包后即可生成index.js和list.js, 如果多出口则如下
output: {
  path: path.resolve(__dirname, "dist"),
  filename: "[name].js" // #1
}
```

1. `[name]`表示占位符，当然也可以加一些其他东西，如`[name]-kkb`，`[name]-[hash]`

   这个hash主要是解决缓存的问题，每次编译打包可以生成一个新的名称




## Loader

​	加载一些非js的文件，默认情况下浏览器只能处理js，webpack本质上也只能处理js，只是说它可以增加一些功能，有点类似插件的功能，让我们实现对一些不同模块的加载，不同类型模块的解析就是依赖不同的loader来实现的。

```js
module: {
  rules: [ // #1
    {
      test: /\.txt$/ // #2
      use: 'raw-loader'
    }
  ]
}
```

1. 每一个规则就是一个对象

2. test就是用来检测的，检测你加载的模块是什么类型，一般情况下就是一个正则检测是什么后缀的，如果是该后缀的就用你刚下载的什么什么loader（`use`），处理完之后进行打包

   

   有时候会发现一个loader不够用，就要两个loader，比如

```js
module: {
  rules: [
    {
      test: /\.md$/,
      use: [ // #1
        { loader: "html-loader" },
        { 
          loader: "mackdown-loader",
          options: {
            // ...
          }
        }
      ]
    }
  ]
}
```

1. 这样写的话比较麻烦，可以写成数组的形式

```js
use: ["html-loader", "mackdown-loader"]
// 先执行mackdown，再执行html
```

​		有些loader有一些自己的配置

```js
module: {
  rules: [
    {
      test: /\.(png|jpe?g|gif)$/,
      use: {
        loader: "file-loader",
        options: {
          name: "[name]_[hash].[ext]",
          outputPath: "./images", // #1
          publicPath: "./images" // #2
        }
      }
    }
  ]
}
```

1. 打包后的存放位置
2. 打包后文件的url，就是可以通过地址栏访问的路径



- url-loader：用法同file-loader，不同的是会将小图片转成base64
- css-loader：将css文件解析出来

```js
let style = document.createElement('style')
style.innerHTML = css[0][1] // #1
doocument.head.appendChild(style)
```

1. 解析后css里的内容会被放在数组第二项

- style-loader：将css生成的内容，用style标签挂载到页面的head上去，其实就是上面这段代码的作用

- sass-loader：把sass语法转成css，依赖node-sass模块
- vue-loader：可以解析vue单文件组件，你都不用装vue脚手架，都可以直接解析.vue单文件了





## Plugin

​	拓展webpack本身的一些功能，会运行在各种模块解析完成以后的打包编译阶段，比如对解析后的模块文件进行压缩等。比如css啊，图片啊已经处理完了，进入下一阶段，就是生成了，就是在生成之前，output之前去进行处理

**HtmlWebpackPlugin**

​	打包完以后，自动生成一个html文件，并把打包生成的js文件引入到该html中

```js
// 使用
1. 安装并引入到config.js中
const HtmlWebpackPlugin = require('html-webpack-plugin')

plugins: [
  new HtmlWebpackPlugin({ // #1
    title: 'app',
    filename: 'app.html',
    template: './public/index.html'
  })
]
```

1. 因为是一个类，所以要new



**CleanWebpackPlugin**

​	每次帮我们构建完了以后删除（清理）目录



**MiniCssExtractPlugin**

​	提取css到一个单独的文件中，原来的我们可以看到都是放到style标签中。既然这样当然`style-loader`就不能用了，这个插件较特殊，用在loader里，如下

```js
// 在原先style-loader的位置写
use: [
  { loader: MiniCssExtractPlugin.loader },
  'css-loader',
  'sass-loader'
]
// 同时
plugins: [
  new MiniCssExtractPlugin({
    filename: '[name].css'
  })
]
```



**sourceMap**

​	打包后的代码不利于我们调试和定位错误，尤其是线上的

```js
module.exports = {
  devtool: 'source-map' // #1
}
```

1. 就可以在浏览器里的source下面的`webpack://`栏下的src目录里看到对应的文件
2. 其实就会在打包后生成一个与该文件同名的但是是以`.map`结尾的文件



**WebpackDevServer**

​	每次代码修改后都需要重新编译打包，刷新浏览器，比较麻烦。（内置）

​	它就可以帮我们启动一个本地服务器

```js
"script": {
  serve: "webpack-dev-server"
}

// webpack.config.js
module.export = {
  devServer: {
    contentBase: './dist',
    open: true,	// 自动打开浏览器
    port: 8081,
    proxy: { // #1
      '/api': {
        target: 'http://localhost:7777',
        pathRewrite: {
          '^/api': ''
        }
      }
    },
    hot: true,	// #2
    hotOnly: true
  }
}
```



**1. Proxy**

也是webpack-dev-server自带的代理，如上



**2. HotModuleReplacement**

不刷新整个页面，只更新变化的部分，以前的化无法保持页面的操作状态

这个的原理是devServer会通过websocket通知浏览器端代码更新了



## 优化

打包时会将所有的东西都打到一个文件中，包括第三方库如axios，lodash等，显然不是我们想要的

```js
optimization: {
  splitChunks: {
    chunks: 'all'
  }
}
```

没懂

完。

















