`npm install -g typescript`

带有类型的JavaScript超集

100%兼容js代码，也就意味着能兼容第三方的库和框架，所有js代码，对象，库可用于ts

强大的类型系统，静态类型检查能力

丰富的class拓展功能



类型系统的好处

编码阶段，或者说编译阶段把一些可能会出现的隐式问题给排查出来，避免这个东西写完以后在线上报错了



# 数组，元组，枚举，Void，Never，any

数组中存储的类型必须一致，元组类似数组，但类型不必一致

```js
let data = [string, number] = ['1', 2]
```



枚举的作用是组织收集一组关联数据的方式，通过枚举可以给一组有关联意义的数据赋予一些友好的名字

```js
enum HTTP_CODE {
  OK: 200,
  NOT_FOUND: 404
}
HTTP_CODE.OK
// 枚举是只读属性, 初始化后不得修改
```



never表示当一个函数永远不可能执行return的时候，返回的就是never，比如抛出一个错误





# 函数类型标注

```js
function fn(a: number, b:number):number {
  return a + b
}
fn(1, 2)
```



# type

声明一个类型

```js
type F = (a:number, b:number) => number
let fn: F = function(a: number, b:number):number {
  return a + b
}

function move(callback: F) {
  ...
}
move(fn)
```



# 泛型



# 接口 - 类

接口只能作为类型标注去使用，不能作为具体值，它只是一种抽象的结构定义，并不是实体，没有具体功能实现。

我的理解就是接口就是一种自定义类型。

类的化即可定义类型，也可以去用，

```js
let p: Person = new Person()
```







# 装饰器

​	对类进行扩展带来更大的便利性，本身对类拓展的化一个是使用继承，一个是接口。

​	现在可以使用装饰器来进行扩展，**对类进行扩展的一种手段**。在ts中，装饰器只能在类里使用。



​	装饰器本质是一个函数，它可以通过@方法附加在类，方法，访问符，属性，参数上，对它们进行包装，然后返回一个包装后的目标对象，装饰器工作在类的构建阶段，而不是使用阶段

```js
// 装饰者模式 - 一种设计模式
function wrap(fn) {
  return function() {
    console.log('日志')
    fn()
  }
}

let dispatch = wrap(libs.dispatch)
dispatch()
```

用在TS上

```js
function log(target: any, name: string, descriptor) { // #1
  let fn = descriptor.value // #2
  descriptor.value = function(a:number, b:number) {
    fn(a, b)
  }
}

class Store {
  @log
  dispatch() {
    ...
  }
}
```

1. 函数装饰器必须要这种格式
2. 这个value就是被装饰的方法



如果是类装饰器的话只要一个参数，也就是该类的构造函数

```js
function d(target: Function) {
  console.log(target)
}

@d
class M {
  ...
}
```







# 模块





# React使用

```js
interface PropTypes {
  name: string;
  age?: number; // #1
}
interface StateTypes {
  val: number;
}

export default class Kaike extends Component<PropTypes, StateTypes> { // #2
  constructor(props: PropTypes) {
    super(props)
    this.state = { // #3
      val: 1
    }
  }
}
```

1. 表示可选的
2. 这两个就是用来定义props和state的类型的
3. 因为约束好了就不能想怎么写就怎么写





# tsconfig.json

```js
{
  "compileOPtions": {
    outDir // 编译输出路径
    target // 使用哪种js
    module // 配置使用哪种模块化方式
  }
}
// 打包后会编译成对应配置下的文件类型

```





































































































