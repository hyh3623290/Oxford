# DOM

​	DOM实则只是把 HTML 中各个标签定义出的元素以对象的形式包装起来。这样做的目的，就是确保开发者可以通过 JS 脚本来操作 HTML。

## 节点

**document**

​	Document 就是指这份文件，也就是这份 HTML 档的开端。当浏览器载入 HTML 文档, 它就会成为 **Document 对象**。

**element**

​	像是`<div>、<span>`这种标签都属于element类型

**Text**

​	就是标签里的文字

**Attribute**

​	元素的特性，也就是各个标签里的属性。

## 操作

​	无非是增删改查

**增**

```js
// 创建新节点
var targetSpan = document.createElement('span')
// 设置 span 节点的内容
targetSpan.innerHTML = 'hello world'

// 把新创建的元素塞进父节点container里去
container.appendChild(targetSpan)
```

**删**

```js
container.removeChild(targetNode)

container.removeChild(container.childNodes[1]) 
```

**改**

```js
// 获取 id 属性
var titleId = title.getAttribute('id')

// 修改 id 属性
title.setAttribute('id', 'anothorTitle')
```

**查**

```js
// 按照 id 查询
var imooc = document.getElementById('imooc')

// 按照标签名查询
var pList = document.getElementsByTagName('p')
console.log(divList.length)
console.log(divList[0])

// 按照类名查询
var moocList = document.getElementsByClassName('mooc')

// 按照 css 选择器查询
var pList = document.querySelectorAll('.mooc')
```



## 事件流

- 事件流：它描述的是事件在页面中传播的顺序

- 事件：它描述的是发生在浏览器里的动作。这个动作可以是用户触发的，也可以是浏览器触发的。像点击（click）、鼠标悬停（mouseover）、鼠标移走（mousemove）这些都是事件。

- 事件监听函数：事件发生后，浏览器如何响应——用来应答事件的函数，就是事件监听函数，也叫事件处理程序。

​    早期 - IE冒泡流，NetScape只认捕获流 -  W3C 介入 - JS 同时支持了冒泡流和捕获流 - 标准 - 这个标准也叫做“DOM2事件流”。W3C 标准约定了一个事件的传播过程要经过以下三个阶段：

1. 事件捕获阶段
2. 目标阶段
3. 事件冒泡阶段



## 事件对象

​	 当 DOM 接受了一个事件、对应的事件处理函数被触发时，就会产生一个事件对象 event 作为处理函数的入参。这个对象中囊括了与事件有关的信息，比如事件具体是由哪个元素所触发、事件的类型等等。事件对象中，有一些属性和方法，是我们特别常用的

**1. currentTarget**

​	事件当下被哪个元素接收（元素是一直在改变的）

**2. target**

​	指触发事件的具体目标，也就是最具体的那个元素，是事件的真正来源。就算事件处理程序没有绑定在目标元素上、而是绑定在了目标元素的父元素上，只要它是由内部的目标元素冒泡到父容器上触发的，那么我们仍然可以通过 target 来感知到目标元素才是事件真实的来源。

**3. preventDefault** 

​	这个方法用于阻止特定事件的默认行为，如 `a` 标签的跳转等。

**4. stopPropagation**

​	这个方法用于终止事件在传播过程的捕获、目标处理或起泡阶段进一步传播。调用该方法后，该节点上处理该事件的处理程序将被调用，事件不再被分派到其他节点。	



## 自定义事件

​	事件对象不一定需要你通过触发某个具体的事件来让它“自然发生”，它也可以手动创建的：事件对象的这个特性，是我们创建自定义事件的基础，它确实非常重要。

```js
 event = new Event(typeArg, eventInit);
```

​	假设这种场景，如果我现在想实现这样一种效果：在点击A之后，B 和 C 都能感知到 A 被点击了，并且做出相应的行为

```html
<div id="divA">我是A</div>
<div id="divB">我是B</div>
<div id="divC">我是C</div>
```

​	在早年，最经典的解法就是——自定义事件。“A被点击了”这件事情，可以作为一个事件来派发出去，由 B 和 C 来监听这个事件，并执行各自身上安装的对应的处理函数。

```js
var clickA = new Event('clickA');

divB.addEventListener('clickA', function(e){
  console.log('我是小B，我感觉到了小A')
  console.log(e.target)
}) 

// A 元素的监听函数也得改造下
divA.addEventListener('click', function(){
  console.log('我是小A')
  // 注意这里 dispatch 这个动作，就是我们自己派发事件了
  divB.dispatchEvent(clickAEvent)
}) 
```



## 事件代理

​	事件代理，又叫事件委托。经典问题，一个`ul`包10个`li`。点击任何一个 li，是不是这个点击事件都会被冒到它们共同的爹地—— ul 元素上去？那我能不能让 ul 来做这个事儿？答案是能，因为 ul 不仅能感知到这个冒上来的事件，它还可以通过 e.target 拿到实际触发事件的那个元素，做到无缝代理



## 防抖节流

​	过度触发的事件，比如scroll 事件之类。throttle（事件节流）和 debounce（事件防抖）

**1. 节流**

​	在某段时间内，不管你触发了多少次回调，我都只认第一次，并在计时结束时给予响应。所谓的“节流”，是通过在一段时间内无视后来产生的回调请求来实现的。

​	每当用户触发了一次 scroll 事件，我们就为这个触发操作开启计时器。一段时间内，后续所有的 scroll 事件都会被当作“一辆车的乘客”——它们无法触发新的 scroll 回调。直到“一段时间”到了，第一次触发的 scroll 事件对应的回调才会执行，而“一段时间内”触发的后续的 scroll 回调都会被节流阀无视掉。

```js
// fn是我们需要包装的事件回调, interval是时间间隔的阈值
function throttle(fn, interval) {
  let last = 0
  return function () {
    let context = this
    let args = arguments
    let now = new Date()
    if (now - last >= interval) {
      last = now;
      fn.apply(context, args);
    }
  }
}

const better_scroll = throttle(() => console.log('触发了滚动事件'), 1000)
document.addEventListener('scroll', better_scroll)
```



**2. 防抖**

```js
// fn是我们需要包装的事件回调, delay是每次推迟执行的等待时间
function debounce(fn, delay) {
  let timer = null
  return function () {
    let context = this
    let args = arguments
    if(timer) {
        clearTimeout(timer)
    }
    timer = setTimeout(function () {
      fn.apply(context, args)
    }, delay)
  }
}

const better_scroll = debounce(() => console.log('触发了滚动事件'), 1000)
document.addEventListener('scroll', better_scroll)
```



# 浏览器

























# ---