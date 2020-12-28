# JavaScript

## 题目

---

```js
var a = function yideng(num) {
  yideng = num
  console.log(typeof yideng)
  return 1
}
a(1)
console.log(typeof yideng())
```

- yideng是只读的，且只在内部使用。在node中这样使用其实一般就是为了调错的

```js
function
yideng is not defined  
```



---

```js
this.a = 20
var test = {
  a: 40,
  init: function() {
    function go() {
      console.log(this.a)
    }
    go()
  }
}
test.init() // 20
```

- `test.init()` 是 `go()` `go`没人调用, 所以是`window`
- ⚠️ 总结：只要函数执行前面没有点，你就给它加个window



---

```js
this.a = 20
var test = {
  a: 40,
  init: () => {
    console.log(this.a)
  }
}
test.init() // 20
var fn = test.init()
fn() // 20
```

- 箭头函数绑定父级作用域，跟写bind的意思一模一样
- ⚠️ 总结：使用箭头函数就不要管点了





---

new

```js
function Foo() {
    getName = function() {
        console.log(1);
    };
    return this;
}
Foo.getName = function() {
    console.log(2);
};
Foo.prototype.getName = function() {
    console.log(3);
};
var getName = function() {
    console.log(4);
};
function getName() {
    console.log(5);
}
Foo.getName();	// 2
getName();	// 4
Foo().getName();	// 1
getName();	// 1
new Foo.getName(); // 2
new Foo().getName(); // 3
new new Foo().getName(); // 3
```

1. Foo.getName自然是访问构造函数的静态属性，2

   ```js
   function User() {
     this.name = 'a' // 公有属性
     var name = 'a' // 私有属性
     function getName() {} // 私有方法
     name = 'a' // 变为全局变量
   }
   User.name = 'a' // 静态属性
   ```

2. 函数提升，变量提升，4

3. Foo内部的getname没有用var，成为全局方法（执行Foo才这样），Foo没有new，所以this是window，1

4. 看3，1

5. `x = Foo.getName; new x()`

6. `x = new Foo(); x.getName()`

7. `x = new Foo(); y = x.getName;new y()`

## 函数式编程

1. 由React和Redux带火的概念，核心思想是将一个复杂的函数复合成一系列简单的函数，通过函数嵌套的方式调用。

2. 并且崇尚纯函数（不依赖外部状态，没有任何副作用，且固定的输入得到固定的输出）。

3. 纯函数为了避免依赖外部变量会将变量硬编码进函数内部的问题，失去灵活性。可以用柯里化优雅解决。柯里化就是先让一个函数接受一部分参数，返回一个函数来处理剩余的参数。

4. 带来洋葱式的代码问题，可以用函数组合来解决。

   ```js
   const compose = (f1, f2) = x => f1(f2(x))
   ```

5. 组合后用作PointFree，简单说就是不要用中间变量，将处理中间变量的方法抽离成简单的方法，再用函数组合。可以让我们减少不必要的命名，更加清晰，灵活

   ```js
   const f = str => str.toUpperCase().split('')
   
   // 转换
   var toUpperCase = word => word.toUpperCase()
   var split = x => (str => str.split(x))
   
   var f = compose(split(' '), toUpperCase)
   f("abcd xyz")
   ```

6. 函数式编程的一大优点就是可以使用声明式的代码，无需考虑内部如何实现，只需关注编写业务代码















