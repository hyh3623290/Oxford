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

​	多入口对应输出文件 `[name].js`，会输出两个文件( bundle )。一个输入文件对应一个chunk，对应一个bundle

# output

占位符 - hash，chunkhash，name，id

```js
output: {
  path: path.resolve(__dirname, "./dist")
  filename: "[name]-[hash:6].js"
}
```

hash的作用就是缓存

# loader

​	模块解析，webpack默认只认识json和js ，其他的就需要loader了

🌰 **css-loader**

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

🌰 **css modules**

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

🌰 **postcss-loader**

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

🌰 **file-loader**

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

1.14.19

# Plugin

​	插件，对webpack的补充

clean-webpack-plugin

​	实际就是删了重建打包文件，做清理工作

```js
// 1.引入 - require
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
// 2.使用 - new
plugins: [new CleanWebpackPlugin()]
```









# ---

⚠️ mode会把process.env设置成对应的，webpack相关的好像都是开发依赖，比如各种loader