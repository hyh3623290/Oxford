# 模块化

​	好多规范 - 像css也有，比如

```css
@import "reset.css"
```

# Webpack

## 环境 - node&npm

​	🌰 - nodejs版本很多 - 用nvm进行版本管理

​	🌰 - npm包管理工具 - package.json包的基本信息 - dependencies依赖

```js
npm install 包名
npm install 包名@x.x.x // 指定版本
npm install // 安装package.json所有依赖
npm install 包名 --save // 安装并写入依赖 -S
npm install 包名 --save-dev // 写入开发依赖 -D
npm uninstall 包名 // 删除某依赖
npm install 包名 -g // 全局 默认本地
```

🌰 - npm不稳定 - 淘宝镜像

```js
npm config set registry https://registry.npm.taobao.org
// 也可以nrm换源
```



## 初始化

```js
// 初始化
npm init -y
// 安装 webpack 和 webpack-cli到开发依赖
npm i webpack --save-dev
npm i webpack-cli --save-dev
```



🌰 - npx - 方便开发者访问 node_modules 内的 bin 命令行的小工具

```js
npm install -g npx
npx webpack -v
// 相当于
node ./node_modules/webpack/bin/webpack
```



此时写完代码执行 webpack 即可打包 - dist文件夹

```js
"scripts": {
  "dev": "webpack --mode development",
  "build": "webpack --mode production"
}
```



## babel

```js
npm i @babel/core babel-loader @babel/preset-env --save-dev
```

然后在项目的根目录下，创建一个 babel 的配置文件 .babelrc ，内容如下：

```js
{
	"presets": ["@babel/preset-env"]
}
```

执行npm run build



## webpack.config.js

🌰 - 只有一个webpack.config.js - 接收函数形式配置

```js
module.exports = (env, argv) => {
  return {
    mode: env.production ? 'production' : 'development',
    devtool: env.production ? 'source-maps' : 'eval'
  }
};
// env和argv：分别对应着环境对象和 Webpack-CLI 的命令行选项
```

🌰 - 如果需要异步加载一些 Webpack 配置需要做的变量 - Promise

```js
module.exports = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve({
        entry: './app.js'
        /* ... */
      });
    }, 5000);
  });
};
```



## 名词解释

| 参数     | 说明                                                         |
| :------- | :----------------------------------------------------------- |
| `entry`  | 项目入口                                                     |
| `module` | 开发中每一个文件都可以看做 module，模块不局限于 js，也包含 css、图片等 |
| `chunk`  | 代码块，一个 chunk 可以由多个模块组成                        |
| `loader` | 模块转化器，模块的处理器，对模块进行转换处理                 |
| `plugin` | 扩展插件，插件可以处理 chunk，也可以对最后的打包结果进行处理，可以完成 loader 完不成的任务 |
| `bundle` | 最终打包完成的文件，一般就是和 chunk 一一对应的关系，bundle 就是对 chunk 进行便意压缩打包等处理后的产出 |



## entry

### 单入口

### 多入口

这样会打包出来三个对应的 bundle

```js
module.exports = {
  entry: {
    home: 'path/to/my/entry/home.js',
    search: 'path/to/my/entry/search.js',
    list: 'path/to/my/entry/list.js'
  }
};
```

## output

```js
output: {
  filename: '[name].js', // 针对多入口
  path: __dirname + '/dist'
}
```

### publicPath

​	对于使用`<script>` 和 `<link>`标签时，当文件路径不同于他们的本地磁盘路径（由`output.path`指定）时，`output.publicPath`被用来作为`src`或者`link`指向该文件。这种做法在需要将静态文件放在不同的域名或者 CDN 上面的 时候是很有用的。

```js
module.exports = {
  output: {
    path: '/home/git/public/assets',
    publicPath: 'http://cdn.example.com/assets/'
  }
};
```

​	则输出：

```html
<head>
    <link href="http://cdn.example.com/assets/logo.png" />
</head>
```



# 原型

​	对象 - `__proto__`

​	函数 - `prototype`

​	对象的`__proto__` === 函数的`prototype`

🌰 - **原型链** - 对象本身没有某属性 - 去它的`__proto__`找 - 链式的结构，所以叫做“原型链”

🌰 - 如何判断这个属性是不是对象本身的属性呢？使用`hasOwnProperty`

🌰 - **执行上下文**

​	在一段 JS 脚本（即一个`<script>`标签中）执行之前，要先解析代码（所以说 JS 是解释执行的脚本语言），解析的时候会先创建一个 **全局执行上下文** 环境，先把代码中即将执行的（内部函数的不算，因为你不知道函数何时执行）变量、函数声明都拿出来。变量先暂时赋值为`undefined`，函数则先声明好可使用。这一步做完了，然后再开始正式执行程序。再次强调，这是在代码执行之前才开始的工作。

​	我们来看下上面的面试小题目，为什么`a`是`undefined`，而`b`却报错了，实际 JS 在代码执行之前，要「全文解析」，发现`var a`，知道有个`a`的变量，存入了执行上下文，而`b`没有找到`var`关键字，这时候没有在执行上下文提前「占位」，所以代码执行的时候，提前报到的`a`是有记录的，只不过值暂时还没有赋值，即为`undefined`，而`b`在执行上下文没有找到，自然会报错（没有找到`b`的引用）。

​	另外，一个函数在执行之前，也会创建一个 **函数执行上下文** 环境，跟 **全局上下文** 差不多

🌰 - **this**

​	`this`的值是在执行的时候才能确认，定义的时候不能确认！

​	`this`执行会有不同，主要集中在这几个场景中

*   作为构造函数执行，构造函数中
*   作为对象属性执行，`a.fn()`
*   作为普通函数执行，`fn1()`
*   用于`call` `apply` `bind`

🌰 - **作用域**

​	JS 没有块级作用域，只有全局作用域和函数作用域。这就是为何 jQuery、Zepto 等库的源码，所有的代码都会放在`(function(){....})()`中

🌰 - **闭包**

​	闭包主要有两个应用场景：

*   函数作为返回值，上面第一个
*   函数作为参数传递，看以下例子

```js
function F1() {
  var a = 100
  return function () {
      console.log(a)
  }
}
var f1 = F1()
var a = 200
f1() // 100
```

```js
function F1() {
  var a = 100
  return function () {
      console.log(a)
  }
}
function F2(f1) {
  var a = 200
  console.log(f1())
}
var f1 = F1()
F2(f1)
```

# Set 和 Map

Set 和 Map 都是 ES6 中新增的数据结构，是对当前 JS 数组和对象这两种重要数据结构的扩展。由于是新增的数据结构，目前尚未被大规模使用，但是作为前端程序员，提前了解是必须做到的。先总结一下两者最关键的地方：

*   Set 类似于数组，但数组可以允许元素重复，Set 不允许元素重复
*   Map 类似于对象，但普通对象的 key 必须是字符串或者数字，而 Map 的 key 可以是任何数据类型

🌰 - **Set**

Set 实例的属性和方法有

*   `size`：获取元素数量。
*   `add(value)`：添加元素，返回 Set 实例本身。
*   `delete(value)`：删除元素，返回一个布尔值，表示删除是否成功。
*   `has(value)`：返回一个布尔值，表示该值是否是 Set 实例的元素。
*   `clear()`：清除所有元素，没有返回值。

Set 实例的遍历，可使用如下方法

*   `keys()`：返回键名的遍历器。
*   `values()`：返回键值的遍历器。不过由于 Set 结构没有键名，只有键值（或者说键名和键值是同一个值），所以`keys()`和`values()`返回结果一致。
*   `entries()`：返回键值对的遍历器。
*   `forEach()`：使用回调函数遍历每个成员。



🌰 - **Map**

Map 实例的属性和方法如下：

*   `size`：获取成员的数量
*   `set`：设置成员 key 和 value
*   `get`：获取成员属性值
*   `has`：判断成员是否存在
*   `delete`：删除成员
*   `clear`：清空所有

Map 实例的遍历方法有：

*   `keys()`：返回键名的遍历器。
*   `values()`：返回键值的遍历器。
*   `entries()`：返回所有成员的遍历器。
*   `forEach()`：遍历 Map 的所有成员。

























# - -

--

# cunning

🌰 - **why webpack**

​	Grunt，Gulp - 做不到按需加载 - 不管页面用不用全部打包 => webpack从入口文件开始完成

🌰 - **loader & plugin**

​	loader 是源代码的处理器， plugin 解决的是 loader 处理不了的事情

🌰 - **hash & chunkhash & contenthash**

1. hash是整个项目的 hash 值，每编译一次变化一次，一个项目的所有文件hash值都是相同的。hash 无法实现前端静态资源在浏览器上长缓存，这时候应该使用 chunkhash。
2. chunkhash根据不同的入口文件（entry），构建对应的 chunk，生成相应的hash。只要组成 entry 的模块文件没有变化，则对应的 hash 也是不变的，所以一般项目优化时，会将公共库代码拆分到一 起，因为公共库代码变动较少的，使用 chunkhash 可以发挥最长缓存的作用；
3. 使用 chunkhash 存在一个问题，当在一个 JS 文件中引入了 CSS 文件，编译后它们的 hash 是 相同的。而且，只要 JS 文件内容发生改变，与其关联的 CSS 文件 hash 也会改变，针对这种情况，可以把 CSS 从 JS 中使用mini-css-extract-plugin 或 extract-text-webpack-plugin抽离出来并使用 contenthash。

🌰  - **js数据类型**

- Boolean，String，Null，Number，Undefined，Symbol
- typeof - 多个function和object



🌰 - **同步异步题目**

```js
var a = true;
setTimeout(function(){
    a = false;
}, 100)
while(a){
    console.log('while执行了')
}
```

​	这是一个很有迷惑性的题目，不少候选人认为`100ms`之后，由于`a`变成了`false`，所以`while`就中止了，实际不是这样，因为JS是单线程的，所以进入`while`循环之后，没有「时间」（线程）去跑定时器了，所以这个代码跑起来是个死循环！

















🥣碗 - 完成✅ - 苹果🍎 - 栗子🌰 - 橘子🍊 - 葡萄🍇🇵🇹🇪🇸🇨🇳🇺🇸 - 雨🌧️ 云☁️😷 - 恶心🤢🤮 - 哈哈😄

哭😭 - 生气😠 