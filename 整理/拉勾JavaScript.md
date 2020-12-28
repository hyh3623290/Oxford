# ES2015

​	let 只会在当前代码块中生效，所以可以解决循环的问题

​	循环中括号是一个是一个作用域，块内部是一个作用域，可以这么理解

## 基本

```js
// 数组解构
const [, , a] = arr
const [a = 2, ...foo] = arr

// 对象解构
const { name: newname = 'foo' } = obj

// 模板字符串
function tag(strings, name) {}
const result = tag`123` // 带标签函数

// includes（数组也有），startsWith，endsWith

// 参数默认值
function foo(baz, bar = '123') { 
  默认值参数要放在最后，不然会出错
}

// 剩余参数
function foo(...args) {}

// 箭头函数里没有this的机制，所以它不会改变this的指向
// 箭头函数外面的this是什么，就是什么，任何情况下都不会发生改变
const person = {
  name: 'tom',
  sayNameAsync: function() {
    setTimeout(function() {
      console.log(this.name)
    }, 1000)
  }
}
person.sayNameAsync() // undefined

// Object.assign
将多个对象的属性复制到一个目标对象中

// Object.is
两个对象是否相等，
+0 === -0 返回true
NaN == NaN 返回false
而使用Object.is正好相反

// Math.pow(2, 10)
2的10次方
```



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

​	静态类，不能new来实例化，只能使用一些静态类的静态方法：`Reflect.get()`，如Math。它封装了一系列对对象的底层操作。有13个方法，和Proxy的方法一样。其实的话，Proxy里面默认就是调用了Reflect的方法，比如没有自己写get方法的话默认就是返回Reflect的get方法。

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



## class

```js
class Person {
  constructor(name) {
    this.name = name
  }
  say() {}
  static create(name) {
    return new Person(name)
  }
}

class Student extends Person {
  constructor(name, id) {
    super(name)
    this.id = id
  }
  hello() {
    super.say()
    ...
  }
}

const p = new Person('tom')
p.say()
const p = Person.create('tom')

```

- 实例方法和静态方法，只能通过实例来调用和通过类来调用
- extends以后student就拥有Person的所有的成员了
- super始终指向父类，super(name)调用它就是调用父类的构造函数



## Set

​	集合，与数组非常类似，但不允许重复

```js
const s = new Set()

s.add(1).add(2)
s.delete(100) // 成功返回true
s.clear()
s.has(100)
s.forEach(i => console.log(i))
for(let i of s)
s.size()


```

`for of` 可遍历数组	`[...new Set(arr)]` 可以去重



## Map

​	类似对象，健值对集合，普通对象的健只能是字符串

```js
const m = new Map()

const tom = { name: 'tom' }
m.set(tom, 90) // 以对象作为key
m.delete(tom)
m.clear()
m.get(tom)
m.forEach((value, key) => console.log(value, key))
```



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



## 生成器

​	Generator，避免异步回调嵌套太深

```js
const *foo() {
  console.log('1')
  return 2
}
const result = foo()
console.log(result) // 生成器对象
console.log(result.next()) // '1' { value: 2, done: true }
```

​	你发现调用生成器函数不会立即执行，而是得到一个生成器对象，只有next之后函数体才开始执行，执行到yield停下并会作为value。最大的特点是惰性执行，就是抽一下动一下

🍇 **应用**

```js
const todos = {
  life: ['a', 'b', 'c'],
  learn: ['d', 'e', 'f'],
  [Symbol.iterator]: function *() {
    const all = [...this.life, ...this.learn]
		for(const item of all) { // 自动生成迭代器接口
      yield item
    }
  }  
}
```

🍇 **异步方案**

```js
function *main() {
  try {
    const users = yield ajax('url')
    console.log(users)
    const goods = yield ajax('url2')
    console.log(goods)    
  } catch(e) {
    ...
  }
}

const g = main()
const result = generator.next()
result.value.then(res => {
  const result2 = g.next(res)
  result2.value.then(res => {
    g.next(res)
  })
})

// 更通用 -> 递归
function handleResult(result) {
  if(result.done) return
  result.value.then(res => {
    handleResult(g.next(res))
  }, error => {
    g.throw(error)
  })
}
handleResult(g.next())
// 接下来稍微封装一下，目前社区已有更完善的库，就是co
```







## ES2017

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



## 异步

​	JavaScript为什么单线程：假设两个线程，一个修改了dom，一个删除了dom...

​	首先说同步，JS有个调用栈的概念：首先JS引擎会将整体的代码都加载进来，然后在调用栈中加载一个匿名的调用（相当于把全局代码放到一个匿名函数去执行），然后如下：

1. 将第一条语句压入调用栈中，并执行，执行完弹出调用栈
2. 将第二条语句压入调用栈...
3. 重复...（函数声明不会产生任何调用，所以先跳过）

### Event Loop

​	异步任务会被放进消息队列中，当调用栈的代码为空时，event loop就会从消息队列中取出任务放入调用栈中。所以它就做一件事情，监听调用栈和消息队列。

### promise

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



🍇 执行时序 / 宏任务 & 微任务

​	回调队列中的任务成为宏任务，宏任务执行过程中可能会加上一些额外的需求，可以选择作为一个新的宏任务进到队列中进行排队，但是也可以作为当前任务的微任务，直接在当前任务执行过后立即执行，而不是等到末尾继续排队。Promise的回调会作为微任务执行。微任务就是后面才提出的，目的就是为了提高整体的响应能力

​	打个比方，我办存款业务排队，办完之后突然想办个信用卡，则不需要重新排队去取号。

​	常用微任务：promise，MutationObserver，process.nextTick（Node），async，await

​	其余大部分异步都是宏任务

​	补充：eventloop：调用栈，消息队列，微任务队列。微任务队列会在调用栈情空时立即执行，并且处理期间新加入的微任务也会一同执行



### async await

​	async函数返回值也是promise









