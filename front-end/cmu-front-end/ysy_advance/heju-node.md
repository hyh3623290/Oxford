

`npm root -g`	可以看到全局安装的位置

fs模块读写文件

目录操作

buffer：toString()

stream：防止内存爆出，把文件分成64kb的小文件

`node xxx.js`可以执行该文件



## npm

​	npm会有一个下载的源地址：`npm config list`（查看源地址register），就是从这样的一个服务器下载的

​	那我们能不能自己写一个发布到官方的源地址上呢（记得改回源地址，不要是淘宝原啥的）

​	`npm config set xxx`

​	发布的化一定得有个package.json，写完以后`npm publish`（都不需要审核，但是得注册账号），发布时候就可以直接安装了，npm unpublish --force就可以删除了

​	一般不推荐用cnpm（ReactNative里提示说路径会比较奇怪）



## nvm

想切换node版本的时候可以用，windows的话需要用nvm-windows（不能用npm安装）



## 创建服务

```js
const http = require("http")
const server = http.createServer((req, res) => { // #1
  res.writeHead(200, {"content-type":"text/html;charset=utf8"}) // #2
  res.write("<h1>hello world</h1>")
  res.end()
})
server.listen(8888)

// 一般情况下其实这么做
let indexData = fs.readFileAsync('./index.html')
res.write(indexData)
```

然后就可以通过nodemon启动

`nodemon index.js`

现在在浏览器打开：localhost:8888就能看到hello world

1. 前端的东西一般在req里，后端的在res里
2. 防止中文乱码，MIME协议，就是不同的文件格式对应设置不同的头部

**实现简单路由**

```js
const http = require("http")
const server = http.createServer((req, res) => {
  res.writeHead(200, {"content-type":"text/html;charset=utf8"})
  if(req.url === '/') { // #1
    let indexData = fs.readFileAsync('./index.html')
    res.write(indexData)      
  } else if(req.url === '/detail') { // #2
    let indexData = fs.readFileAsync('./detail.html')
    res.write(indexData)     
  } else if(req.url === '/getData') { // #3
    let obj = {
      info: 123,
      code: 0
    }
    res.write(JSON.stringify(obj))
  }
  res.end()
})
server.listen(8888)
```

1. 加载首页
2. 加载详情页
3. 模拟一个后端接口



## 状态码

1. 相关的就是正在请求，继续请求之类，如websocket
2. 相关的就是成功
3. 相关的就是重定向
4. 相关的就是客户端错误
5. 相关的就是服务器错误



## koa

```js
const Koa = require("koa")
const app = new Koa()

app.use(async (ctx) => { // #1
  ctx.status = 404 // #3
  ctx.response.body = 'hello world' // #2
})
app.listen(8887)
```

1. 相当于把req和res都整合到一起了，createServer
2. 非常方便，中文也不需要设置不会乱码，其实也可以直接`ctx.body = xxx`，它会有一个相当于别名的东西
3. 设置状态码



## koa-router

也是需要安装

```js
const Koa = require("koa")
const Router = require("koa-router")
const app = new Koa()
const router = new Router()

router.get('/index', ctx => {
  ctx.body = "首页"
})
router.get('/detail', ctx => {
  ctx.body = "详情页"
})
router.get('/getData', ctx => {
  ctx.body = { // #1
    name: 'lisi',
    age: 10
  }
})


app.use(router.routes())
app.listen(8887)
```

1. 会自动帮我们进行处理



## koa-static

加载静态资源，如图片，css，js等（02集2.24没有仔细看）

```js
const Koa = require("koa")
const Router = require("koa-router")
const static = require("koa-static")

const app = new Koa()
const router = new Router()

app.use(static(__dirname + '/static'))

app.use(router.routes())
```



## mysql

数据持久化保存

```js
const mysql = require('mysql2')
const connection = 	mysql.createConnection({
  host: 'localhost', // 服务器
  user: 'root',
  password: 'xxx'
  database: 'test' // 数据库名称
  charset: "utf8"
})

router.get('/', ctx => {
  let sql = ""
  connection.query(sql, (err, result) => {
    ...
  })
})
```























---------------------------------------------

egg是相当于koa又做了一层封装



