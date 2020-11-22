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

















# - -

# cunning

🌰 - **why webpack**

​	Grunt，Gulp - 做不到按需加载 - 不管页面用不用全部打包 => webpack从入口文件开始完成

🌰 - **loader & plugin**

​	loader 是源代码的处理器， plugin 解决的是 loader 处理不了的事情

🌰 - **hash & chunkhash & contenthash**

1. hash是整个项目的 hash 值，每编译一次变化一次，一个项目的所有文件hash值都是相同的。hash 无法实现前端静态资源在浏览器上长缓存，这时候应该使用 chunkhash。
2. chunkhash根据不同的入口文件（entry），构建对应的 chunk，生成相应的hash。只要组成 entry 的模块文件没有变化，则对应的 hash 也是不变的，所以一般项目优化时，会将公共库代码拆分到一 起，因为公共库代码变动较少的，使用 chunkhash 可以发挥最长缓存的作用；
3. 使用 chunkhash 存在一个问题，当在一个 JS 文件中引入了 CSS 文件，编译后它们的 hash 是 相同的。而且，只要 JS 文件内容发生改变，与其关联的 CSS 文件 hash 也会改变，针对这种情况，可以把 CSS 从 JS 中使用mini-css-extract-plugin 或 extract-text-webpack-plugin抽离出来并使用 contenthash。









🥣碗 - 完成✅ - 苹果🍎 - 栗子🌰 - 橘子🍊 - 葡萄🍇🇵🇹🇪🇸🇨🇳🇺🇸 - 雨🌧️ 云☁️😷 - 恶心🤢🤮 - 哈哈😄

哭😭 - 生气😠 