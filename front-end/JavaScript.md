# Event Loop

​	当我们执行 JS 代码的时候其实就是往执行栈中放入函数，那么遇到异步代码的时候该怎么办？其实当遇到异步的代码时，会被**挂起**并在需要执行的时候加入到 Task（有多种 Task） 队列中。一旦执行栈为空，Event Loop 就会从 Task 队列中拿出需要执行的代码并放入执行栈中执行，所以本质上来说 JS 中的异步还是同步行为。

​	不同的任务源会被分配到不同的 Task 队列中，任务源可以分为 **微任务**（microtask） 和 **宏任务**（macrotask）。在 ES6 规范中，microtask 称为 `jobs`，macrotask 称为 `task`。下面来看以下代码的执行顺序

```js
console.log('script start')

async function async1() {
  await async2()
  console.log('async1 end')
}
async function async2() {
  console.log('async2 end')
}
async1()

setTimeout(function() {
  console.log('setTimeout')
}, 0)

new Promise(resolve => {
  console.log('Promise')
  resolve()
})
  .then(function() {
    console.log('promise1')
  })
  .then(function() {
    console.log('promise2')
  })

console.log('script end')
// script start => async2 end => 
// Promise => script end => promise1 => promise2 => 
// async1 end => setTimeout
// 同步 > promise > setTimeout
```



# 跨域

​	因为浏览器出于安全考虑，有同源策略。也就是说，如果协议、域名或者端口有一个不同就是跨域，Ajax 请求会失败。

​	那么是出于什么安全考虑才会引入这种机制呢？ 其实主要是用来防止 CSRF 攻击的。简单点说，CSRF 攻击是利用用户的登录态发起恶意请求。

​	也就是说，没有同源策略的情况下，A 网站可以被任意其他来源的 Ajax 访问到内容。如果你当前 A 网站还存在登录态，那么对方就可以通过 Ajax 获得你的任何信息。当然跨域并不能完全阻止 CSRF。

​	然后我们来考虑一个问题，请求跨域了，那么请求到底发出去没有？ 请求必然是发出去了，但是浏览器拦截了响应。你可能会疑问明明通过表单的方式可以发起跨域请求，为什么 Ajax 就不会。因为归根结底，跨域是为了阻止用户读取到另一个域名下的内容，Ajax 可以获取响应，浏览器认为这不安全，所以拦截了响应。但是表单并不会获取新的内容，所以可以发起跨域请求。同时也说明了跨域并不能完全阻止 CSRF，因为请求毕竟是发出去了。

## JSONP

​	JSONP 的原理很简单，就是利用 `<script>` 标签没有跨域限制的漏洞。通过 `<script>` 标签指向一个需要访问的地址并提供一个回调函数来接收数据当需要通讯时。

```html
<script src="http://domain/api?param1=a&param2=b&callback=jsonp"></script>
<script>
    function jsonp(data) {
    	console.log(data)
	}
</script>
```

​	JSONP 使用简单且兼容性不错，但是只限于 `get` 请求。

​	在开发中可能会遇到多个 JSONP 请求的回调函数名是相同的，这时候就需要自己封装一个 JSONP，以下是简单实现

## CORS

​	CORS 需要浏览器和后端同时支持。IE 8 和 9 需要通过 `XDomainRequest` 来实现。

​	浏览器会自动进行 CORS 通信，实现 CORS 通信的关键是后端。只要后端实现了 CORS，就实现了跨域。

​	服务端设置 `Access-Control-Allow-Origin` 就可以开启 CORS。 该属性表示哪些域名可以访问资源，如果设置通配符则表示所有网站都可以访问资源。

​	虽然设置 CORS 和前端没什么关系，但是通过这种方式解决跨域问题的话，会在发送请求时出现两种情况，分别为简单请求和复杂请求。

以 Ajax 为例，当满足以下条件时，会触发简单请求

1. 使用下列方法之一：
   - `GET`
   - `HEAD`
   - `POST`
2. `Content-Type` 的值仅限于下列三者之一：
   - `text/plain`
   - `multipart/form-data`
   - `application/x-www-form-urlencoded`

​    请求中的任意 `XMLHttpRequestUpload` 对象均没有注册任何事件监听器； `XMLHttpRequestUpload` 对象可以使用 `XMLHttpRequest.upload` 属性访问。

**复杂请求**

​	那么很显然，不符合以上条件的请求就肯定是复杂请求了。

​	对于复杂请求来说，首先会发起一个预检请求，该请求是 `option` 方法的，通过该请求来知道服务端是否允许跨域请求。

​	对于预检请求来说，如果你使用过 Node 来设置 CORS 的话，可能会遇到过这么一个坑。

​	以下以 express 框架举例：

```js
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*')
  res.header('Access-Control-Allow-Methods', 'PUT, GET, POST, DELETE, OPTIONS')
  res.header(
    'Access-Control-Allow-Headers',
    'Origin, X-Requested-With, Content-Type, Accept, Authorization, Access-Control-Allow-Credentials'
  )
  next()
})
```

​	该请求会验证你的 `Authorization` 字段，没有的话就会报错。

​	当前端发起了复杂请求后，你会发现就算你代码是正确的，返回结果也永远是报错的。因为预检请求也会进入回调中，也会触发 `next` 方法，因为预检请求并不包含 `Authorization` 字段，所以服务端会报错。

​	想解决这个问题很简单，只需要在回调中过滤 `option` 方法即可

```js
res.statusCode = 204
res.setHeader('Content-Length', '0')
res.end()
```

**document.domain**

​	该方式只能用于**二级域名相同**的情况下，比如 `a.test.com` 和 `b.test.com` 适用于该方式。

​	只需要给页面添加 `document.domain = 'test.com'` 表示二级域名都相同就可以实现跨

​	这种方式通常用于获取嵌入页面中的第三方页面数据。一个页面发送消息，另一个页面判断来源并接收消息

```js
// 发送消息端
window.parent.postMessage('message', 'http://test.com')
// 接收消息端
var mc = new MessageChannel()
mc.addEventListener('message', event => {
  var origin = event.origin || event.originalEvent.origin
  if (origin === 'http://test.com') {
    console.log('验证通过')
  }
})
```

# 面向对象

​	首先它是一个编程思想吧，js本身就是基于面向对象构建出来的。比如说，js里面有很多内置类...像Promise就是ES6新增的内置类，我们一般都是基于new Promise来创建一个实例来管理异步编程。还有用的Vue，React也都是基于面向对象的，平常开发的时候也都是创建他们的实例来进行操作（new Vue，class extends component等等）。

  不过js的面向对象跟其他语言还不太一样，如重载，一般的语言是多个方法方法名相同，参数个数或者类型不同来实现，但是js的话相同的函数名后者就覆盖了前者，不过我们可以写一个方法，在方法里面判断参数类型来实现重载

  面向对象三大概念：重载，重写和继承。重写就是子类可以重写父类的方法，继承就是子类继承父类的属性和方法。

# 存储

 cookie

​	一般服务器生成，可以设置过期时间，4K，每次都会携带在 header 中，对于请求性能影响

indexDB

​	除非被清理，否则一直存在，存储无限





























































------------------------------------------------------------------

​	如果你对设计模式有所了解的话，就会知道，一个 API 并非越庞大越复杂才越优秀。或者说得更直接一点，庞大和复杂的 API 往往会带来维护层面的“灾难”。getDerivedStateFromProps







