# 数据类型

## 8种数据类型

​	Undefined，Null，Boolean，String，Number，Symbol，BigInt

​	Object：Array，RegExp，Date，Math，Function

​	前 7 种类型为基础类型，最后 1 种（Object）为引用类型

​	基础类型存储在栈内存，被引用或拷贝时，会创建一个完全相等的变量；

​	引用类型存储在堆内存，存储的是地址，多个引用指向同一个地址，这里会涉及一个“共享”的概念。



## 数据类型检测

### typeof

```js
typeof null // 'object'  JS 存在的一个悠久 Bug
typeof [] // 'object'
typeof console // 'object'
typeof console.log // 'function'
```

​	引用类型除了`function`，其余都是`object`



### instanceof

​	通过 instanceof 我们能判断这个对象是否是之前那个构造函数生成的对象

```js
let Car = function() {}
let benz = new Car()
benz instanceof Car // true
let car = new String('Mercedes Benz')
car instanceof String // true
let str = 'Covid-19'
str instanceof String // false

// instanceof 可以准确地判断复杂引用数据类型，但是不能正确判断基础数据类型；
```

**实现自己的instanceof**

```js
function myInstanceof(left, right) {
  if(typeof left!=='object' || left === null) return false
  let proto = Object.getPrototypeOf(left)
  while(true) {
    if(proto === null) return false
    if(proto === right.prototype) return true
    proto = Object.getPrototypeOf(left)
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





## 数据类型转换

```js
'' == null    // false
'' == 0        // true
[] == 0        // false or true?
[] == ''       // false or true?
[] == ![]      // false or true?
null == undefined // true
null == 0 // false
Number(null)     // 0
Number('')      // 0
parseInt('');    // NaN
{}+10           // 返回什么？
let obj = {
    [Symbol.toPrimitive]() {
        return 200;
    },
    valueOf() {
        return 300;
    },
    toString() {
        return 'Hello';
    }
}
console.log(obj + 200); // 这里打印出来是多少？
```



### 强制类型转换

​	强制类型转换方式包括 Number()、parseInt()、parseFloat()、toString()、String()、Boolean()

**Number() 方法的强制转换规则**

- true 和 false 分别被转换为 1 和 0
- null，返回 0
- undefined，返回 NaN
- 字符串，数字和浮点数直接转换（其他进制自动转为10进制），`''` 转为0，其他返回 NaN
- Symbol，抛出错误

**Boolean() 方法的强制转换规则**

​	除了 undefined、 null、 false、 ''、 0（包括 +0，-0）、 NaN 转换出来是 false，其他都是 true。



### 隐式类型转换

。。。。





# 数组

## reduce

​	对数组的每一个元素执行一个reducer函数，将其结果汇总成一个单一的值

```js
arr.reduce(reducer, 5)
```



​	reducer函数接收四个参数，第一个是累加器，返回的值会成为累加器，其他三个参数就是`item, index, array`

















