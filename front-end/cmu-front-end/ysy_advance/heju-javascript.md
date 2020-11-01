# 数据类型

Boolean

String

Null

Undefined

Number

Symbol

Object

# 对象新增

```js
Object.defineProperty(cat, "name", {
  value: "123", //能否使用delete、能否需改属性特性、或能否修改访问器属性、，false为不可重新定义，默认值为true
  writable: false, // name不能被改变
  enumrable: true, // 可以出现在for-in循环里
  configurable: false
})
```

```js
Object.keys(obj) // key会被放进数组里
```

```js
var p = Object.create(obj) // 属性都被放到__proto__里，产生对象的一个副本
// Object.create()方法创建一个新对象，使用现有的对象来提供新创建的对象的__proto__（MDN）
```

```js
Object.observe(obj, callback) // 用于异步的监视一个对象是否变化
```

```js
Object.is(a,b) // 判断两个值是否相等，指向同一内存地址
```

# 数组新增

indexOf，lastIndexOf，every，some，forEach，map，filter，reduce，reduceRight，Array.isArray

# 构造函数的返回值

**没有返回值**和返回**简单数据类型**时会返回构造函数的实例对象，当返回**对象类型**时会返回该值

# 继承

1. `Child.call(this)`
2. `Child.prototype = 深拷贝父级.prototype`
3. 修正子类的`constructor`

```js
var __proto = Object.create(Parent.prototype)
__proto.constructor = Child
Child.prototype = __proto
```



# 关于提升

​	函数提升和变量提升：说白了就是看见函数就全部按顺序先提上去，看见变量就先把变量声明全都按顺序提上去，提完以后开始执行代码吧。

# Call，Apply，bind

call逗号，apply数组

# label stament

js里也有像c语言的goto类似的语句，就是跳到某个被标记的语句，声明a: balabala

逗号表达式要求每一项都是表达式，并输出最后一项的结果,小括号包起来的内容可以被当成表达式来解析

```js
var temp=0;  
start:  
for(var i=0; i<5; i++) {  
    for(var m=0; m<5; m++) {  
        if(m==1) {  
            break start;  
        }  
        temp++;  
    }  
}  
alert(temp);
```

有start这个标签，并且在break时加上start，则弹窗是1，去掉start，则弹窗是5



# 闭包

最常用的闭包就是，函数的嵌套并且对外提供访问就会产生闭包

闭包由两部分组成：

1. 函数
2. 环境：函数创建时作用域内的局部变量

```js
function makeCOunter(init) {
  var init = init || 0
  return function() {
    return ++init
  }
}

var counter = makeCounter(10)
console.log(counter())
console.log(counter())
```

​	一般来说，函数执行完了就完了，里面的变量之类的就被释放了，但如果这个函数（设为fn）里面还有个函数（fn2）并且返回了这个函数，同时这个函数使用了外面函数的变量（闭包）。在这种情况下，你定义一个变量`a = fn()`，那么fn里的变量就会一直在内存中，其实也就是一直被里面的函数（fn2）所占用。直到你手动的·a = null·

# 惰性函数

```js
function eventBindGenerator() {
  if(ie) { return function() { ... } }
  else { return function() { ... } }
}
// 这样的化每次执行的时候都会问一遍, 但其实最好是就只问一遍就可以了，不用每次都问

var addEvent = eventBindGenerator()
// 以后就只用这个就好了
```



# this

this指向函数执行时的当前对象

1. 普通函数 - 严格模式：undefined	非严格：window或global（Node）
2. 构造函数：对象的实例
3. 对象的方法：对象本身

```js
document.getElementById('btn').onclick = function() {
  console.log(this) // 2. 这个btn按钮
}
```

**描述下new一个对象的过程**

- 创建一个新对象
- this指向这个新对象
- 执行代码，即对this赋值
- 返回this

# 事件流



完整的事件流分三阶段：捕获，目标阶段，冒泡

事件通过捕获到达目标元素，这个时候就是目标阶段，从目标元素上传到window对象就是冒泡阶段

**描述DOM事件捕获的基本流程：**

第一个接受到事件对象的是window

# 原型

每定义一个函数或者对象时，都会包含一些预定义的属性，函数由`prototype`，对象有`__proto__`

干嘛用，用来实现面向对象开发，类的继承

# 深浅拷贝

**浅拷贝**

​	除了直接赋值以外，可以通过 Object.assign 来解决这个问题，很多人认为这个函数是用来深拷贝的。其实并不是，Object.assign 只会拷贝所有的属性值到新的对象中，如果属性值是对象的话，拷贝的是地址，所以是浅拷贝。

​	也就是只会深拷贝第一层，后面的还是浅拷贝

​	另外我们还可以通过展开运算符 ... 来实现浅拷贝



**深拷贝**

这个问题通常可以通过 JSON.parse(JSON.stringify(object)) 来解决。

但是该方法也是有局限性的：

1. 会忽略 undefined
2. 会忽略 symbol
3. 不能序列化函数
4. 不能解决循环引用的对象（如下就是循环引用的意思）

```
let newObj = JSON.parse(JSON.stringify(obj))
```

​	当然你可能想自己来实现一个深拷贝，但是其实实现一个深拷贝是很困难的，需要我们考虑好多种边界情况，比如原型链如何处理、DOM 如何处理等等，所以这里我们实现的深拷贝只是简易版，并且我其实更推荐使用 [lodash 的深拷贝函数](https://link.juejin.im/?target=https%3A%2F%2Flodash.com%2Fdocs%23cloneDeep)。



# 函数式编程

​	主旨在于将复杂的函数复合成简单的函数，运算过程尽量写成一系列嵌套的函数调用。真正的火热是随着react高阶函数而逐步升温

## 纯函数

对于相同的输入，永远得到相同的输出，而且没有任何可观察的副作用，也不依赖外部环境的状态

如果要用函数式编程，函数必须要纯

Array.slice是纯函数，而splice就不是，因为第一次执行后会改变原数组的值

```js
var x = [1,2,3,4,5]
x.slice(0, 3)
x.slice(0, 3)
x.splice(0, 3)
x.splice(0, 3)

var a = 10; 
return x > a
// 这个就是依赖外部环境的意思，依赖a的值，就不是纯的，如何变纯呢
return x > 10 // 虽然纯了，但是可扩展性比较差了

// （柯里化优雅解决）
var x = min => (age => a > min)
var y = c(18)
y(20)
```

优点：

1. 不仅可以有效的降低系统的复杂度，还有很多很棒的特性，比如可缓存性，因为输入相同输出就一定相同

   lodash就是纯函数库

## 柯里化

​	传递给函数一部分参数来调用它，让它返回一个函数去处理剩余的参数。就是不一次性把所有的参数传完

```js
// 柯里化之前
function add(x, y) {
  return x + y
}
add(1, 2)

// 之后
function addX(y) {
  return function(x) {
      return x + y
  } 
}
addX(2)(1)
```



​	事实上柯里化是一种”预加载“函数的方法，通过传递较少的参数，得到一个已经记住了这些参数的新函数，某种意义上讲，这是一种对函数的缓存，是一种非常高效的编写函数的方法



## 函数组合

使用柯里化以后可能会变成洋葱式的：`f(g(h(x)))`

函数组合来解决

```js
const compose = (f, g) => (x => f(g(x)))

var first = arr => arr[0]
var reverse = arr => arr.reverse()
var last = compose(first, reverse)
last([1, 2, 3, 4, 5])
```



## 高阶函数

​	函数当参数，把传入的函数做一个封装，然后返回这个封装函数，达到更高的抽象



## 尾调用优化

​	什么是尾调用，指函数内部的最后一个动作是函数调用。该调用的返回值，直接返回给函数。

```js
function f(x) {
  // balabala
  return g(x)
}
```

​	尾调用有什么好处呢？我们知道函数的调用会在内存中形成一个调用记录，又称调用帧，保存调用位置和内部变量等信息，上面的情况如果`f(3)`就等同于`g(3)`，调用g之后，函数f就结束了，所以到最后一步就可以完全删除f的调用记录，值保留g的调用记录。

​	这就叫做"尾调用优化"（Tail call optimization），即只保留内层函数的调用记录。如果所有函数都是尾调用，那么完全可以做到每次执行时，调用记录只有一项，这将大大节省内存。这就是"尾调用优化"的意义。



### 尾递归 

​	函数调用自身，成为递归。如果尾调用自身，成为尾递归。递归非常耗费内存，因为需要同时保存成千上百个调用记录，很容易发生"栈溢出"错误（stack overflow）。但对于尾递归来说，由于只存在一个调用记录，所以永远不会发生"栈溢出"错误。

```js
function factorial(n) {
  if (n === 1) return 1;
  return n * factorial(n - 1);
}
```

上面代码是一个阶乘函数，计算n的阶乘，最多需要保存n个调用记录，复杂度 O(n) 。

如果改写成尾递归，只保留一个调用记录（永远都是更新当前的栈帧），复杂度 O(1) 。

```js
function factorial(n, total) {
  if (n === 1) return total;
  return factorial(n - 1, n * total);
}
```



# Promsie

解决异步的写法问题，即回调地狱，使代码结构更清晰

```js
let p = new Promise((resolve, reject) => {
  console.log(1) // 1
})
p.then(() => { // 2
  console.log(2)
}, () => {
  console.log(3) // 3
})
```

1. 将会执行这个函数，叫promise的回调函数
2. promise有then方法，promise内部resolve之后调用
3. then方法有两个回调，第二个回调会执行reject之后的操作

## 链式操作

```js
p.then((data) => { // 1
  console.log(data)
  return 1
}).then(data => {
  console.log(data)
  return 2
})
```

1. then方法本身返回的也是一个promise，（理解：`.then === new Promise`）默认就是resolve状态的promise，当then里面return了一个非promise对象时也是同理，默认是resolve状态的promise
2. 如果then内返回的是promise对象，则就按这个返回的promise走（理解：`return 1 === resolve(1)`）

## all和race方法

`Promise.all([p1, p2, p3]).then`

`Promise.race([p1, p2, p3]).then`



# 手写Promise

首先promise是一个类，并且有三个状态：pending（等待），fulfilled（成功），rejected（失败），默认是pengding

里面会传入一个函数，叫执行器，会立即执行

```js
const PENGDING = 'PENDING'
RESOLVED = 'RESOLVED'
REJECTED = 'REJECTED'

class Promise {
  constructor(exector) { // #1
    this.status = 'pending'
    this.value = undefined // #3
    this.reason = undefined
    this.onResolvedCallbacks = [] // #7
    this.onRejectedCallbacks = []
    let resolve = (value) => { // #2
      if(this.status === 'pending') { // #4
        this.value = value
        this.status = 'resolved'
        this.onResolvedCallbacks.forEach(fn => fn())
      }
    }
    let reject = (reason) => {
      if(this.status === 'pending') {
	    this.reason = reason
        this.status = 'rejected'
        this.onRejectedCallbacks.forEach(fn => fn())
      }
    }
    try { // #5
      exector(resolve, reject)
    } catch(e) {
      reject(e)
    }
  }
    
  then(onfulfilled, onrejected) { // #6
    let promsie2 = new Promise((resolve, reject) => { // #9
      if(this.status === 'resolved') {
        setTimeout(() => { // #12
          try {
            let x = onfulfilled(this.value) // #11
            resolvePromise(promise2, x, resolve, reject)              
          } catch (e) {
            reject(e)
          }
        })
      }
      if(this.status === 'rejected') {
        let x = onrejected(this.reason)
        resolvePromise(promise2, x, resolve, reject)
      }
      if(this.status === 'pending') {
        this.onResolvedCallbacks.push(() => { // #8
          let x = onfulfilled(this.value)
          resolvePromise(promise2, x, resolve, reject)
        })
        this.onRejectedCallbacks.push(() => {
          let x = onrejected(this.reason)
          resolvePromise(promise2, x, resolve, reject)
        })
      }     
    })
    return promise2 // #10
  }
}

const resolvePromise = (promise2, x, resolve, reject) => {
  if(promise2 === x) { // #13
    return reject(new TypeError)
  }
  if(typeof x === 'object' && x !== null || typeof x === function) {
    try {
      let then = x.then
      if(typeof then === 'function') {
        then.call(x, y => {
          resolve(y)
        }, r => {
          reject(r)
        })
      } else {
        resolve(x)
      }
    } catch (e) {
      reject(e)
    } 
  } else {
    resolve(x)
  }
}

module.export = Promise

// let promise = require('./Promise')
```

1. 我们`new Promise()`的时候传的是一个执行器，这个执行器会立即执行
2. 回调会接收两个函数类型的参数，箭头函数保证this指向没有问题
3. `resolve, reject`的时候要把参数传给then，所以要记录下来
4. 状态改了之后就不能再被改了，防止你既调resolved，又调rejected
5. 回调函数也可以直接抛出错误
6. 之后就开始调用then了，then目前有两个参数，成功就执行第一个，失败执行第二个
7. 这一步之前会有什么问题呢，想一下如果在`exector`内执行异步操作以后在resolved，这会儿走到then函数的时候其实promise的状态还没有被改变，还是pending状态，就会有问题。所以我们定义两个数组，把成功的回调和失败的回调分开放进去，当状态改变时再一一执行。
8. 这样push在未来有需要的时候我可以加入一些自己的逻辑
9. 解决链式调用的问题
10. then调用完以后返回新的promise，方便链式调用
11. then里面返回一个普通值会传给下一个then，x可能时普通值，也可能是promise，如果是promise我就直接执行，如果是普通值就直接resolve，如果是失败的promsie，就调reject。因为后面也有相同的操作，所以把这个方法抽取出来。就是`resolvePromise`
12. 如果不用setTimeout的话在下面的 `resolvePromise`里拿不到promise2，因为new还没走完呢，走完才会有promise2。后面的也同理用setTimeout包起来
13. 自己不能返回自己 `promise2 = p.then(() => { return promise2 })`



# async await

```js
async function fn() {
  let p = await new Promise((resolve) => {
    setTimeout(() => {
      resolve(10)
      console.log(1)
    })
  })
  console.log(p) // 10 p是resolve内的值
}
```

















---------------------------------

​	**按值传递和按引用传递（标准说法）**



​	`var b = c = 20` 相当于`var b = 20; c = 20;` 也即是说c变成全局的了



​	js是函数级作用域



​	NaN和NaN不管两等还是三等都是false，Object.is(NaN, NaN)就是true



​	JSON.stringfy和JSON.parse都是可以传第二个参数的，是一个回调，可以对key value做一些处理



​	Babel是一个JavaScript编译器。babel只会转换新的js语法，而不转换新的API，比如Iterator，Generator，Set，Proxy，Promsie等等，所以需要`babel-polyfill`，polyfill就是一段代码，或者一个插件

```js
<script src="babel.min.js"><script>
<script type="text/babel">
  ... // 写ES6代码
<script>


// 解构
const [a, b, c] = [arr]
const {a, b} = obj

// includes, startsWith, endsWith
b = '345'
const a = [1, 2, ...b] // [1, 2, 3, 4, 5]

const result = {
  [k+1]: 1,
  q() {
      
  }
}

arr = [1,2,3]
for (let a of arr) {
  console.log(a) // 1 2 3 只能数组，不能是对象
}
```

​	

```java
<ul>
  <li>1</li>
  <li>2</li>
  <li>3</li>
  <li>4</li>
  <li>5</li>
  <li>6</li>
</ul>

<script>
  var list_li = document.getElementsByTagName("li")
  for(var i=0; i< list_li.length; i++) {
    list_li[i].onclick = function() { // #1
      console.log(i)
    }
  }
</script>
```

​	我本意是想点哪个输出几，但是不行

​	点击事件，setTimeout，Ajax都是异步的。都会在同步代码执行完后才会执行，所以如果有个死循环的话，异步代码将永远不会执行

1. 闭包解决
2. let
3. this

```js
(function(i){ // 1
  list_li[i].onclick = function() {
    console.log(i)
  }  
})(i)

list_li[i].onclick = function() { // 3, this是谁调用指谁
  console.log(this.innerHTML)
} 
```



- 第一题

```js
alert(a)
a() // #1
var a = 3
function a() {
  alert(10)
}
alert(a)
a = 6
a()
```

- function a

- 10

- 3

- a is not a function 报错

1. 即使在函数的后面声明函数，依旧可以执行，也就是函数提升

   如果函数和变量同时提升，变量名没有被赋值，则函数生效



```js
var a = function yideng(num) {
  yideng = num
  console.log(typeof yideng) // #1
}
a(1)
```

1. yideng时只读的，所以输出function。且只能在函数内部使用，外部访问不了

   

- 第二题

```js
this.a = 20
var test = {
  a: 40,
  init: () => { // function() #1
    console.log(this.a)
    function go() {
      // this.a = 60
      console.log(this.a)
    }
    go.prototype.a = 50
    return go
  }
}

// var p = test.init()
// p()
new (test.init())()
```

20 50

this是谁调用指谁，如果`test.init`，this就是test，如果fn = test.init，this就是window

1. 箭头函数会绑定父级作用域，这里的父级是window



```js
function test(a) {
  this.a = a
}
test.prototype.a = 20
test.prototype.init = function() {
  console.log(this.a)
}
var s = new test()
s.init() // #1
```

1. 输出30，构造函数优先于原型链，找不到的话才去原型链上找

es6简写的函数体不能被new



- 第三题

```js
function C1(name) {
  if(name) this.name = name
}
function C2(name) {
  this.name = name
}
function C3(name) {
  this.name = name || 'fe'
}
C1.prototype.name = "yideng"
C2.prototype.name = "lao"
C3.prototype.name = "yuan"
console.log((new C1().name) + (new C2().name) + (new C2().name))
```

`yideng undefined 'fe'`

先执行`new C1()` 再取`.name`



- 第五题

```js
function yideng() {
  console.log(1)
}
(function(){
  if(false) {
    function yideng() {
      console.log(2)
    }
  }
  yideng()
})()
```

在不同的浏览器结果不一样

1. 老式浏览器函数提升结果为2（ie6，ie7）
2. 现代浏览器报错，提升以后相当于`var yideng = undefined`

所有的变量会提升到函数作用域的顶端，因为js是函数作用域



- 第六题

请用一句话算出0-100之间学生的等级，90-100为1等，80-90为2等，不允许使用if，else

`10 - 98 / 10 = 1等生`



- 第七题

请用一句话便利变量a，已知`a=“abc”`

`[...new Set(a)]	Array.from(a)`

**继承原型的方法**

`Child.prototype = Object.create(Parent.prototype)`



- 第八题

```js
var length = 10
function fn() {
  console.log(this.length)
}

var yideng = {
  length: 5,
  method: function(fn) {
    fn()
    arguments[0]()
  }
}

yideng.method(fn, 1)
```

10	2

window.length等于页面iframes的个数

拓展

```js
function yideng() {}
yideng.prototype.a = 11
console.log(yideng.a) // undefined
```

函数的话必须要new了以后才回去原型链上去找















