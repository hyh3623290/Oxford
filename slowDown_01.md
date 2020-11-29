# 闭包

## 概念

​	闭包 - 背后的故事 - 引出一系列东西 - 如作用域，作用域链，变量提升、暂时性死区、执行上下文等附加话题

​	如果一个函数引用了一些不属于它当前作用域的变量，就称这个函数为闭包

```js
var a = 1 // LHS
var a = number // RHS
```

**词法作用域和动态作用域**

- 词法作用域： 在代码书写的时候完成划分，作用域链沿着它**定义的位置**往外延伸
- 动态作用域： 在代码运行时完成划分，作用域链沿着它的**调用栈**往外延伸

​	JS 的作用域遵循的就是词法作用域模型

```js
var name = 'xiuyan';

function showName() {
    console.log(name);
}

function changeName() {
    var name = 'BigBear';
    showName();
}

changeName();
// 词法：输出'xiuyan'（JS）
// 动态：输出'BigBear'
```

**如何修改词法作用域**

1. `eval`

```js
function showName(str) {
  eval(str)
  console.log(name)
}

var name = 'xiuyan'
var str = 'var name = "BigBear"'

showName(str) // 输出 BigBear
```



🌰 -**第一题 下面代码输出什么**

```js
for (var i = 0; i < 5; i++) {
    setTimeout(function() {
        console.log(i);
    }, 1000);
}

console.log(i); // 5个5
```

​	函数它自身的作用域里，是不是压根儿没有 i 这个变量，得去上层作用域找。函数第一次被执行，也是 1000ms 以后的事情了，此时它试图向外、向上层作用域（这里就是全局作用域）去找一个叫 i 的变量。此时 for 循环早已执行完毕， i 也进入了尘埃落定的最终状态 ——5。

那么如何输出正常的呢

1. setTimeout 函数的第三个参数会传给里面的回调

```js
for (var i = 0; i < 5; i++) {
    setTimeout(function(j) {
        console.log(j);
    }, 1000, i);
}
```

2. 在 setTimeout 外面再套一层函数，利用这个外部函数的入参来缓存每一个循环中的 i 值

```js
var output = function (i) {
    setTimeout(function() {
        console.log(i);
    }, 1000);
};

for (var i = 0; i < 5; i++) {
    // 这里的 i 被赋值给了 output 作用域内的变量 i
    output(i);  
}
```

3. 和第二种思路比较相似，同样是在 setTimeout 外面再套一层函数，只不过这个函数是一个**立即执行函数**。利用立即执行函数的入参来缓存每一个循环中的 i 值

```js
for (var i = 0; i < 5; i++) {
    // 这里的 i 被赋值给了立即执行函数作用域内的变量 j
    (function(j) {  
        setTimeout(function() {
            console.log(j);
        }, 1000);
    })(i);
}
```



🌰 - **第二题 下面代码输出什么**

```js
var a = 1;
function test(){
    a = 2;
    return function(){
        console.log(a);
    }
    var a = 3;
}
test()();
```

​	画图，变量提升，此外记住词法作用域是在**书写的过程中，根据你把它写在哪个位置来决定**，所以会输出2



## 应用

1. **我们可以用闭包模拟私有变量**

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
      // 使用 fetch 进行登录请求，同上，此处省略
    }
  }
  return User
})()

let user = new User('xiuyan', 'xiuyan123')
user.username // xiuyan
user.password // undefined
```

​	像 password 这样的变量，我们希望它仅在对象内部生效，无法从外部触及，这样的变量，就是私有变量。内部可以拿到、外部拿不到 => 函数作用域



2. **偏函数和柯里化**

​    其实柯里化和偏函数并不复杂（只是名字有点拗口），他们都是可以帮我们把需要多个入参的函数，转化为需要更少入参的函数的方法。柯里化应该就是只需要单一参数。

为什么需要这个

​	假设我们现在是一家电商公司，旗下有多个电商站点。为了确保商品名的唯一性，我们考虑使用 prefix（一个标识不同站点的前缀字符串）、 type（商品类型）、name（商品原本名称）三个字符串拼接的方式来为商品生成一个完整版名称。对应的方法如下：

```js
function generateName(prefix, type, itemName) {
    return prefix + type + itemName
}
```

​	如果是作为一个细分工种的程序员，我负责的东西可能更具体了，比如仅仅负责 “大卖网” 站点下的 “母婴” 类商品，那么我每次生成完整名称的时候，调用这个函数就是这样传参的：

```js
generateName('大卖网', '母婴', itemName)
```

​	此时我们多么希望，有一种魔法，可以让函数在必要的情况下帮我们 “记住” 一部分入参。在这个场景下，柯里化可以帮我们很大的忙。现在我们对 generateName 进行柯里化

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

// “记住”prefix，生成大卖网母婴商品名专属函数
var salesBabyName = salesName('母婴')

// "记住“prefix和type，生成洗菜网生鲜商品名专属函数
var vegFreshName = generateName('大卖网')('母婴')

// 输出 '大卖网母婴奶瓶'
salesBabyName('奶瓶')
```

​	这样一来，原有的 generateName (prefix, type, name) 现在经过柯里化已经变成了 generateName (prefix)(type)(itemName)。通过后者这种形式，我们可以选择性地决定是否要 “记住” prefix、type，从而即时地生成更加符合我们预期的、复用程度更高的目标函数。此外，柯里化还可以帮助我们以嵌套的形式把多个函数的能力组合到一起，这就是柯里化的魅力。

​	偏函数应用是不强调 “单参数” 这个概念的。它的目标仅仅是把函数的入参拆解为两部分。

## JS内存管理机制

​	闭包中与内存泄漏之间的关系 - 有的语言会开放内存管理 - 比如 C 语言中的 `malloc()` 和 `free()` 方法 ，这些方法的暴露，使得开发者能够切身感受到内存管理这件事情的存在。而在另一些语言 —— 比如 JS 中，这种能力是被 “隐藏” 了的：JS 并没有暴露任何内存操作给开发者，而是自己默默地自动完成了所有的管理动作。

1⃣️ **栈内存与堆内存**

​	基本类型被放在 JS 的栈内存里存储。引用类型放在 JS 的堆内存里存储。大家知道，堆和栈分别是不同的数据结构。栈是线性表的一种，而堆则是树形结构。访问基本类型之间从栈里取值即可，访问引用类型需要两步走：

1. 从栈中获取变量对应对象的引用（即它在堆内存中的地址）
2. 拿着 1 中获取到的地址，再去堆内存空间查询，才能拿到我们想要的数据



2⃣️ 垃圾回收

​	以上是内存的分配和使用，后面是内存的释放。

​	每隔一段时间，JS 的垃圾收集器就会对变量做 “巡检”。当它判断一个变量不再被需要之后，它就会把这个变量所占用的内存空间给释放掉，这个过程叫做垃圾回收。

​	那么 JS 是如何知道一个变量是否不被需要的呢？ 这里就引出了内存管理的又一个考点 —— 垃圾回收算法（引用计数法和标记清除法）。

**引用计数法**

​	最初级 - 已淘汰 - 内存中每个值都会对应一个引用的计数，当垃圾收集器感知到某个值的引用计数为 0就会释放它，比如我们给某个值赋值为null。为什么被废弃呢，因为循环引用无法解决的问题

**标记清除法**

​	在标记清除算法中，一个变量是否被需要的判断标准，是它是否可抵达 。

​	这个算法有两个阶段，分别是标记阶段和清除阶段：

- 标记阶段：垃圾收集器会先找到根对象（Window或Global），从根对象出发，垃圾收集器会扫描所有可以通过根对象触及的变量，这些对象会被标记为 “可抵达”。
- 清除阶段： 没有被标记为 “可抵达” 的变量，就会被认为是不需要的变量，这波变量会被清除



# this

## 指向问题

​	this 的指向是在调用时决定的，而不是在书写时决定的。这点和闭包恰恰相反

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

​	结果是 BigBear—— 竟然不是 xiaoming？的确，我们打眼看过去，直觉上肯定会认为是 you 这个对象在调用 hello 方法、进而调用 targetFunc，所以此时 this 肯定指向 you 对象啊。在面试命题过程中，this 指向问题如果想往难了出， 就会像楼上这样把声明位置和调用位置故意揉在一起

⚠️【总结】： 只要是函数，一律先看它前面有没有点，没有点肯定就是window了，要清楚一个，比如`a.f()`，你要小心fn里面是不是还套用了一个函数`g()`，这个`g()`里面的this可不是`a`哦，要看这个`g()`有没有点。尤其要小心立即执行函数哦，因为立即执行函数作为一个匿名函数，在被调用的时候，我们往往就是直接调用，而不会（也无法）通过属性访问器（ 即 [xx.xxx](http://xx.xxx)） 这样的形式来给它指定一个所在对象，所以它的 this 是非常确定的，就是默认的全局对象 `window`。`setTimeout `和 `setInterval `也是同理哦

⚠️ 严格模式下，普通函数作用域内，this指向被绑定的对象，没有绑定就是undefined（不是window），全局作用域下才是window

⚠️ 箭头函数下认词法作用域，所以说箭头函数中的 this，你就不要管点的事情了，由你书写它的位置决定

⚠️ 对象里哪有this哦沙雕，`obj = { x: this } // this是window`



## 改变指向

1. 改变后直接调用
   - `fn.call(obj, arg1, arg2)`
   - `fn.apply(obj, [arg1, arg2])`
2. 仅仅改变不调用
   - `fn.bind(bj, arg1, arg2)`



🌰 - **手写call**

```js
Function.prototype.myCall = function(context, ...args) {
  // step1: 把函数挂到目标对象上（这里的 this 就是我们要改造的的那个函数）
  context.func = this
  // step2: 执行函数，利用扩展运算符将数组展开
  context.func(...args)
  // step3: 删除 step1 中挂到目标对象上的函数，把目标对象”完璧归赵”
  delete context.func
}
// fn.call() -> fn也是一个对象哦，所以call里的this就是fn
```



# 执行上下文

​	执行上下文，从定义上理解，是 “执行代码的环境” -- 分为全局上下文和函数上下文。每一个执行上下文都会经历这样一个生命周期

- 创建阶段 —— 执行上下文的初始化状态，此时一行代码都还没有执行，只是做了一些准备工作
- 执行阶段 —— 逐行执行脚本里的代码

🌰 **- 理解变量提升**

​    所谓的 “变量提升”，只是变量的创建过程（在上下文创建阶段完成）和真实赋值过程（在上下文执行阶段完成）的不同步带来的一种错觉。

## 调用栈

​	我们看到函数执行完毕后，其对应的执行上下文也随之消失了。这个消失的过程，我们叫它” 出栈 “—— 没错，在 JS 代码的执行过程中，引擎会为我们创建” 执行上下文栈 “（也叫调用栈）。

🌰 **- 理解作用域**

​	作用域是什么？之前，我们认为作用域是” 访问变量的一套规则 “。但现在我要告诉大家，作用域其实就是当前所处的执行上下文。我们知道，作用域在嵌套的情况下，外部作用域是不能访问内部作用域的变量的。现在，结合调用栈的情况，相信你会更清楚这其中的原因：

​	我们看到，最初处于外部作用域（testA、全局上下文）时，testB 对应的上下文还没有被推入调用栈；而当 testB 执行结束、代码执行退回到外部作用域时，testB 早已从栈顶弹出。这意味着，每次位于外部作用域时，testB 的执行上下文都压根不存在于调用栈内。此时就算 testA 函数上下文和全局上下文无论如何也找不到任何关于 testB 的线索，自然访问不到它内部的变量啦！

🌰 **- 闭包 —— 特殊的 “弹出”**

​	在执行上下文的创建阶段，跟着被创建的还有作用域链！这个作用域链在函数中以内部属性的形式存在，在函数定义时，其对应的父变量对象就会被记录到这个内部属性里。闭包正是通过这一层作用域链的关系，实现了对父作用域执行上下文信息的保留。



# 原型

​	原型是 JavaScript 面向对象系统实现的根基。

```js
class Dog {
  constructor(name ,age) {
   this.name = name
   this.age = age
  }
  
  eat() {
    console.log('肉骨头真好吃')
  }
}
```

完全等价于

```js
function Dog(name, age) {
  this.name = name
  this.age = age
}
Dog.prototype.eat = function() {
  console.log('肉骨头真好吃')
}
```

​	构造函数有个`prototype`，`prototype`有个`constructor`，`constructor`指向构造函数。

​	实例有个`__proto__`，指向构造函数的`prototype`。

🌰 **- new做了什么**

- 为这个新的对象开辟一块属于它的内存空间
- 把函数体内的 this 指到 1 中开辟的内存空间去
- 将新对象的`__proto__ `这个属性指向对应构造函数的 prototype 属性，把实例和原型对象关联起来
- 执行函数体内的逻辑，最后即便你没有手动 return，构造函数也会帮你把创建的这个新对象 return 出来

# Promise

​	注意一个Promise的值穿透问题，then 方法的入参只能是函数。

```js
Promise.resolve(1)
  .then(Promise.resolve(2))
  .then(3)
  .then()
  .then(console.log)
// 输出1
```

​	具体到我们这个题里，第一个 then 方法中传入的是一个 Promise 对象，then 说：”我不认识“；第二个 then 中传入的是一个数字， then 继续说”我不认识“；第四个干脆啥也没穿，then 说”入参undefined了，拜拜“；直到第五个入参，一个函数被传了进来，then 哭了：”终于等到一个我能处理的！“，于是只有最后一个入参生效了。

​	我们最初 resolve 出来那个值，穿越了一个又一个无效的 then 调用，就好像是这些 then 调用都是透明的、不存在的一样，因此这种情形我们也形象地称它是 Promise 的“值穿透”。

⚠️ 手写Promise - 15，16章

# 变量提升与暂时性死区

​	事实上，JS也是有编译阶段的，它和传统语言的区别在于，JS不会早早地把编译工作做完，而是一边编译一边执行。简单来说，所有的JS代码片段在执行之前都会被编译，只是这个编译的过程非常短暂（可能就只有几微妙、或者更短的时间），紧接着这段代码就会被执行。

​	正是在这个短暂的编译阶段里，JS 引擎会搜集所有的变量声明，并且提前让声明生效。至于剩下的语句，则需要等到执行阶段、等到执行到具体的某一句的时候才会生效。这就是变量提升背后的机制。



⚠️ **const**：引用类型的属性值（包括数组的元素）可以被更改，只要你不修改引用的指向。



```js
var me = 'xiuyan';

{
	me = 'bear';
  // -------
	let me; 
}
```

​	事实上，这段代码啥也运行不出来，它会报错：这是因为 ES6 中有明确的规定：如果区块中存在 let 和 const 命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。假如我们尝试在声明前去使用这类变量，就会报错。这一段会报错的危险区域，有一个专属的名字，叫”暂时性死区“。在我们的 demo 中，以注释线为界，上面的区域就是暂时性死区：



# ES6

## 数组解构

```js
const [a, b, c] = [1, 2, 3]
const [a, ,c] = [1,2,3] // 中间位留空
```



## 对象解构

​	对象解构基本都会了，但是有时我们会遇到一些嵌套程度非常深的对象：

```js
const school = {
  classes: {
    stu: {
      name: 'Bob',
      age: 24,
    }
  }
}

const { classes: { stu: { name } }} = school

// 如果你接到的需求是给 name 起个新名字，采取 属性名：新变量名 这种形式就好：
const { classes: { stu: { name: newName } }} = school
```



## ...

```js
// 1. 对象拓展

// 2. 数组拓展
console.log(...['haha', 'hehe', 'xixi']) // haha hehe xixi

function mutiple(x, y) {
  return x*y
}

const arr = [2, 3]
mutiple(...arr) // 6

const newArr = [...arr1, ...arr2] // 合并两个数组

// 3. rest参数
function mutiple(...args) {
  console.log(args) // [1, 2, 3, 4]
  let result = 1;
  for (var val of args) {
    result *= val;
  }
  return result;
}

mutiple(1, 2, 3, 4) // 24
```

# 类数组

1. 它必须是一个对象
2. 它有 length 属性

```js
const book = {
  name: 'how to read a book',
  age: 10,
  length: 300 
} // 这是一个类数组对象
```

如何把类数组对象转换为真正的数组

```js
// 1.Array原型上的slice方法
const arr = Array.prototype.slice.call(arrayLike);

// 2.Array.from方法(ES6)
const arr = Array.from(arrayLike);
```

​	此外，扩展运算符——"…"也可以把类数组对象转换为数组，前提是这个类数组对象上部署了遍历器接口。在这个例子中，book 没有部署遍历器接口，所以这条路走不通。但一些对象，比如函数内部的 arguments 变量（它也是类数组对象），就满足条件，可以用这种方法来转换：

```js
function demo() {
  console.log('转换后的 arguments 对象：', [...arguments])
} 

demo(1, 2, 3, 4) // 转换后的 arguments 对象：[1, 2, 3, 4]
```























# --

【参考文献】

🌰 - muke小册解锁前端面试

```js
Function.prototype.myCall = function(context, ...args) {
  context.func = this
  context.func(args)
  delete context.func
}
```