# CSS权威指南

# 第二章 选择符

## 拼接伪类

css是允许将伪类拼接在一起的，如

```less
a:link:hover { color: red }
a:visites:hover { color: blue }
```

## 唯一的子元素

```less
img:only-child // img是它父级的唯一子元素
```

## 第一个子元素

```less
p:first-child
// 选中p，且p是其父级的第一个子元素
// 注意：不是选中p的第一个子元素
```

## 最后一个子元素

```less
p:last-child
```

## 第一个某种元素

```less
table:first-of-type
table:last-of-type // 最后一个某种元素

p:nth-of-type(1)
p:nth-last-of-type(1)
```

## 第n个子元素

```less
p:nth-child(1) // p是第一个子元素

p:nth-last-child(1)
```

## 每第n个子元素

```js
ul > li:nth-child(3n+1) // 1, 4, 7, 10... n从0开始
```

## 每第n个某种元素

```less
p > a:ntn-of-type(2n) // 偶数个
```

## 动态伪类

```less
a:link // 可以用作超链接的，也就是具有href的
a:visited // 被访问过的
a:hover // 鼠标指针放置后
a:active // 鼠标激活后的
input:focus // 获取输入焦点的
```

## 伪元素

使用双冒号与伪类区分，所有伪元素只能出现在选择符的最后

```less
p::first-letter // 首字母
p::first-line // 首行
```

这两个只能用在块级元素上

# 第三章 特指度和层叠

- id                                  - 0 1 0 0
- 类，伪类，属性选择符 - 0 0 1 0
- 元素，伪元素               - 0 0 0 1
- 通配符 连接符              - 0 0 0 0



# JavaScript高级程序设计

# Date

​	`let date = new Date()`，构造函数内应传毫秒数，如果不传就是当前日期和时间，它存的是1970年1月1日0点到此刻的毫秒数。也可以传字符串表示的日期，因为它会默认对字符串做一个Date.parse方法，会将字符串转为毫秒数

```js
let date = new Date()
console.log(date) // Sun Oct 04 2020 10:19:10 GMT+0800 (中国标准时间)

let someDate = new Date("5/23/2019") // 等价于
let someDate = new Date(Date.parse("5/23/2019"))
```

​	通过Date.now()可以获取当前毫秒数，可以用来计算程序执行的时间。

​	需要对date做格式化的话可以用`toLocaleString()`之类的方法，也可以分别获取日期的年月日来自定义，如`getFullYear()`等

# RegExp

`let expression = /patterrn/flags`（模式/标记）

flags的种类：

g - 全局，没有这个的话匹配到第一个就会结束

i - 不区分大小写

m - 表示多行模式

u - 表示unicode模式

y - 只查找从lastIndex开始的

s - 表示匹配任何字符，如\n呀啥的

简单正则

```java
let reg1 = /at/g // 匹配所有的at
  
let reg2 = /[bc]at/i // 匹配bat，cat不区分大小写
  
let reg3 = /.at/gi // 匹配所有以at结尾的，不区分大小写
```

如果要匹配元字符（就是特殊字符）就需转义，用反斜线 \ 

当然除了这种字面量的方式，也可以用构造函数的形式。传两个参数即可，模式和标记

```java
let reg1 = new RegExp("at", "g")

// 还是用字面量吧，这个需要转两次义，如下
let reg2 = /\[bc\]at/g // 匹配所有的[bc]at
let reg3 = new RegExp("\\[bc]\\]at", "g") // 因为要二次转义，其实就是是多加一个反斜杠
```

​	正则实例有两个方法，一个是exec，一个是test，exec方法返回的实例包含index和input两个属性，分别是第一个匹配的下标和被匹配的字符串。如果多次执行index就返回第n个下标（当有g标记时）。

# toFixed

​	会返回数值的小数形式，可以传参数n返回n位小数。并且默认四舍五入

```js
let num = 10
num.toFixed(2) // 10.00
```

# 字符串操作方法

## concat

​	将一个或多个字符串拼接成一个新字符串，不过一般都是用的+来做这个操作

```js
let str1 = "hello"
let str2 = str1.concat(" world", "!")
console.log(str1, str2) // "hello, hello world!"
```

## slice，substr，substring

​	都接收两个参数，第一个是开始位置，第二个是结束的位置，不同的是substr第二个参数是截取的数目，如果不传第二个参数，则都表示到结尾

## indexOf，lastIndexOf

​	都是查找某个字符在字符串的位置，后者是从后往前找，如果传第二个参数，则表示从这个位置开始找

## startsWith，endsWith，includes

​	判断是否包含某个子字符串，1是从0位置开始匹配，2是从最后开始匹配，3是整个字符串是否包含

```js
let message = "foobarbaz"
message.startsWith("bar") // false
message.startsWith("foo") // true
message.endsWith("baz") // true
message.endsWith("bar") // false
```

## trim

消除字符串两头的空格。

## repeat

```js
let str = "abc"
str.repeat(2) // "abcabc"
```

## padStart，padEnd

​	填充字符串

```js
let str = "foo"
str.padStart(1) // "foo"
str.padStart(4) // " foo"
str.padStart(6) // "   foo"
str.padStart(6, ".") // "...foo"
str.padStart(6, "ba") // "babfoo"
```

## toLowerCase，toUpperCase

​	大小写转换

## localeCompare

​	两个字符串排序，参数内的字符串排在前返回1，排在后返回0

# EventLoop

 	JS是一门单线程语言，异步和多线程的实现是靠EventLoop事件循环机制实现的。大体是三个部分组成，调用栈，消息队列和微任务队列。eventloop开始时会在全局栈中一行一行执行，遇到函数会将其压入调用栈中，被压入的函数叫做帧，当函数返回后会弹出调用栈。JS中的异步操作，如fetch，事件回调，setTimeout，setInterval中的回调函数会入队到消息队列中，而消息会在调用栈清空时执行。使用promise，async/await创建的异步操作会加入到微任务队列中，它会在调用栈被清空时立即执行。并且处理期间新加入的微任务也会一同执行。当执行完后会执行消息队列中的任务。

宏任务：

​	script（整体代码），setTimeout，setInterval，setImmediate, I/O, UI rendering

微任务：

​	promise, Object.observe, MutationObserver

任务优先级：



# URL编码

encodeURI，encodeURIComponent，decodeURI，decodeURIComponent

前者不会编码属于url的特殊字符，如# ：/ ？之类，后者会

# eval

`eval("balabala")` 会直接执行balabala

# Math

```js
max() // 最大值
min() //

round() // 四舍五入
ceil() // 向上取整
floor() // 向下取整

random() // 随机数 [ 0，1 )
function selectFrom(a, b) { // 返回 [a, b]内的数，包含边界
  return Math.floor(Math.random() * (b - a + 1) + a)
}
```

# Array

Array.from - 将类数组转为真正的数组

Array.of - 相当于外面包了个中括号

## length妙用

```js
a.length = 2 // 删除下标为1后的元素
a[length] = 2 // 末尾添加元素
```

## 栈和队列方法

push - 添加到数组末尾

pop - 删除最后一项并返回那项

shift - 删除第一项并返回那项

unshift - 添加到数组开头

## 排序方法

reverse - 反向排列

sort - 升序按照字符串排列，如15 < 2, 也可传入回调

```js
// a和b，如果a在前面就返回-1，如果a在后面就返回1，相等就返回0
// 感觉就是需要交换位置就返回1，否则都不交换位置
function compare(a, b) {
	if(a <= b) {
    return -1
  } else {
    return 1
  }
}
// return a-b 
```

## concat

给数组末尾添加数组，不影响原来的数组

## slice，splice

```js
arr.slice(1, 2) // 截取下标1到2点=的部分，不影响原数组，不传第二个参数则截取到末尾
```

splice被称为最强的数组方法，它主要目的是在数组中间插入元素

```js
arr.splice(0, 2, x, y, z) // 从下标0开始删除2个并插入项x，y，z
```

## indexOf, lastIndexOf, includes

前2返回下标，3返回布尔值

## 迭代方法

每个迭代方法都会接收两个参数，第一个是回调函数，第二个是会影响函数中this值的对象

回调函数都接收三个参数，数组元素，下标，当前数组

every - 每一项都返回true，则返回true

some - 有一项返回true，就返回true

filter - 将返回true的元素组成新的数组返回

map - 对每一项做操作后返回操作后的函数

forEach - 对每一项做操作

```js
let array = [1, 2, 3, 4, 5]
// 1.every
let res = array.every((item) => item > 2) // false

// 2.some
let res = array.some((item) => item > 2) // true

// 3.filter
let res = array.filter((item) => item > 2) // [3, 4, 5]
```

## 归并方法

reduce和reduceRight - 迭代数组的所有项，并在此基础上构建一个最终返回值，一个是从前往后，一个是从后往前

接收两个参数，一个归并函数和一个一个初始值

归并函数接收四个参数 - ( 上一个归并值，当前项，下标，当前数组 )

```js
let sum = array.reduce((prev, cur, index, array) => prev + cur) // 15
```



# 第八章 对象、类与面向对象编程

## 属性类型

​	现在创建对象一般就用字面量形式：`let obj = {}`

​	要修改对象的一些特性，就用`Object.defineProperty(要添加属性的对象, 属性名称, 描述符对象)`

```js
let person = {}
Object.defineProperty(person, "name", {
  writable: false, // 只读 - 1
  configurable: false, // 是否可修改它的特性，不可用delete删除
  enumberable: false, // 是否可以通过for-in循环返回
  value: "Nicholas",
  get() {
    return this.name
  },
  set(newVal) {
    this.name = newVal
  }
})

person.name = "xiaoming" // 修改被忽略，不成功
// Object.defineProperties 可以一次定义多个属性
```

## 合并对象

`Object.assign(目标对象, 一个或多个源对象)`

```js
let dest = {}, src = { id: 10 }
let res = Object.assign(dest, src, { class: 'x' }) // { id: 10, class: 'x' }
```

## 创建对象



​	





















