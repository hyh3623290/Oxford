# 2.25

## instanceof 的内部机制是什么

​	instanceof内部机制是通过原型链判断的

## 实现一个通用的判断类型的方法

```js
function getType(obj){
  let type  = typeof obj;
  if (type !== "object") {    // 先进行typeof判断，如果是基础数据类型，直接返回
    return type;
  }
  // 对于typeof返回结果是object的，再进行如下的判断，正则返回结果
  return Object.prototype.toString.call(obj).replace(/^\[object (\S+)\]$/, '$1');  // 注意正则中间有个空格
}
```

​	

## 闭包的定义是什么

​	闭包的定义其实很简单：函数 A 内部有一个函数 B，函数 B 可以访问到函数 A 中的变量，那么函数 B 就是闭包。



## 常用的深浅拷贝方法及某个深拷贝方法的缺点

​	Object.assign，`...`

​	`JSON.parse(JSON.stringify(object))`

- 拷贝的对象的值中如果有函数、undefined、symbol 这几种类型，经过 JSON.stringify 序列化之后的字符串中这个键值对会消失；

- 无法拷贝对象的循环应用，即对象成环 (obj[key] = obj)。

## 手写浅拷贝

```js
const shallowClone = (target) => {
  if (typeof target === 'object' && target !== null) {
    const cloneTarget = Array.isArray(target) ? []: {};
    for (let prop in target) {
      if (target.hasOwnProperty(prop)) {
          cloneTarget[prop] = target[prop];
      }
    }
    return cloneTarget;
  } else {
    return target;
  }
}
```





## 手写深拷贝

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



## __proto__ 和 prototype有什么关系

​	打印obj，你发现`obj.__proto__.constructor.prototype === obj.__proto__`

## ES5继承的方法

## map, filter, reduce

​	map 是对每个数组做操作返回

​	filter 是只返回为true的

​	reduce 可以将数组中的元素通过回调函数最终转换为一个值，接受两个参数，分别是回调函数和初始值

```js
const sum = arr.reduce((acc, current) => acc + current, 0)
```



## 说一下Generator

​	通过 * 来声明generator函数，执行它普通函数不同，会返回一个迭代器，要执行迭代器的next才会继续往下执行，第一次执行时，会暂停在第一个yield处，并向外返回一个值。如果传参数的话，会代替上一个yield的返回值

## 定时器有什么问题吗

注意点

- 因为 JS 是单线程执行的，如果前面的代码影响了性能，就会导致 `setTimeout` 不会按期执行
- 如果定时器 `setInterval` 执行过程中出现了耗时操作，多个回调函数会在耗时操作结束以后同时执行

## 手写 call apply bind

```js
Function.prototype.myCall = function(context, ...args) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  context = context || window
  context.fn = this
  const result = context.fn(...args)
  delete context.fn
  return result
}
```

```js
Function.prototype.myApply = function(context) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  context = context || window
  context.fn = this
  let result
  // 处理参数和 call 有区别
  if (arguments[1]) {
    result = context.fn(...arguments[1])
  } else {
    result = context.fn()
  }
  delete context.fn
  return result
}
```

```js
Function.prototype.myBind = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  const _this = this
  const args = [...arguments].slice(1)
  // 返回一个函数
  return function F() {
    // 因为返回了一个函数，我们可以 new F()，所以需要判断
    if (this instanceof F) {
      return new _this(...args, ...arguments)
    }
    return _this.apply(context, args.concat(...arguments))
  }
}
```

​	1

## 手写 new

在调用 `new` 的过程中会发生以上四件事情：

1. 新生成了一个对象
2. 链接到原型
3. 绑定 this
4. 返回新对象

```js
function create() {
  let obj = {}
  let Con = [].shift.call(arguments)
  obj.__proto__ = Con.prototype
  let result = Con.apply(obj, arguments)
  return result instanceof Object ? result : obj
}
```

​	1

## 手写 instanceof

```js
function myInstanceof(left, right) {
  let prototype = right.prototype
  left = left.__proto__
  while (true) {
    if (left === null || left === undefined)
      return false
    if (prototype === left)
      return true
    left = left.__proto__
  }
}
```

​	1



# 2.27

## Fiber阅读











# ^_^

周末可以看手写代码的题，如fiber，diff等，周内可以看简单的，也可以看interview的