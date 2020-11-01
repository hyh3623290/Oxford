# JS面向对象



抽象，继承，封装，多态



## 创建对象的三种方式



1. 字面量
2. 构造函数

```js
let obj = new Object(); obj.xxx = xxx
```

1. Object.create()

```js
// 属性方法会放在原型上 __proto__
Object.create({
  name: '小明',
  hobby: function() {
    console.log('football')
  }
})
```



## 工厂模式

最常用的设计模式之一，属于创建兴模式，提供一种创建对象的最佳方式。但是js很少用工厂模式

```js
let lisi = {
  name: 'lisi',
  age: 10,
  balabala
}
// 多创建几个的话非常冗余 => 工厂模式: 类
function Person(name, age) {
  let obj = {}
  obj.name = name
  obj.age = age
  return obj
}
let lisi = new Person('lisi', 10)
```



## new运算符



1. 首先会执行函数

```js
let str = new String(); // 比方

function test() {
  console.log('test')
}
test() // 打印'test'
new test() // 打印'test'
new test // 打印'test'
```

1. 自动（隐式）创建一个空对象
2. 把这个空对象和函数的this绑定，也就是this === 这个空对象，把加在this上的属性和方法都加在该对象上
3. 如果没有返回，它会隐式返回this



看上面的工厂函数是不是也是新创建一个空对象，最后返回新对象了呢，所以如果要用new的话其实可以这样

```js
function Person(name, age) {
  this.name = name
  this.age = age
}
let lisi = new Person('lisi', 10)
```



## 构造函数



​	其实就是我们的工厂模式。1. 首字母要大写	2. this指向实例化对象

​	在构造函数中，不要使用return，因为return基本类型会被实例化对象覆盖，return复杂类型会覆盖实例化对象，也即没有构造函数的意义了。

## class

​	为了帮我们区分区分构造函数（类）和普通函数，es6增加了class。但是有别于传统语言的类（Java），这里的类只是一个语法糖

​	面向对象的特征：抽象，继承，封装，多态

- 抽象：抽取相似的功能，归为一类

```js
class Person {
  constructor(name, age) {// 就是之前写的构造函数
    this.name = name
    this.age = age
    this.sayName = function() {
      console.log(this.name)
    }
  }
  // 如果sayName写在这里 #1
  sayName() {
    console.log(this.name)
  }
}
```

1. 如果打印这个类的实例化对象会发现这个sayName方法会加在对象的`__proto__`下

class里的类里面要使用`useStrict`严格模式



## 静态属性和方法



​	这个是属于类本身的方法，而不是实例化对象的方法。有时候我们不需要把一些属性放到实例化对象上面，比如我想统计一下这个类被创建了几次，就用`Person.name`



## 构造函数的性能问题

​	每个实例化对象都有一个方法，如果这个方法做的是同一件事，就会比较浪费。类似于我们家买了一把菜刀，大家都用这个就好，如果每个人买了一把是不是就有点浪费呢。

```js
let zs = new Person("张三")
let ls = new Person("李四")// 假设都有个hobby方法做了同一件事
zs.hobby === ls.hobby // false, 因为函数会重新开辟一块内存
```

​	那么假设有100个人，那么都会在内存开辟100个新的地址，会比较消耗内存，性能。所以有公共的空间帮我们存放相同的方法，可以更好的节约我们的内存，在性能这块也会得到很大的提升 。也就是我们的原型。所有的实例化对象调用的就是一个方法，而不在分别去占用内存空间生成新的方法。



## 原型

​	在JS中，每个函数都有一个属性叫`prototype`，它本身是一个对象，在该对象中我们可以定义一些该函数的实例化对象的公用方法。

​	在JS中，每个对象都有一个属性叫`__proto__`，指向该对象构造函数的原型



​	每个构造函数在实例化的过程中，都会有两部分构成，一部分也就是我们的构造函数，另外一部分就会提供一个公共空间：原型。`Person.prorotype`

​	实际上这个公共空间就是原型。原型里的this也是指向我们的实例化对象

​	实际上实例化的对象也会有两部分构成，一部分是它自己的属性，另一部分就是也是原型，不过叫`__proto__`

```js
Person.prototype.fn = function() {
  console.log('fn')
}
let zs = new Person()
// 实际上就会看到: zs.__proto__里就有fn
```



`__proto__`和`prototype`是什么关系呢，实际上就是一个东西，只不过表现形式不一样



```js
zs.__proto__ === Person.prototype // true
```



此外：原型上有个固有属性指向构造函数



```js
Person.prototype.constructor === Person
```



也就是`zs.constructor = Person`，这个constructor主要是让我们知道他是那个构造函数的实例

**拓展：**`__proto__`不是规范属性，只是部分浏览器实现了此属性，对应的标准属性应该是`[[Prototype]]`



## 继承

​	继承其实只有三种，继承构造函数，继承原型，一种是拷贝继承，其他都是炒出来的概念，本质都是这三种。如果再网上搜，可以搜出七八种

**如何继承父类的构造函数**

```js
function Child(name, age) {
  Parent.call(this, name, age)
}
```

**如何继承父类的原型上的东西呢**

写法有很多，如

```js
Child.prototype = new Person
```

​	构造函数的原型是一个对象，这时我们把Person实例放在这里Person的实例对象就可以拿到Person的prototype的所有方法。但是涉及到一个问题，在new Person的时候也会执行父类的构造函数，也会生成对应的父类的属性，只不过没有被赋值而已。所以我们通常会做一个包装

```js
function extend(Parent) {
  function F(){}
  F.prototype = Parent.prototype
  return new F
}
Child.prototype = extend(Parent)
Child.prototype.constructor = Child
```

这样就只有原型中的内容，没有实例中的内容。

**拷贝继承**

```js
Child.prototype = Person.prototype
// 对象是传址的， 这样修改子类时会影响到父类
```





call和apply会执行这个函数，bind是返回一个新的函数，不会自动执行

```js
function Person(name) {
  this.name = name
}
function Programmer(name) {
  Person.call(this, name) // 执行父构造函数并绑定子类的this, 继承父类, 帮我们少写很多代码
  // Person.apply(this, [name])
  // Person.bind(this)(name)
  this.job = 'web engineer'
}
```



​	但是我们一般把方法之类的写到`prototype`上，call的话无法继承`prototype`里的属性，



​	`Son.prototype = Parent.prototype` 很容易想到这样的方法，但是这样是不行的，因为这样当修改子类的原型上的方法的时候也会影响父类的原型。复杂数据类型都会涉及到传址问题



**扩展**	深拷贝方法`JSON.parse(JSON.stringify(obj))`的不足，obj会丢失掉函数，还有值为undefined的属性



​	所以我们可以深拷贝：`Son.prototype = deepClone(Parent.prototype)`，此外还有组合继承方法



## 组合继承

```js
let Link = function() {}
Link.prototype = Parent.prototype
Son.prototype = new Link() // 通过实例化开辟一块新的地址
Son.prototype.constructor = Son
// 这样原型之间也不会相互影响
```



## ES6继承

```js
class Engineer extends Person {
  constructor(name, job) {
	super(name) // #1
  }
  sayJob() {
    console.log('job')
  }
}
```

1. super就会执行父类的构造函数，并把子类的参数传给父级
2. `Engineer => __proto__是constructor: class Person,sayJob: f(),有sayJob方法;`
3. ` 再__proto__是constructor: class Object,sayName: f()是父类的原型上的方法`



## this

1. 箭头函数内没有`this`，其`this`继承声明作用域时所在`this`（ - 找箭头里的`this`就看花括号外面的`this`）
2. `function`里的`this`是谁调用指谁
   - 直接调用（`window`）
   - 对象的属性调用（该对象）
3. 类中的`this`指向其实例化对象



## 基于观察者模式实现Event类

​	观察者模式：当对象间存在一对多的关系时，则使用观察者模式。比如，当一个对象被修改时，则会自动通知它的依赖对象。观察者模式属于行为型模式

## 静态方法与属性

`Array.isArray()`

```js
class Person {
  constructor {
 	...
  }
  static 物种 = "哺乳类"
  static eat() {
     ...
  }
}

Person.物种
Person.eat()

```



# 拖拽

```js
// <div id="box"></div>

let box = document.querySelector("#box")
let startMouse = {}
let startEl = {}
let move = (e) => {
  let nowMouse = {// #5
    x: e.clientX,
    y: e.clientY
  }
  let disMouse = {// #6
    x: nowMouse.x - startMouse.x,
    y: nowMouse.y - startMouse.y
  }
  css(box, "left", startEl.x + disMouse.x) // #7
  css(box, "top", startEl.y + disMouse.y)
}
box.addEventListener("mousedown", (e) => {
  startMouse = { // #1
    x: e.clientX,
    y: e.clientY
  }
  startEl = { // #2
    x: css(box, "left"),
    y: css(box, "top")
  }
  document.addEventListener("mousemove", move) // #3
  document.addEventListener("mouseup", () => { // #4
    document.removeEventListener("mousemove", move)
  }, {once: true})
  e.preventDefault()
})

function css(el, attr, val) {
  if(val === undefined) {
    return parseFloat(getComputedStyle(el)[attr])
  }
  el.style[attr] = val + 'px'
}
```

1. 记录鼠标按下时鼠标的位置
2. 记录鼠标按下时元素的位置
3. 鼠标按下时添加move事件
4. 抬起时取消move事件
5. move事件：记录当前鼠标的位置
6. move事件：记录鼠标移动的距离 - 当前位置 - 按下时的位置
7. move事件：设置元素的新位置
8. 一般加给document，不然加给box的话移动太快的时候鼠标会飞出去



**可以看出来已经有点费劲了，如果想拖拽两个div呢？**

​	上面是典型的面向过程的写法。面向对象：`let boxDrag = new DragEl(box)`，就可以实现拖拽了



**考虑写成一个类**

 ## 抽象

拖动之后元素跟着移动

## 封装

封装成类

```js
class Drag {
    constructor(el){
        this.el = el;
        this.startMouse = {};
        this.startEl = {};
        this.start();
    }
    // 摁下时候要做的事情
    start(){
        let move = (e)=>{
            // e:event
            this.move(e);
        };
       this.el.addEventListener("mousedown",(e)=>{
            //    console.log(this);// 实例化对象
            //    console.log(this.el);
            // 记录摁下时鼠标的位置
            this.startMouse = {
                x: e.clientX,
                y: e.clientY
            };
            this.startEl= this.getOffset();
            document.addEventListener("mousemove",this.move);
            document.addEventListener("mouseup",()=>{
                document.removeEventListener("mousemove",move);
            },{once: true});
       }); 
    }
    // 移动中要做的事情
    move(e){
        let nowMouse = {
            x: e.clientX,
            y: e.clientY
        };
        let disMouse = {
            x: nowMouse.x - this.startMouse.x,
            y: nowMouse.y - this.startMouse.y
        };
        this.setOffset(disMouse);
        this.el.style.left = dis.x + this.startEl.x + "px";
        this.el.style.top = dis.y + this.startEl.y + "px"
    }
    // 获取元素的位置
    getOffset(){
        return {
            x: parseFloat(getComputedStyle(this.el)["left"]),
            y: parseFloat(getComputedStyle(this.el)["top"])
        }
    }
    // 设置元素的位置
    setOffset(dis){
        this.el.style.left = dis.x + this.startEl.x + "px";
        this.el.style.top = dis.y + this.startEl.y + "px"
    }
}
{
    let box = document.querySelector("#box");
    let d = new Drag(box);
}
```

## 多态

同一操作作用于不同的对象，会产生不同的结果

```js
let p1 = new Person()
let p2 = new Teacher()
p1.say()
p2.say()
```









