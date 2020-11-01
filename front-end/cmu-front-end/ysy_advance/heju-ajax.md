# 手写一个ajax以及跨域

```js
var xhr = new XMLHttpRequest()
xhr.open('GET', '/api', false)// false表示异步
xhr.onreadystatechange = function(){
  if(xhr.readyStatus == 4){
    if(xhr.status == 200){
      alert(xhr.responseText)
    }
  }
}
xhr.send(null)
// send以后会随时监听状态变化，每次readyStatus变化都会触发上面的事件
```



**readyStatus:**

- 0 - (未初始化，还没有调用send方法)
- 1 - 正在发送请求
- 2 - 已经接受到全部响应内容
- 3 - 正在解析响应内容
- 4 - 响应内容解析完成，可以在客户端调用了



**status：**



- 2xx - 表示成功处理请求
- 3xx - 需要重定向，浏览器拿到这个值就自己跳了，直接跳转
- 4xx - 客户端请求错误
- 5xx - 服务端错误
- 跨域的几种实现方式：小公司比较少，大公司各个域名都不一样，比如下面就都是跨域
  imooc.com	m.imooc.com	coding.imooc.com
  浏览器有同源策略，不允许ajax访问其他域接口，条件：协议，域名，端口
  但是三个标签允许跨域加载资源：

```js
<img src=xxx />
<link href=xxxx /> // link加载css的标签
<script src=xxx> // 可以用于JSONP
```

- JSONP实现原理：

- - 加载http://.../index.html时服务器端不一定真的有index.html这个文件
  - 服务器是可以根据请求，动态的生成一个文件返回的，同理script里的src

```js
<script>
  window.callback = function(data){
    console.log(data) // 这是我们跨域得到的信息
  }    
</script>
<script src="http://.../api.js"></script>
// 这个地址返回的是callback({x:100,y:200})
// 我们定义了一个callback函数,而js返回的信息呢，正好一加载一执行，callback函数就拿到data了
```

另一种方式是服务器端设置http header（http请求头）

第二个参数可以有一些白名单限制，“http://a.com”





# 前后端交互

## ajax

ajax的要点就是页面不刷新，实现把数据提交到后台

ajax英文翻译即是异步的JavaScript和XML

有时候后端忘记写头部了，比如xml的，我们也可以前端改response的头部

```js
xhr.overrideMimeType("text/xml") // content-type
```









## 文件上传

```js
// <input type="file" />
document.querySelector("button").onclick = function() {
  let files = document.querySelector("input").files
  console.log(files) // #1
    
  let file = files[0]
  let form = new FormData() // #4
  form.append("img", file) // #2
    
  let xhr = new XMLHttpRequest()
  xhr.open("post", "/upload", true) // #3
  xhr.onload = function() {
    console.log(xhr.responseText)
  }
  xhr.send(form)
}
// 后台
router.post("/upload", (ctx, next) => {
  console.log(ctx.request.files.body) // #5
  console.log(ctx.request.files.img)
})
```

1. 拿到文件对象，是个类数组，打印出来可以看到里面有文件的各种属性，如大小，名称，类型，修改时间等

2. 这个img相当于拿到的这个file的名字（name），其实也可以添加其他数据，比如

   `form.append("name", "张三")`

3. 文件只能用post的哦，只能通过正文的方式进行传递。true表示异步

4. 使用formData会自动帮我们设置Content-Type

5. body里的内容就是：`name: {...}, img: {...}`，下面的img就是对应#2里的img名字

**进度及取消上传**

```html
<input type="file" class="myFile"/>
进度：<progress value="10" max=100></progress>
速度：<span class="speed">20b/s</span>
<button>点击上传</button>
<button>取消上传</button>
```

```js
let xhr = new XMLHttpRequest()

let btns = document.querySelectorAll("button")
btns[0].onclick = function() {
  let file = document.querySelector(".myFile").files[0]
  let form = new FormData()
  form.append("myFile", file)
    
  xhr.open("post", "/upload", true)
  xhr.onload = function() {
    console.log(xhr.responseText)
  }
  xhr.upload.onloadstart = function() {
    console.log("开始上传")
  }
  xhr.upload.onprogress = function(evt) {
    console.log("上传中") // #2
    evt.loaded/evt.total // #3
  }
  xhr.upload.onload = function() {
    console.log("上传成功")
  }
  xhr.upload.onloadend = function() {
    console.log("上传完成") // #1
  }
  xhr.upload.onabort = function() {
    console.log("取消上传")
  }
  xhr.send(file)
}

btns[1].onclick = function() {
  xhr.abort() // 取消上传
}
```

1. 不管上传成功还是失败都有一个上传完成的钩子
2. 可以发现正在上传打印了好几次
3. 当前上传的文件大小及总文件大小可以通过这个方式拿到

# WebSocket



​	没有跨域问题，但是有些兼容性问题，可以用socket.io库。websocket是基于TCP协议，sse是基于Http协议的，所以也还有跨域问题。

**全双工通信协议**

**后端部分**

```js
// 不基于插件框架的写法
var WebSocketServer = require('ws').Server,
wss = new WebSocketServer({ port: 8181 });
wss.on('connection', function (ws) { // #1
  console.log('client connected');
  ws.on('message', function (message) {
    //监听接收的数据
    console.log(message);
  });
  setInterval(() => {
    let somedata = {
      name: "张三",
      age: 20
    }
    ws.send(JSON.stringify(somedata));
  }, 1000);
});
// socket.io写法
```

1. 有一个客户端连接，就会生成一个ws对象



**前端部分：和sse很像**

```js
var ws = new WebSocket("ws://localhost:8181");
ws.onopen = function () {
  console.log("连接成功")
}
ws.send("客户端数据");
ws.onmessage = function(e){
  console.log(e.data); // #1
}
ws.close();

// 使用socket.io
let socket = io.connect("/")
socket.on("getData", function(data) {
  console.log(data)
})
socket.emit("addData", "数据")
```

1. 数据一般放在data里，在调试环境中是显示ws，可以看对应的websocket信息。只要服务器一有推送，就会触发这个回调，推多少次触发多少次。
2. ws的状态码可以看到是101，就表示一直在请求



# 从输入url到得到html的详细过程



- 加载资源的形式
- 加载一个资源的过程

- - 浏览器根据DNS服务器得到域名的IP地址
  - 向这个IP的机器发送http请求
  - 服务器收到，处理并返回http请求
  - 浏览器得到返回内容

- 浏览器渲染页面的过程

- - 根据html结构生成一个dom树
  - 根据css生成CSSOM树
  - 将DOM和CSSOM整合形成一个render树
  - 浏览器根据renderTree开始渲染和展示
  - 遇到script标签时会执行并阻塞渲染





-----------------------------------------