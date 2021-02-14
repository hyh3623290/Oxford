# EcmaScript 6

## let const var区别

1. 全局作用域下使用 `let` 和 `const` 声明变量，变量并不会被挂载到 `window` 上
2. 没有变量提升
3. 块级作用域



## 箭头函数

​	箭头函数存在的意义，第一写起来更加简洁，第二可以解决 ES6 之前函数执行中`this`是全局变量的问题，看如下代码

```js
function fn() {
    console.log('real', this)  // {a: 100} ，该作用域下的 this 的真实的值
    var arr = [1, 2, 3]
    // 普通 JS
    arr.map(function (item) {
        console.log('js', this)  // window 。普通函数，这里打印出来的是全局变量，令人费解
        return item + 1
    })
    // 箭头函数
    arr.map(item => {
        console.log('es6', this)  // {a: 100} 。箭头函数，这里打印的就是父作用域的 this
        return item + 1
    })
}
fn.call({a: 100})
```

​	也就是说只跟外面的**函数**的this有关系



## Module

​	ES6 中模块化语法更加简洁，直接看示例。

​	如果只是输出一个唯一的对象，使用`export default`即可，代码如下

```js
// 创建 util1.js 文件，内容如
export default {
    a: 100
}

// 创建 index.js 文件，内容如
import obj from './util1.js'
console.log(obj)

```

​	如果想要输出许多个对象，就不能用`default`了，且`import`时候要加`{...}`，代码如下

```js
// 创建 util2.js 文件，内容如
export function fn1() {
    alert('fn1')
}
export function fn2() {
    alert('fn2')
}

// 创建 index.js 文件，内容如
import { fn1, fn2 } from './util2.js'
fn1()
fn2()

```



## class

```js
// ES5
function MathHandle(x, y) {
  this.x = x
  this.y = y
}

MathHandle.prototype.add = function() {
  return this.x + this.y
}

// ES6
class MathHandle(x, y) {
  constructor(x, y) {
    this.x = x
    this.y = y
  }
  
  add() {
    return this.x + this.y
  }
}

// 使用方法一样, 上面两种类完全等价
const m = new MathHandle(1, 2)
m.add()
```

​	注意class中函数的写法没有function

**继承写法**

```js
// ES5
function Animal() {}
function Dog() {}
Dog.prototype = new Animal()

// ES6
class Dog extends Animal {
  constructor(name) {
    super(name)
  }
}

// ---
var hashiqi = new Dog()
```



## Set

​	Set 类似于数组，但数组可以允许元素重复，Set 不允许元素重复

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

```js
let set = new Set(['aaa', 'bbb', 'ccc']);

for (let item of set.keys()) {
  console.log(item);
}
// aaa
// bbb
// ccc

for (let item of set.entries()) {
  console.log(item);
}
// ["aaa", "aaa"]
// ["bbb", "bbb"]
// ["ccc", "ccc"]

// 遍历器对象一般用for of循环
```



## Map

​	Map 类似于对象，但普通对象的 key 必须是字符串或者数字，而 Map 的 key 可以是任何数据类型

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



## reduce

​	对数组的每一个元素执行一个reducer函数，将其结果汇总成一个单一的值

```js
arr.reduce(reducer, 5)
```

​	reducer函数接收四个参数，第一个是累加器，返回的值会成为累加器，其他三个参数就是`item, index, array`



## 解构

```js
// 数组
const [a, ,c] = [1,2,3]

// 对象
const school = {
  classes: {
    stu: {
      name: 'Bob',
      age: 24,
    }
  }
}
const { classes: { stu: { name } }} = school
const { classes: { stu: { name: newName } }} = school // 同时重命名
```



## 扩展运算符...

```js
console.log( ...['haha', 'hehe', 'xixi'] ) // haha hehe xixi

console.log( ...{a: 1, b: 2} ) // Uncaught TypeError: Found non-callable @@iterator
console.log({ ...{a: 1, b: 2} })

function mutiple(...args) {
  console.log(args)
}
mutiple(1, 2, 3, 4) // [1, 2, 3, 4], 没有...就只输出1
```

​	理解就是拆掉外面的括号，对象的话拆开后外面必须再包一层，不然输出就报错。在函数里的话可以把函数的多个入参收敛进一个数组里



## 类数组

啥是类数组对象，ECMA-262对它的定义是：

1. 它必须是一个对象
2. 它有 length 属性

```js
const arrayLike = {0: 'Bob', 1: 'Lucy', 2: 'Daisy', length: 3 } // 这是一个类数组对象
```

如何把类数组对象转换为真正的数组

- Array原型上的slice方法如果不传参数的话会返回原数组的一个拷贝，可以用此方法转换类数组到数组：

```js
const arr = Array.prototype.slice.call(arrayLike);
```

- Array.from方法
- 扩展运算符——"…"也可以把类数组对象转换为数组，前提是这个类数组对象上部署了遍历器接口。在这个例子中，arrayLike 没有部署遍历器接口，所以这条路走不通。



## 模板字符串

```js
// 在模板字符串中，空格、缩进、换行都会被保留
let list = `
	<ul>
		<li>列表项1</li>
		<li>列表项2</li>
	</ul>
`;

console.log(message); // 顺利输出，不存在报错
```

其他字符串方法：includes、startsWith、endsWith，repeat

## Proxy

​	代理对象

​	如果我们想要监视某个对象的属性的读写，可以使用`Object.defineProperty`添加属性，从而捕获到对象中的属性读写过程。Proxy是为对象专门提供的一个访问代理器的，可以轻松的监视到对象读写过程

```js
const person = {
  name: 'John',
  age: 20
}

const personProxy = new Proxy(person, {
  get(target, property) {
    return 100
    return property in target ? target[property] : 'default'
  },
  set(target, property, value) {
    if(property === 'age') {
      if(!Number.isInteger(value)) {
        throw new TypeError('balabala')
      }
      target[property] = value
    }
  },
  // 更多额外操作
  deleteProperty(target, property) { },
})

console.log(personProxy.name) // 100
```

- Proxy更为强大，能监听到更多对象操作，而defineProperty只能监听到对象的读写。

  例如delete，或者对对象方法的调用

- 更好的支持数组对象的监视

- 以非侵入的方式监管了对象的读写，也就是说一个已经定义好的对象我们不需要对对象本身去做任何的操作就可以监视到它内部成员的读写，而要求我们必须要通过特定的方式单独去定义对象当中一些需要被监视的属性，就是需要做很多额外的操作，如数组方法

## Reflect

​	静态类，不能new来实例化，只能使用一些静态类的静态方法：`Reflect.get()`，类似于Math也只有静态方法。它封装了一系列对对象的底层操作。有13个方法，和Proxy的方法一样。其实的话，Proxy里面默认就是调用了Reflect的方法，比如没有自己写get方法的话默认就是返回Reflect的get方法。

​	为什么要有这个，价值在哪：统一提供了一套用于操作对象的API

```js
'name' in obj
delete obj['age']
Object.keys(obj)
// 如上，太乱，不够统一

Reflect.has(obj, 'name')
Reflect.deleteProperty(obj, 'age')
Reflect.ownKeys(obj)
```

## Iterator

​	可迭代对象，我们发现for of 只能遍历数组类型的数据结构，为了给各个数据结构提供一种统一的遍历方式，就有了Iterator接口，也就是使用for of的前提。我们发现所有类数组的对象的`__proto__`上都有`Symbol(Symbol.Iterator)`这个属性，它是一个方法，我们直接调用试一下

```js
arr[Symbol.iterator]() // Array Iterator {} 数组的迭代器对象
```

​	展开这个对象后发现它有一个next方法

```js
const iterator = arr[Symbol.iterator]()
iterator.next() // { value: 'foo', done: false }
```

​	此时就能够想到这个迭代器内部应该维护了一个数据指针，每调用一次next，指针便往后移一位

🍇 遍历对象

```js
const obj = {
  store: ['foo', 'bar', 'baz'],
  [Symbol.iterator]: function() { // 实现可迭代接口 Iterable Symbol.iterator
    let index = 0
    const self = this
    return {
      next: function() { // 实现迭代器接口 Iterator next
        const result = { // 迭代结果接口 IterationResult
          value: self.store[index],
          done: index++ >= self.store.length
        }
        return result
      }
    }
  }
}
```

​	这个意义在哪呢，事实上这个就是迭代器模式。

​	假定我们现在正在协同开发一个任务清单应用，我的任务是设计一个用于存放所有任务的对象，我为了可以更有结构的去记录每一个数据，设计一个对象结构。你的任务是把我定义的这个对象当中所有的任务项罗列呈现到界面上，此时，对于你而言，你就必须要了解这个对象的数据结构才行，你可以分别遍历我的life和learn，那如果突然我的数据结构变化了，那你还得修改代码。所以如果我能提供一个统一的遍历接口，那你就不用关心我对象内部的结构，更不用担心我的结构改变产生的影响

```js
// 我的代码
const todos = {
  life: ['a', 'b', 'c'],
  learn: ['d', 'e', 'f'],
  [Symbol.iterator]: function() {
    const all = [...this.life, ...this.learn]
    let index = 0
    return {
      next: function() {
        return {
          value: all[index],
          done: index++ >= all.length
        }
      }
    }
  }  
}

// 你的代码
for (const item of todos) {
  console.log(item)
}
```



## 遍历

```js
for // 数组
for in // 健值对

// 全新的方式 for of 遍历所有的数据结构的统一方式 数组，伪数组，set，map，对象
const map = new Map()
map.set('foo', '123')
for (const item of map) {
  item 是 ['foo', '123']
}
```

1. for of 可以用break终止循环，forEach不可以，遍历数组类的话每个item就是每个值
2. `for( const [key, value] of map)` 借助解构方便的拿到map的值

## Symbol

​	对象属性都是字符串，而字符串是有可能重复的。现如今我们大量使用第三方模块，很多时候需要去拓展第三方模块中提供的一些对象，那此时你是不知道这个对象中是否存在某个指定的健，如果直接去冒然的拓展，就有可能会产生冲突的问题。目前它就是为了为对象创建一个独一无二的标识符

```js
typeof s // symbol
Symbol() === Symbol() // false

obj[Symbol()] = '123'
```

​	另外我们还可以借助这个属性的特点模拟实现对象的私有成员问题，因为我们没有办法去创建一个完全相同的symbol，于是我们不能访问到这个成员

```js
const name = new Symbol()
const person = {
  [name]: 'tom',
  say() { this[name] }
}
person(new Symbol()) // undefined
```

​	目前一共7种数据类型，未来还会加一个BigInt数据类型

​	如果想要获取相同的Symbol怎么办，

```js
const s1 = Symbol.for('foo')
const s2 = Symbol.for('foo')
s1 === s2 // true
```

​	要注意的是如下方式都获取不到Symbol对象的属性

```js
for(var key in obj)
Object.keys(obj)
JSON.stringify(obj)

办法是 Object.getOwnPropertySymbols(obj)
```

# EcmaScript7

```js
// Object.values(obj)
返回对象的value，跟Object.keys很类似

// Object.entries(obj)
将对象转为数组形式 [['foo','1'], ['bar','1']]

for(const [key, value] of Object.entries(obj))
  
new Map(Object.entries(obj))

// padEnd padStart
name.padEnd(5, '-') => 'tom--'
name.padStart(5, '-') => '--tom'
```



# 存储

​	它是设计用来在服务器和客户端进行信息传递的，因此我们的每个 HTTP 请求都带着 cookie。但是 cookie 也具备浏览器端存储的能力（例如记住用户名和密码），因此就被开发者用上了。

​	使用起来也非常简单，`document.cookie = ....`即可。

但是 cookie 有它致命的缺点：

*   存储量太小，只有 4KB
*   所有 HTTP 请求都带着，会影响获取资源的效率
*   API 简单，需要封装才能用

​    后来，HTML5 标准就带来了`sessionStorage`和`localStorage`，先拿`localStorage`来说，它是专门为了浏览器端缓存而设计的。其优点有：

*   存储量增大到 5MB
*   不会带到 HTTP 请求中
*   API 适用于数据存储 `localStorage.setItem(key, value)` `localStorage.getItem(key)`

​    另外告诉大家一个小技巧，针对`localStorage.setItem`，使用时尽量加入到`try-catch`中，某些浏览器是禁用这个 API 的，要注意。

# Ajax

```js
var xhr = new XMLHttpRequest()
xhr.onreadystatechange = function () {
    // 这里的函数异步执行，可参考之前 JS 基础中的异步模块
    if (xhr.readyState == 4) {
        if (xhr.status == 200) {
            alert(xhr.responseText)
        }
    }
}
xhr.open("GET", "/api", false)
xhr.send(null)
```

​	`xhr.readyState`是浏览器判断请求过程中各个阶段的

*   0 -代理被创建，但尚未调用 `open()` 方法。
*   1 -`open()` 方法已经被调用。
*   2 -`send()` 方法已经被调用，并且头部和状态已经可获得。
*   3 -下载中， `responseText` 属性已经包含部分数据。
*   4 -下载操作已完成



​	`xhr.status`是 HTTP 协议中规定的不同结果的返回状态说明。

*   `200` 正常
*   `3xx`
    *   `301` 永久重定向。如`http://xxx.com`这个 GET 请求（最后没有`/`），就会被`301`到`http://xxx.com/`（最后是`/`）
    *   `302` 临时重定向。临时的，不是永久的
    *   `304` 资源找到但是不符合请求条件，不会返回任何主体。如发送 GET 请求时，head 中有`If-Modified-Since: xxx`（要求返回更新时间是`xxx`时间之后的资源），如果此时服务器 端资源未更新，则会返回`304`，即不符合要求
*   `404` 找不到资源
*   `5xx` 服务器端出错了

## 跨域

url 哪些地方不同算作跨域？

*   协议
*   域名
*   端口

​    但是 HTML 中几个标签能逃避过同源策略——`<script src="xxx">`、`<img src="xxxx"/>`、`<link href="xxxx">`，这三个标签的`src/href`可以加载其他域的资源，不受同源策略限制。

*   `<script>`和`<link>`可以使用 CDN，CDN 基本都是其他域的链接。
*   另外`<script>`还可以实现 JSONP，能获取其他域接口的信息，接下来马上讲解。

### JSONP

```jsx
<script>
window.callback = function (data) {
    // 这是我们跨域得到信息, 服务器可动态生成内容
    console.log(data)
}
</script>
```

### 服务器端设置 http header

```js
response.setHeader("Access-Control-Allow-Origin", "http://m.juejin.com/");  // 第二个参数填写允许跨域的域名称，不建议直接写 "*"
response.setHeader("Access-Control-Allow-Headers", "X-Requested-With");
response.setHeader("Access-Control-Allow-Methods", "PUT,POST,GET,DELETE,OPTIONS");

// 接收跨域的cookie
response.setHeader("Access-Control-Allow-Credentials", "true");
```



# 防抖节流

throttle（事件节流）和 debounce（事件防抖）

## throttle

​	节流的中心思想在于：在某段时间内，不管你触发了多少次回调，我都只认第一次，并在计时结束时给予响应。

​	所谓的“节流”，是通过在一段时间内无视后来产生的回调请求来实现的

​	每当用户触发了一次 scroll 事件，我们就为这个触发操作开启计时器。一段时间内，后续所有的 scroll 事件都会被当作“一辆车的乘客”——它们无法触发新的 scroll 回调。直到“一段时间”到了，第一次触发的 scroll 事件对应的回调才会执行，而“一段时间内”触发的后续的 scroll 回调都会被节流阀无视掉。

```js
// fn是我们需要包装的事件回调, interval是时间间隔的阈值
function throttle(fn, interval) {
  let last = 0
  return function () {
      let context = this
      let args = arguments
      let now = +new Date()
      if (now - last >= interval) {
          last = now;
          fn.apply(context, args);
      }
    }
}
// 用throttle来包装scroll的回调
const better_scroll = throttle(() => console.log('触发了滚动事件'), 1000)
document.addEventListener('scroll', better_scroll)
```



## Debounce

​	防抖的中心思想在于：我会等你到底。在某段时间内，不管你触发了多少次回调，我都只认最后一次。有人上车，我就再等10分钟，10分钟内没人上车就走

```js
// fn是我们需要包装的事件回调, delay是每次推迟执行的等待时间
function debounce(fn, delay) {
  let timer = null
  return function () {
    let context = this
    let args = arguments
    // 每次事件被触发时，都去清除之前的旧定时器
    if(timer) {
        clearTimeout(timer)
    }
    // 设立新定时器
    timer = setTimeout(function () {
      fn.apply(context, args)
    }, delay)
  }
}
// 用debounce来包装scroll的回调
const better_scroll = debounce(() => console.log('触发了滚动事件'), 1000)
document.addEventListener('scroll', better_scroll)
```





# BOM

​	BOM（浏览器对象模型）是浏览器本身的一些信息的设置和获取，例如获取浏览器的宽度、高度，设置让浏览器跳转到哪个地址。

*   `navigator`
*   `screen`
*   `location`
*   `history`

​    获取浏览器特性（即俗称的`UA`）然后识别客户端，例如判断是不是 Chrome 浏览器

```js
var ua = navigator.userAgent
var isChrome = ua.indexOf('Chrome')
```

​	获取屏幕的宽度和高度

```js
console.log(screen.width)
console.log(screen.height)
```

​	获取网址、协议、path、参数、hash 等

```js
// 例如当前网址是 https://juejin.im/timeline/frontend?a=10&b=10#some
console.log(location.href)  // https://juejin.im/timeline/frontend?a=10&b=10#some
console.log(location.protocol) // https:
console.log(location.pathname) // /timeline/frontend
console.log(location.search) // ?a=10&b=10
console.log(location.hash) // #some
```

​	另外，还有调用浏览器的前进、后退功能等

```js
history.back()
history.forward()
```

# DOM

​	我们开发完的 HTML 代码会保存到一个文档中（一般以`.html`或者`.htm`结尾），文档放在服务器上，浏览器请求服务器，这个文档被返回。因此，最终浏览器拿到的是一个文档而已，文档的内容就是 HTML 格式的代码。

​	但是浏览器要把这个文档中的 HTML 按照标准渲染成一个页面，此时浏览器就需要将这堆代码处理成自己能理解的东西，也得处理成 JS 能理解的东西，因为还得允许 JS 修改页面内容呢。

​	基于以上需求，浏览器就需要把 HTML 转变成 DOM，HTML 是一棵树，DOM 也是一棵树。对 DOM 的理解，可以暂时先抛开浏览器的内部因素，先从 JS 着手，即可以认为 DOM 就是 JS 能识别的 HTML 结构，一个普通的 JS 对象或者数组。

## 获取DOM节点

```js
// 通过 id 获取
var div1 = document.getElementById('div1') // 元素

// 通过 tagname 获取
var divList = document.getElementsByTagName('div')  // 集合
console.log(divList.length)
console.log(divList[0])

// 通过 class 获取
var containerList = document.getElementsByClassName('container') // 集合

// 通过 CSS 选择器获取
var pList = document.querySelectorAll('p') // 集合

// 获取父元素
var parent = div1.parentElement

// 获取子元素
var child = div1.childNodes
```



## 新增节点

```js
var p1 = document.createElement('p')
p1.innerHTML = 'this is p1'
div1.appendChild(p1) // 添加新创建的元素
```



## 移动节点

```js
// 移动已有节点。注意，这里是“移动”，并不是拷贝
var p2 = document.getElementById('p2')
div1.appendChild(p2)
```



## 删除节点

```js
var div1 = document.getElementById('div1')
var child = div1.childNodes
div1.removeChild(child[0])
```



## property 和 attribute

​	DOM 节点就是一个 JS 对象。`p`可以有`style`属性，有`className` `nodeName` `nodeType`属性。注意，这些都是 JS 范畴的属性，符合 JS 语法标准的。

​	property 的获取和修改，是直接改变 JS 对象，而 attribute 是直接改变 HTML 的属性，两种有很大的区别。attribute 就是对 HTML 属性的 get 和 set，和 DOM 节点的 JS 范畴的 property 没有关系。

​	而且，get 和 set attribute 时，还会触发 DOM 的查询或者重绘、重排，频繁操作会影响页面性能。

```js
var p = document.querySelectorAll('p')[0]

// property
p.style.width = '100px'  // 修改样式

// attribute
p.getAttribute('data-name')
p.setAttribute('data-name', 'juejin')
p.getAttribute('style')
p.setAttribute('style', 'font-size:30px;')
```



# 正则表达式

## 字面量创建

```js
let str = "dh73287d372ydh3hd"
let reg = /\d+/g // #1
let res = str.match(reg)
console.log(res) // ['7328', '372', '3']
```

1. \d表示数字，+表示一到多个，g表示修饰符，是全局，即找到匹配项以后会继续去查找
2. \d这种其实是模糊匹配，此外还有精确匹配，如 /abc/



## 构造函数创建

```js
let reg = new RegExp("\\d+", "g") // \转义
let res = str.match(reg)
console.log(res)
```



## 方法

### test

​	返回布尔值，判断是否匹配



### exec

​	每一次都会基于上一次的结果进行匹配

```js
let str = "333rrr33rr"
let reg = /\d+/g
reg.exec(str) // 333
reg.exec(str) // 33
```



​	此外也可以用在字符串上，如

```js
str.replace(reg, "*")
str.split(/d+/) // 分割字符串
```



## 元字符

### 字符相关

- \w：数字，字母，下划线 --- \W：非数字，字母，下划线

- \d：数字 --- \D：非数字

- \s：匹配空格

- .（点）：除\n   \r（回车）   \u2028（段落结束符）   \u2029（行结束符）



### 数量相关

​	`{}   ?   +   *`

```js
/aeeea/ === /ae{3}a/

/ae{1,4}a/  // 1到4个e 闭区间
  
/ae{1,}/  // 1到无限个e

? === {0,1}
+ === {1,}
* === {0,}
```



### 位置相关

`^   $   \b   \B`

- ^：第一个
- $：最后一个
- \b：边界符

```js
let reg = /^\w/g    // 第一个满足数字，字母，下划线的

this is an apple
你会发现有两个is   第二个is是有边界的，就是空格
/\bis\b/g
那什么是边界呢，非\w的都是边界
```



### 括号相关

`()   [ ]  { }`

​	表示分组和集合（`{}是？`）

```js
let str = "abababc"

/ababab/;   // 可以匹配到但是有点麻烦
/ab{3}/;    //  发现不行，反而会匹配3个b
/(ab){3}/;    //  这样就可以了 

此外还可以提取值
let str = "2020-02-02"
let reg = /(\d{4})-(\d{2})-(\d{2})/
RegExp.$1 // $2, $3通过一个全局属性拿到

此外还可以替换
let str = "2020-02-02"
let reg = /(\d{4})-(\d{2})-(\d{2})/
let res = str.replace(reg, "$2/$3/$1")  //  02/02/2020
let res = str.replace(reg, function(arg, year, mouth, day) {
  return mouth + "/" + day + "/" + year
})


let className = "news-container-nav"
let reg = /\w{4}(-|_)\w{9}(\1)\w{3}/  
// (\1)就是表示跟第一个（-|_）相同，前面是什么样后面也是什么样，就是
// 想要引用之前匹配到过的字符的意思，这种方式叫做反向引用
```



## 匹配模式

`g   i   m   s   u   y`

g：全局匹配

i：忽略大小写

m：多行，比如用在模板字符串上

s：可以让 . 也可以匹配换行

u：可以匹配unicode编码









**⚠️ 数组的sort方法**

```js
let arr = [1, 2, 3]
arr.sort((n1, n2) => { // 2,1 3,2
  return -1 // 返回负值交换数组顺序 [3, 2, 1]
  return 1 // 返回正值不交换 [1, 2, 3]
})
```

所以从小到大一般`n1 - n2`，n1 < n2 返回负值交换顺序



## 题目

---

```js
var a = function yideng(num) {
  yideng = num
  console.log(typeof yideng)
  return 1
}
a(1)
console.log(typeof yideng())
```

- yideng是只读的，且只在内部使用。在node中这样使用其实一般就是为了调错的

```js
function
yideng is not defined  
```



---

```js
this.a = 20
var test = {
  a: 40,
  init: function() {
    function go() {
      console.log(this.a)
    }
    go()
  }
}
test.init() // 20
```

- `test.init()` 是 `go()` `go`没人调用, 所以是`window`
- ⚠️ 总结：只要函数执行前面没有点，你就给它加个window



---

```js
this.a = 20
var test = {
  a: 40,
  init: () => {
    console.log(this.a)
  }
}
test.init() // 20
var fn = test.init()
fn() // 20
```

- 箭头函数绑定父级作用域，跟写bind的意思一模一样
- ⚠️ 总结：使用箭头函数就不要管点了





---

new

```js
function Foo() {
    getName = function() {
        console.log(1);
    };
    return this;
}
Foo.getName = function() {
    console.log(2);
};
Foo.prototype.getName = function() {
    console.log(3);
};
var getName = function() {
    console.log(4);
};
function getName() {
    console.log(5);
}
Foo.getName();	// 2
getName();	// 4
Foo().getName();	// 1
getName();	// 1
new Foo.getName(); // 2
new Foo().getName(); // 3
new new Foo().getName(); // 3
```

1. Foo.getName自然是访问构造函数的静态属性，2

   ```js
   function User() {
     this.name = 'a' // 公有属性
     var name = 'a' // 私有属性
     function getName() {} // 私有方法
     name = 'a' // 变为全局变量
   }
   User.name = 'a' // 静态属性
   ```

2. 函数提升，变量提升，4

3. Foo内部的getname没有用var，成为全局方法（执行Foo才这样），Foo没有new，所以this是window，1

4. 看3，1

5. `x = Foo.getName; new x()`

6. `x = new Foo(); x.getName()`

7. `x = new Foo(); y = x.getName;new y()`











# ---

