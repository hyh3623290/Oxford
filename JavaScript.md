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
```



## 值类型 vs 引用类型

​	值类型是按值传递，引用类型是按共享传递。（function也属于引用类型）

​	假设a`和`b`都是引用类型，指向了同一个内存地址，即两者引用的是同一个值，因此`b`修改属性时，`a`的值随之改动。

​	JS 中这种设计的原因是：按值传递的类型，复制一份存入栈内存，这类类型一般不占用太多内存，而且按值传递保证了其访问速度。按共享传递的类型，是复制其引用，而不是整个复制其值（C 语言中的指针），保证过大的对象等不会因为不停复制内容而造成内存的浪费。



## 垃圾回收机制

#### 引用计数法

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



#### 标记清除法

​	在标记清除算法中，一个变量是否被需要的判断标准，是它是否可抵达 。

这个算法有两个阶段，分别是标记阶段和清除阶段：

- 标记阶段：垃圾收集器会先找到根对象。从根对象出发，垃圾收集器会扫描所有可以通过根对象触及的变量，这些对象会被标记为 “可抵达”。
- 清除阶段： 没有被标记为 “可抵达” 的变量，就会被认为是不需要的变量，这波变量会被清除

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

# 作用域和闭包

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

### 变量提升

​	在一段 JS 脚本（即一个`<script>`标签中）执行之前，要先解析代码（所以说 JS 是解释执行的脚本语言），解析的时候会先创建一个 **全局执行上下文** 环境，先把代码中即将执行的（内部函数的不算，因为你不知道函数何时执行）变量、函数声明都拿出来。变量先暂时赋值为`undefined`，函数则先声明好可使用。这一步做完了，然后再开始正式执行程序。再次强调，这是在代码执行之前才开始的工作。

总结一下：

*   范围：一段`<script>`、js 文件或者一个函数
*   全局上下文：变量定义，函数声明
*   函数上下文：变量定义，函数声明，`this`，`arguments`

​    函数提升优先于变量提升，函数提升会把整个函数挪到作用域顶部，变量提升只会把声明挪到作用域顶部



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

**Promise 值穿透问题**

​	但是无论如何，then 方法的入参只能是函数，我们最初 resolve 出来那个值，穿越了一个又一个无效的 then 调用，就好像是这些 then 调用都是透明的、不存在的一样





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



# 事件代理

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



# 防抖节流

throttle（事件节流）和 debounce（事件防抖）

## throttle

​	节流的中心思想在于：在某段时间内，不管你触发了多少次回调，我都只认第一次，并在计时结束时给予响应。

​	所谓的“节流”，是通过在一段时间内无视后来产生的回调请求来实现的

​	每当用户触发了一次 scroll 事件，我们就为这个触发操作开启计时器。一段时间内，后续所有的 scroll 事件都会被当作“一辆车的乘客”——它们无法触发新的 scroll 回调。直到“一段时间”到了，第一次触发的 scroll 事件对应的回调才会执行，而“一段时间内”触发的后续的 scroll 回调都会被节流阀无视掉。

```js
// fn是我们需要包装的事件回调, interval是时间间隔的阈值
function throttle(fn, interval) {
  // last为上一次触发回调的时间
  let last = 0
  
  return function () {
      // 保留调用时的this上下文
      let context = this
      let args = arguments
      // 记录本次触发回调的时间
      let now = +new Date()
      
      // 判断上次触发的时间和本次触发的时间差是否小于时间间隔的阈值
      if (now - last >= interval) {
      // 如果时间间隔大于我们设定的时间间隔阈值，则执行回调
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

​	防抖的中心思想在于：我会等你到底。在某段时间内，不管你触发了多少次回调，我都只认最后一次。

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

























# 参考文献

- 掘金 - web前端面试指南与高频考题解析
- 幕客 - 解锁前端面试体系核心攻略 - 修言
- 掘金 - 前端面试之道

