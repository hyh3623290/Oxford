[toc]

# 通识

Proxy SwitchyOmega



# 变量类型

​	JavaScript 是一种弱类型脚本语言，所谓弱类型指的是定义变量时，不需要什么类型，在程序运行过程中会自动判断类型。

ECMAScript 中定义了 8 种数据类型：

*   Boolean
*   String
*   Number
*   Null
*   Undefined
*   Symbol（ES6 新定义）
*   Object



## 判断变量类型

### typeof

​	比较简单，需要注意的地方：null返回了object



### instanceof

​	用于实例和构造函数的对应。

```js
function Foo(name) {
    this.name = name
}
var foo = new Foo('bar')
console.log(foo instanceof Foo) // true

let str = 'Covid-19'
str instanceof String // false

// instanceof 可以准确地判断复杂引用数据类型，但是不能正确判断基础数据类型；
```



**手写instanceof**

```js
function myInstanceof(left, right) {
  if(typeof left!=='object' || left === null) return false
  let proto = Object.getPrototypeOf(left)
  while(true) {
    if(proto === null) return false
    if(proto === right.prototype) return true
    proto = Object.getPrototypeOf(proto)
  }
}

console.log(myInstanceof(new Number(123), Number));    // true
console.log(myInstanceof(123, Number));                // false
```



### Object.prototype.toString

​	统一返回格式为 `[object Xxx]` 的字符串。

​	对于 Object 对象，直接调用 `toString()` 就能返回 [object Object]；而对于其他对象，则需要通过 call 来调用

```js
Object.prototype.toString({})       // "[object Object]"
Object.prototype.toString.call({})  // 同上结果，加上call也ok
Object.prototype.toString.call(1)    // "[object Number]"
Object.prototype.toString.call('1')  // "[object String]"
Object.prototype.toString.call(true)  // "[object Boolean]"
Object.prototype.toString.call(function(){})  // "[object Function]"
Object.prototype.toString.call(null)   //"[object Null]"
Object.prototype.toString.call(undefined) //"[object Undefined]"
Object.prototype.toString.call(/123/g)    //"[object RegExp]"
Object.prototype.toString.call(new Date()) //"[object Date]"
Object.prototype.toString.call([])       //"[object Array]"
Object.prototype.toString.call(document)  //"[object HTMLDocument]"
Object.prototype.toString.call(window)   //"[object Window]"
```

​	从上面这段代码可以看出，`Object.prototype.toString.call()` 可以很好地判断引用类型，甚至可以把 document 和 window 都区分开来。

### 封装getType

```js
function getType(obj){
  let type  = typeof obj;
  if (type !== "object") {
    return type;
  }
  return Object.prototype.toString.call(obj).replace(/^\[object (\S+)\]$/, '$1');  
  // 注意正则中间有个空格
}

getType([])     // "Array" typeof []是object，因此toString返回
getType('123')  // "string" typeof 直接返回
getType(window) // "Window" toString返回
getType(null)   // "Null"首字母大写，typeof null是object，需toString来判断
getType(undefined)   // "undefined" typeof 直接返回
getType()            // "undefined" typeof 直接返回
getType(function(){}) // "function" typeof能判断，因此首字母小写
getType(/123/g)      //"RegExp" toString返回
```





## 值类型 vs 引用类型

​	值类型是按值传递，引用类型是按共享传递。（function也属于引用类型）

​	假设a`和`b`都是引用类型，指向了同一个内存地址，即两者引用的是同一个值，因此`b`修改属性时，`a`的值随之改动。

​	JS 中这种设计的原因是：按值传递的类型，复制一份存入栈内存，这类类型一般不占用太多内存，而且按值传递保证了其访问速度。按共享传递的类型，是复制其引用，而不是整个复制其值（C 语言中的指针），保证过大的对象等不会因为不停复制内容而造成内存的浪费。



## 垃圾回收机制

​	每隔一段时间，JS 的垃圾收集器就会对变量做 “巡检”。当它判断一个变量不再被需要之后，它就会把这个变量所占用的内存空间给释放掉，这个过程叫做垃圾回收。

### 引用计数法

​	最初级，已淘汰。当我们用一个变量指向了一个值，那么就创建了一个针对这个值的 “引用”。

​	在引用计数法的机制下，内存中的每一个值都会对应一个引用计数。当垃圾收集器感知到某个值的引用计数为 0 时，就判断它 “没用” 了，随即这块内存就会被释放。

​	下面这行代码首先是开辟了一块内存，把右侧这个数组塞了进去，此时这个数组就占据了一块内存。随后 students 变量指向它，这就是创建了一个指向该数组的 “引用”。

```js
const students = ['修言', '小明', 'bear']
```

​	如果把 students 指向一个 null ：

```js
students = null
```

​	那么 [‘xiuyan’, ‘xiaoming’, ‘bear’] 这个数组所具备的引用计数就会跟着变成 0

缺点：无法甄别循环引用



### 标记清除法

​	在标记清除算法中，一个变量是否被需要的判断标准，是它是否可抵达 。

这个算法有两个阶段，分别是标记阶段和清除阶段：

- 标记阶段：垃圾收集器会先找到根对象。从根对象出发，垃圾收集器会扫描所有可以通过根对象触及的变量，这些对象会被标记为 “可抵达”。
- 清除阶段： 没有被标记为 “可抵达” 的变量，就会被认为是不需要的变量，这波变量会被清除

​    比如一个函数执行完，里面的变量就被确定为不可达。

# 原型和原型链

​	JavaScript 是基于原型的语言，原型理解起来非常简单，但却特别重要，下面还是通过题目来理解下JavaScript 的原型概念。

*   所有的引用类型（数组、对象、函数），都具有对象特性，即可自由扩展属性（`null`除外）
*   所有的引用类型（数组、对象、函数），都有一个`__proto__`属性，属性值是一个普通的对象
*   所有的函数，都有一个`prototype`属性，属性值也是一个普通的对象
*   所有的引用类型（数组、对象、函数），`__proto__`属性值指向它的构造函数的`prototype`属性值

```js
// 总结一下
函数 - prototype
引用类型 - __proto__
引用类型的__proto__ === 它的构造函数的prototype

但是其实函数，prorotype也是引用类型
```



​	当试图得到一个对象的某个属性时，如果这个对象本身没有这个属性，那么会去它的`__proto__`（即它的构造函数的`prototype`）中寻找。这也就是所谓的**原型链**

​	`f.__proto__`中没有找到`toString`，那么就继续去`f.__proto__.__proto__`中寻找。

​	最上层是什么 —— `Object.prototype.__proto__ === null`

​	那么如何判断这个属性是不是对象本身的属性呢？使用`hasOwnProperty`，常用的地方是遍历一个对象的时候。

```js
for (let item in f) {
    // 高级浏览器已经在 for in 中屏蔽了来自原型的属性，但是这里建议大家还是加上这个判断，保证程序的健壮性
    if (f.hasOwnProperty(item)) {
        console.log(item)
    }
}
```



​	原型链中的this和普通的this没有区别

​	在 JavaScript 中，每个构造函数都拥有一个 `prototype` 属性，它指向构造函数的原型对象，这个原型对象中有一个 construtor 属性指回构造函数；每个实例都有一个`__proto__`属性，当我们使用构造函数去创建实例时，实例的`__proto__`属性就会指向构造函数的原型对象。

## new

​	new做了什么

- 为这个新的对象开辟一块属于它的内存空间
- 把函数体内的 this 指到 1 中开辟的内存空间去
- 将新对象的`__proto__ `这个属性指向对应构造函数的 prototype 属性，把实例和原型对象关联起来
- 执行函数体内的逻辑，最后即便你没有手动 return，构造函数也会帮你把创建的这个新对象 return 出来

# 作用域和闭包

- 词法作用域： 在代码书写的时候完成划分，作用域链沿着它**定义的位置**往外延伸
- 动态作用域： 在代码运行时完成划分，作用域链沿着它的**调用栈**往外延伸

**如何修改词法作用域**

```js
function showName(str) {
  eval(str)
  console.log(name)
}

var name = 'xiuyan'
var str = 'var name = "BigBear"'

showName(str) // 输出 BigBear
```



​	如下，点击编号为几的链接就`alert`弹出其编号

```jsx
<ul>
    <li>编号1，点击我请弹出1</li>
    <li>2</li>
    <li>3</li>
    <li>4</li>
    <li>5</li>
</ul>
```

​	需要闭包解决

```js
var list = document.getElementsByTagName('li');
for (var i = 0; i < list.length; i++) {
    list[i].addEventListener('click', function(i){
        return function(){
            alert(i + 1)
        }
    }(i), true)
}
```

​	大家想想，这个函数第一次被执行，也是 1000ms 以后的事情了（另一种setTimeout的例子），此时它（`i变量`）试图向外、向上层作用域（这里就是全局作用域）去找一个叫 i 的变量。此时 for 循环早已执行完毕， i 也进入了尘埃落定的最终状态

其他解决方式

- setTimeout 函数的第三个参数会作为回调函数的附加参数
- let



闭包面试题，先画图，从最开始执行的函数开始画出作用域的嵌套关系



## 执行上下文

​	执行上下文，从定义上理解，是 “执行代码的环境” -- 分为全局上下文和函数上下文。每一个执行上下文都会经历这样一个生命周期

- 创建阶段 —— 执行上下文的初始化状态，此时一行代码都还没有执行，只是做了一些准备工作
- 执行阶段 —— 逐行执行脚本里的代码

### 变量提升

​	在一段 JS 脚本（即一个`<script>`标签中）执行之前，要先解析代码（所以说 JS 是解释执行的脚本语言），解析的时候会先创建一个 **全局执行上下文** 环境，先把代码中即将执行的（内部函数的不算，因为你不知道函数何时执行）变量、函数声明都拿出来。变量先暂时赋值为`undefined`，函数则先声明好可使用。这一步做完了，然后再开始正式执行程序。再次强调，这是在代码执行之前才开始的工作。

​		所谓的 “变量提升”，只是变量的创建过程（在上下文创建阶段完成）和真实赋值过程（在上下文执行阶段完成）的不同步带来的一种错觉。

总结一下：

*   范围：一段`<script>`、js 文件或者一个函数
*   全局上下文：变量定义，函数声明
*   函数上下文：变量定义，函数声明，`this`，`arguments`

​    函数提升优先于变量提升，函数提升会把整个函数挪到作用域顶部，变量提升只会把声明挪到作用域顶部

### 调用栈

​	我们看到函数执行完毕后，其对应的执行上下文也随之消失了。这个消失的过程，我们叫它” 出栈 “—— 没错，在 JS 代码的执行过程中，引擎会为我们创建” 执行上下文栈 “（也叫调用栈）。



## this

### 指向问题

​	先搞明白一个很重要的概念 —— `this`的值是在执行的时候才能确认，定义的时候不能确认！

`this`执行会有不同，主要集中在这几个场景中

*   作为构造函数执行，构造函数中
*   作为对象属性执行，上述代码中`a.fn()`
*   作为普通函数执行，上述代码中`fn1()`
*   用于`call` `apply` `bind`，上述代码中`a.fn.call({name: 'B'})`



- 对于直接调用 `foo` 来说，不管 `foo` 函数被放在了什么地方，`this` 一定是 `window`
- 对于 `obj.foo()` 来说，我们只需要记住，谁调用了函数，谁就是 `this`，所以在这个场景下 `foo` 函数中的 `this` 就是 `obj` 对象
- 对于 `new` 的方式来说，`this` 被永远绑定在了 `c` 上面，不会被任何方式改变 `this`
- 箭头函数中的 `this` 只取决包裹箭头函数的第一个普通函数的 `this`

​    所有的方法都是需要寄宿在一个宿主也就是对象上，当然方法也是对象。（自己理解）

​	谁调的函数，this 就归谁

```js
// 声明位置
var me = {
  name: 'xiuyan',
  hello: function() {
    console.log(`你好，我是${this.name}`)
  }
}

var you = {
  name: 'xiaoming',
  hello: function() {
    var targetFunc = me.hello
    targetFunc()
  }
}

var name = 'BigBear'

// 调用位置
you.hello()
```

​	输出BigBear

**技巧**

三种特殊情境下，this 会 100% 指向 window

- 立即执行函数（IIFE）
- setTimeout 中传入的函数
- setInterval 中传入的函数



### 手写call、apply 及 bind 函数

```js
// fn.myCall(obj, 123)
Function.prototype.myCall = function(context, ...args) {
  if (typeof this !== 'function') { // this -> fn
    throw new TypeError('Error')
  }
  context = context || window
  context.fn = this
  const result = context.fn(...args)
  delete context.fn
  return result
}
```

- call 是可以被所有的函数继承的，所以 call 方法应该被定义在 Function.prototype 上
- 因为 `call` 可以传入多个参数作为调用函数的参数，所以需要将参数剥离出来
- 然后调用函数并将对象上的函数删除，指向改变指向，不想给对象新增一个方法
- 函数如果是有返回值的话怎么办？是不是新开一个 result 变量存储一下这个值，最后 return 出来就可以了

 

```js
// fn.myApply(obj, [1, 2, 3])
Function.prototype.myApply = function(context, arr) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  context = context || window
  context.fn = this
  let result
  // 处理参数和 call 有区别
  if (arr) {
    result = context.fn(arr)
  } else {
    result = context.fn()
  }
  delete context.fn
  return result
}
```

​	

​	`bind` 的实现对比其他两个函数略微地复杂了一点，因为 `bind` 需要返回一个函数，需要判断一些边界问题，以下是 `bind` 的实现

```js
fn.bind().bind(a)() // window 只跟第一次绑定的对象有关系
```





## 作用域

​	变量、函数、标识符可以被访问的区域。作用域本质是一套规则，访问，获取变量的规则

​	作用域就是一个独立的地盘，让变量不会外泄、暴露出去。JS 没有块级作用域，只有全局作用域和函数作用域。全局作用域的坏处就是很容易撞车、冲突，这就是为何 jQuery、Zepto 等库的源码，所有的代码都会放在`(function(){....})()`中。

​	如果当前作用域没有定义的变量，就会向父级作用域寻找 -- 作用域链

​	

## 闭包

​	引用了自由变量（不属于当前作用域的变量）的函数，就叫闭包。

​    闭包的定义其实很简单：函数 A 内部有一个函数 B，函数 B 可以访问到函数 A 中的变量，那么函数 B 就是闭包。

```js
// 函数作为返回值
function F1() {
  var a = 100
  return function () {
    console.log(a)
  }
}
var f1 = F1()
var a = 200
f1()

// 函数作为参数传递
function F2(f1) {
  var a = 200
  console.log(f1())
}
var f1 = F1()
F2(f1)

```

​	自由变量将从作用域链中去寻找，但是依据的是函数定义时的作用域链，而不是函数执行时，以上这个例子就是闭包。闭包主要有两个应用场景：

*   函数作为返回值
*   函数作为参数传递

### 模拟私有变量的实现

​	比如有个User类，通过`this.password`存了密码，那么可以很容易的通过实例`user.password`获取，甚至修改密码，这个是很可怕的，所以我们需要想办法从代码层面去保护 password

​	像 password 这样的变量，我们希望它仅在对象内部生效，无法从外部触及，这样的变量，就是私有变量。

​	我们的思路就是把私有变量用函数作用域来保护起来，形成一个闭包

```js
const User = (function() {
  // 定义私有变量_password
  let _password
  class User {
    constructor (username, password) {
      // 初始化私有变量_password
      _password = password
      this.username = username
    }

    login() {
      // 这里我们增加一行 console，为了验证 login 里仍可以顺利拿到密码
      console.log(this.username, _password)
    }
  }
  return User
})()
```



### 偏函数和柯里化

​	柯里化是把**接受 n 个参数的 1 个函数**改造为**只接受 1个参数的 n 个互相嵌套的函数**的过程。也就是` fn (a, b, c) `会变成` fn (a)(b)(c)`

​	为啥改造，假设一个商品，我们考虑使用 prefix（一个标识不同站点的前缀字符串）、 type（商品类型）、name（商品原本名称）三个字符串拼接的方式来为商品生成一个完整版名称。对应的方法如下：

```js
function generateName(prefix, type, itemName) {
  return prefix + type + itemName
}
```

​	再假设我只负责大卖网，“生鲜” 类商品，传两个固定参数没必要

```js
generateName('大卖网', '生鲜', itemName)
```

​	此时即可改造

```js
function generateName(prefix) {  
 return function(type) {
   return function (itemName) {
     return prefix + type + itemName
   }    
 }
}
// 生成大卖网商品名专属函数
var salesName = generateName('大卖网')
// “记住”prefix，生成大卖网生鲜商品名专属函数
var salesBabyName = salesName('生鲜')
// 啥也不记，直接生成一个商品名
var itemFullName = generateName('洗菜网')('生鲜')('菠菜')
```

​	通过后者这种形式，我们可以选择性地决定是否要 “记住” prefix、type，从而即时地生成更加符合我们预期的、复用程度更高的目标函数。此外，柯里化还可以帮助我们以嵌套的形式把多个函数的能力组合到一起，这就是柯里化的魅力。

**偏函数**

​	柯里化是将一个 n 个参数的函数转换成 n 个单参数函数。你有 10 个入参，就得嵌套 10 层函数，且每层函数都只能有 1 个入参。它的目标就是把函数的入参拆解为精准的 n 部分。

​	偏函数应用相比之下就 “随意” 一些了。偏函数是说，固定你函数的某一个或几个参数，然后返回一个新的函数（这个函数用于接收剩下的参数）。你有 10 个入参，你可以只固定 2 个入参，然后返回一个需要 8 个入参的函数 —— 偏函数应用是不强调 “单参数” 这个概念的。它的目标仅仅是把函数的入参拆解为两部分。

​	总结就是柯里化传参是固定模式，偏函数不固定



# 异步

​	JS 需要异步的根本原因是JS 是单线程运行的，即在同一时间只能做一件事，不能“一心二用”。

```js
var a = true;
setTimeout(function(){
    a = false;
}, 100)
while(a){
    console.log('while执行了')
}
// 进入while循环之后，没有「时间」（线程）去跑定时器了，所以这个代码跑起来是个死循环！
```

前端异步的场景

*   定时 `setTimeout` `setInterval`
*   网络请求，如 `Ajax` `<img>`加载

```js
console.log('start')
var img = document.createElement('img')
// 或者 img = new Image()
img.onload = function () {
    console.log('loaded')s
    img.onload = null
}
img.src = '/xxx.png'
console.log('end')
```



## Generator

​	最大的特点就是可以控制函数的执行。

```js
function *foo(x) {
  let y = 2 * (yield (x + 1))
  let z = yield (y / 3)
  return (x + y + z)
}
let it = foo(5)
console.log(it.next())   // => {value: 6, done: false}
console.log(it.next(12)) // => {value: 8, done: false}
console.log(it.next(13)) // => {value: 42, done: true}
```

- 首先 `Generator` 函数调用和普通函数不同，它会返回一个迭代器
- 当执行第一次 `next` 时，传参会被忽略，并且函数暂停在 `yield (x + 1)` 处，所以返回 `5 + 1 = 6`
- 当执行第二次 `next` 时，传入的参数等于上一个 `yield` 的返回值，如果你不传参，`yield` 永远返回 `undefined`。此时 `let y = 2 * 12`，所以第二个 `yield` 等于 `2 * 12 / 3 = 8`
- 当执行第三次 `next` 时，传入的参数会传递给 `z`，所以 `z = 13, x = 5, y = 24`，相加等于 `42`



## Promise

​	Promise 对象是一个代理对象。它接受你传入的 executor（执行器）作为入参，允许你把异步任务的成功和失败分别绑定到对应的处理方法上去。一个 Promise 实例有三种状态。

​	Promise实例的状态是可以改变的，但它只允许被改变一次。

```js
Promise.resolve(1)
  .then(Promise.resolve(2))
  .then(3)
  .then()
  .then(console.log)
// 1
```

​	promise嵌套使用是其常见的误区，这样写依旧会陷入回调地狱，还不如回调的方式，所以应使用链式

```js
// 嵌套
promise.then(res => {
  promise2.then(res => {
    ...
  })
})
  
// 链式
promise
  .then(res => {
  	...
	})
  .then(res => {
  	...
	})
  .catch(err => { 
  	... 
	})


// catch相当于如下
.then(undefined, err => {
  ...
})
和在then里放第二个函数效果是一样的
不过catch可以捕获到它之前的then里异常，而then里的第二个参数不能捕获到第一个参数的异常
因为错误会一直向后传递的，直到被捕获
```

- 每个`then`即是一个全新的`promise .then() === new Promise()`
- `then`只要返回了（即使是返回`new Error`），就相当于`resolved`，会传给第二个`then`，没有返回即是默认`undefined`。除非报错，如`throw new Error`，就会走后面的then的第二个参数。即相当于reject

🍇 静态方法

​	`Promise.resolve()`快速把一个值转换为promise对象。如果原来就是promise，则会原样返回，可以试一下两个用三等符号，返回的就是true。同理有`Promise.reject()`

```js
Promise.resolve({
  then: (resolved, reject) => {
    resolved(1)
  }
})
.then(v => {
  console(v)
})
// 这样实现一个then也是可以的
// 俗称实现thenable接口
```

🍇 并行

```js
// all
Promise.all([
  ajax1('url'),
  ajax2('url')
])
.then()

// race 同理
// 案例：ajax请求超时异常的控制
var request = ajax('url')
var timeout = new Promise(() => { setTimeout })

Promise.race([request, timeout])

```











### 值穿透问题

​	但是无论如何，then 方法的入参只能是函数，我们最初 resolve 出来那个值，穿越了一个又一个无效的 then 调用，就好像是这些 then 调用都是透明的、不存在的一样



### catch与then里的第二个参数

```js
.then(onFulfilled, onRejected)
.catch(e => { })
// catch捕获错误的范围更大，也包括onFulfilled里错误
// onRejected只能捕获到promise里面的回调的错误
```



### Promise 规范

​	只要符合规范的对象和函数都可以成为promise（`onFulfilled -> 1, onRejected -> 2`）

​	then方法必须返回一个promsie对象，`promise2 = promise1.then(onFulfilled, onRejected)`

- 只要1或2返回一个值x，promise2就会返回onFulfilled
- 只要1或2抛出一个异常，promise2就会是拒绝状态
- 如果1或2不是函数那就是值穿透情况，就会以相同的状态返回给promise2



### 实现promise

```js
static resolve(value) {
  if(value instanceof Promise) {
    return value
  }
  return new Promise(resolve => resolve(value))
}

static reject() {}

static all(list) {
  return new Promise((resolve, reject) => {
    let count = 0
    const values = []
    for(const [i, promiseInstance] of list.entries()) {
      // Promise.resolve(promiseInstance)
      promiseInstance.then(res => {
        values[i] = res
        count++
        if(count === list.length) resolve(values)
      }, err => {
        reject(err)
      })
    }
  })
}

static race() {} 
```

​	执行triggerResolve后要将then里的第一个回调（onFulfilled参数）拿出来执行，then是同步的，所以我们在triggerResolve里面用setTimeout就能拿到onFulfilled

```js
const p1 = new Promise((resolve, reject) => {
  resolve(100)
  // reject(100)
})

p1
.then(res => {
  return '1'
}, err => {

})
.then(r => {

})                      
```





## async await

```js
let a = 0
let b = async () => {
  a = a + await 10
  console.log('2', a) // -> '2' 10
}
b()
a++
console.log('1', a) // -> '1' 1
```

- 首先函数 `b` 先执行，在执行到 `await 10` 之前变量 `a` 还是 0，因为 `await` 内部实现了 `generator` ，`generator` 会保留堆栈中东西，所以这时候 `a = 0` 被保存了下来
- 因为 `await` 是异步操作，后来的表达式不返回 `Promise` 的话，就会包装成 `Promise.reslove(返回值)`，然后会去执行函数外的同步代码
- 同步代码执行完毕后开始执行异步代码，将保存下来的值拿出来使用，这时候 `a = 0 + 10`

​    async/await 和 generator 方案，相较于 Promise 而言，有一个重要的优势：Promise 的错误需要通过回调函数捕获，try catch 是行不通的。而 async/await 和 generator 允许 try/catch



## EventLoop

### 执行栈

​	可以把执行栈认为是一个存储函数调用的**栈结构**，遵循先进后出的原则。当开始执行 JS 代码时，首先会执行一个 `main` 函数，然后执行我们的代码。根据先进后出的原则，后执行的函数会先弹出栈

### 浏览器中的 Event Loop

​	当遇到异步的代码时，会被**挂起**并在需要执行的时候加入到 Task（有多种 Task） 队列中。一旦执行栈为空，Event Loop 就会从 Task 队列中拿出需要执行的代码并放入执行栈中执行，所以本质上来说 JS 中的异步还是同步行为。

​	不同的任务源会被分配到不同的 Task 队列中，任务源可以分为 **微任务**（microtask） 和 **宏任务**（macrotask）。



# 深浅拷贝

## 浅拷贝

浅拷贝只解决了第一层的问题

1. Object.assign
2. 展开运算符 `...`



## 深拷贝

1. `JSON.parse(JSON.stringify(object))`
   - 会忽略 `undefined`
   - 会忽略 `symbol`
   - 不能序列化函数（忽略）
   - 不能解决循环引用的对象
2. lodash深拷贝函数
3. 手写深拷贝（简易）

```js
function deepClone(obj) {
  function isObject(o) {
    return (typeof o === 'object' || typeof o === 'function') && o !== null
  }

  if (!isObject(obj)) {
    throw new Error('非对象')
  }

  let isArray = Array.isArray(obj)
  let newObj = isArray ? [...obj] : { ...obj }
  Reflect.ownKeys(newObj).forEach(key => {
    newObj[key] = isObject(obj[key]) ? deepClone(obj[key]) : obj[key]
  })

  return newObj
}
```

# 事件

## 事件绑定

```js
var btn = document.getElementById('btn1')
btn.addEventListener('click', function (event) {
    // event.preventDefault() // 阻止默认行为
    // event.stopPropagation() // 阻止冒泡
    console.log('clicked')
})
```



## 事件冒泡

​	事件会沿着 DOM 树向上级节点冒泡，如果上级节点绑定了事件则会触发



## 事件代理

```html
<div id="div1">
    <a href="#">a1</a>
    <a href="#">a2</a>
    <a href="#">a3</a>
    <a href="#">a4</a>
</div>
<button>点击增加一个 a 标签</button>
```

​	我们想快捷方便地为所有`<a>`绑定事件，就可以用到事件代理。我们要监听`<a>`的事件，但要把具体的事件绑定到`<div>`上，然后看事件的触发点是不是`<a>`。因为当点击`<a>`以后，会冒泡到父级触发父级的事件

```js
var div1 = document.getElementById('div1')
div1.addEventListener('click', function (e) {
    // e.target 可以监听到触发点击事件的元素是哪一个
    var target = e.target
    if (e.nodeName === 'A') {
        // 点击的是 <a> 元素
        alert(target.innerHTML)
    }
})
```



## 事件流

完整的事件流分三阶段：捕获，目标阶段，冒泡

事件通过捕获到达目标元素，这个时候就是目标阶段，从目标元素上传到window对象就是冒泡阶段

























# 参考文献

- 掘金 - web前端面试指南与高频考题解析
- 幕客 - 解锁前端面试体系核心攻略 - 修言
- 掘金 - 前端面试之道

