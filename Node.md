# 异步IO

​	js单线程 - 所有io处理如文件读取，网络IO缓慢，要交给异步方法进行处理

```js
const fs = require('fs')

const data = fs.readFildSync('./data.js')
console.log(data.toString())
```

​	如果文件是2g呢，如果在地球那边呢，那么cpu会等待你很长时间

```js
// 异步调⽤
fs.readFile('./conf.js', (err, data) => {
 if (err) throw err;
 console.log(data);
})
```

​	还是那个问题，怎么样变优雅呢

```js
// promisify
const {promisify} = require('util')
const readFile = promisify(fs.readFile)
readFile('./conf.js').then(data=>console.log(data))
```

Async await可以try catch捕获

```js
// async/await
(async () => {
 const fs = require('fs')
 const { promisify } = require('util')
 const readFile = promisify(fs.readFile)
 const data = await readFile('./index.html')
 console.log('data',data)
})()
```



# Buffer

​	就是为了处理二进制的

​	 ⽤于在 TCP 流、⽂件系统操作、以及其他上下⽂中与⼋位字节流进⾏交互。 ⼋位字节组 成的数组，可以有效的在JS中存储⼆进制数据

```js
// 创建⼀个⻓度为10字节以0填充的Buffer
const buf1 = Buffer.alloc(10);
console.log(buf1); // <Buffer 00 00 00 00 00 00 00 00 00 00>
// 每个00是一个字节

const buf2 = Buffer.from('a')
console.log(buf2,buf2.toString()) //<Buffer 61>

const buf3 = Buffer.from('Buffer创建⽅法');
console.log(buf3.toString('utf-8'));
```

# Stream

```js
//⼆进制友好，图⽚操作
const fs = require('fs')
const rs2 = fs.createReadStream('./01.jpg') // 读的流
const ws2 = fs.createWriteStream('./02.jpg') // 写的流
rs2.pipe(ws2); // 连在一起
```

流不占用内存

# http

简单的web服务

```js
const http = require('http');
const server = http.createServer((request, response) => {
 console.log('there is a request');
 response.end('a response from server');
});
server.listen(3000);
```

显示一个html首页

```js
const http = require('http');
const server = http.createServer((request, response) => {
	const {url, method, headers} = request;
  if (url === '/' && method === 'GET') {
 		fs.readFile('index.html', (err, data) => {
 			if (err) {
 				response.writeHead(500, { 'Content-Type':'text/plain;charset=utf-8' });
 				response.end('500，服务器错误');
 				return;
 			}
		  response.statusCode = 200;
 			response.setHeader('Content-Type', 'text/html');
 			response.end(data);
 		});
 } else if (url === '/users' && method === 'GET') {
 		response.writeHead(200, { 'Content-Type': 'application/json' });
 		response.end(JSON.stringify([{name:'tom',age:20}]));
 } else if (headers.accept.indexOf('image/*') !== -1 && method === 'GET') {
    fs.createReadStream('.' + url).pipe(response);        
 } else {
 		response.statusCode = 404;
 		response.setHeader('Content-Type', 'text/plain;charset=utf-8');
 		response.end('404, ⻚⾯没有找到');
 }
});
server.listen(3000);	
```

- Accept代表发送端（客户端）希望接受的数据类型。 ⽐如：`Accept：text/xml;` 代表客户端希望 接受的数据类型是xml类型。 
- Content-Type代表发送端（客户端|服务器）发送的实体数据的数据类型。 ⽐如：`ContentType：text/html; `代表发送端发送的数据格式是html。 
- ⼆者合起来， `Accept:text/xml； Content-Type:text/html `，即代表希望接受的数据类型是xml格 式，本次请求发送的数据的数据格式是html。

# CLI⼯具

```js
npm init -y
npm i commander download-git-repo ora handlebars figlet clear chalk open -s
```

​	先建立一个bin文件夹作为程序的入口，再写一个kkb.js

bin/kkb.js

```java
// 指定脚本解释器为node
#!/usr/bin/env node
console.log('cli.....')
```

package.json

```js
"bin": {
 "kkb": "./bin/kkb.js"
},
// 注册一下，执行kkb的指令的时候指向的是冒号后面那个文件
```

```js
npm link // 这个命令相当于把我们现在的kkb注册到全局，相当于把这个包npm i -g到安装全局的效果
// 这时候如果执行kkb就会打印那个 cli.....
```



**定制命令⾏界⾯**

bin/kkb.js

```js
const program = require('commander')
program.version(require('../package').version)
program
 .command('init <name>')
 .description('init project')
 .action(name => {
 console.log('init ' + name)
})
program.parse(process.argv)
```

1.38.40 D

# Koa

​	概述：Koa 是⼀个新的 web 框架， 致⼒于成为 web 应⽤和 API 开发领域中的⼀个更⼩、更富有 表现⼒、更健壮的基⽯。升级版的express，koa2完全使⽤Promise并配合 async 来实现异步。

```js
const Koa = require('koa')
const app = new Koa()
app.use((ctx, next) => {
 ctx.body = [
   {
    name: 'tom'
   }
 ]
 next()
})
// next就是执行下一个中间件
app.use(() => {
  ...
})
```

28.54 D



























------------