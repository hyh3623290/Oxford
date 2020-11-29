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

​	网络资源 - 浏览器 - 前端界面 

​	浏览器从上到下首先是用户界面，它主要负责把命令转交给“幕后”的浏览器引擎。浏览器引擎虽然也无法独自完成这些命令，但能“读懂”你的命令，进而把这个命令用渲染引擎能理解的方式传递给它。渲染引擎不仅自己有一套任务逻辑，它还能够调度一系列执行层面的工作模块。看似毫无生机的网络资源，经过了渲染引擎的“魔术手”之后，便能够摇身一变、化身为色彩斑斓可交互的页面呈现在你面前了。--再后面，人们干脆直接把浏览器内核和渲染引擎划上了等号。

## 渲染引擎工作原理

​	中间这个“渲染引擎处理”目前对大家来说仍然是一个黑盒。我们把这个黑盒拆开，会看到它其实包含了以下几个具体流程：资源文件  ->  HTML解析  ->  CSS解析  ->  样式与结构合并  ->  布局阶段  ->  页面绘制

### HTML解析 —— DOM 树

​	浏览器不能够直接理解 HTML，它首先会把它转化成自己能理解的 DOM 树

### CSS 解析 —— CSSOM 树

​	虽然我们编写 CSS 代码时不像 HTML 代码一样表现出明显的嵌套关系，但是CSSOM 也是具有树结构的。这是因为在样式计算的过程中，浏览器总是从适用于该节点的最通用规则开始（例如 div 节点是 body 元素的子项，则应用所有 body 样式），一层一层递归细化出具体的样式。

### DOM树与CSSOM树“合体” ——渲染树

​	值得注意的是，渲染树的特点是它只包含渲染网页所需的节点。所以在构建渲染树的过程中，除了“结合”之外，浏览器还做了一些关键的小动作，这些小动作可能作为考点出现：

- step1： 从 DOM 树的根节点开始遍历，筛选出所有**可见**的节点；
- step2：仅针对可见节点，为其匹配 CSSOM 中的 CSS 规则；
- step3：发射可见节点（连同其内容和计算的样式）。

### 布局盒子模型

​	我们已经掌握了需要渲染的所有节点之间的结构关系及其样式信息。但是我们还不知道它们在渲染时，到底应该出现在浏览器视口的哪个位置上、占据多大的空间——计算这些信息，就是布局阶段要做的事情。浏览器对渲染树进行遍历，将元素间嵌套关系以盒子模型的形式写入文档流

### 目标界面

​	布局阶段结束后，浏览器终于拿到了它绘制页面所需要的所有信息。此时它会将渲染树上的每一个节点转换成我们肉眼可见的像素，最终将页面呈现在我们面前，这个过程就是“绘制”。绘制完成后，目标界面也就在你眼前了



## 重排，重绘和图层

​	当我们的操作引发了 DOM 几何尺寸的变化（比如修改元素的宽、高或隐藏元素等）时，浏览器需要重新计算元素的几何属性（其他元素的几何属性和位置也会因此受到影响），然后再将计算的结果绘制出来。这个过程就是**重排**

​	当我们对 DOM 的修改导致了样式的变化、却并未影响其几何属性（比如修改了颜色或背景色）时，浏览器不需重新计算元素的几何属性、直接为该元素绘制新的样式（跳过了上图所示的布局环节）。这个过程叫做**重绘**

**还有什么动作会触发重排**

- 改变 DOM 树的结构，节点的增减、移动等
- 获取一些特定属性的值（为啥？）

​    如offsetTop、offsetLeft、 offsetWidth、offsetHeight、scrollTop、scrollLeft、scrollWidth、scrollHeight、clientTop、clientLeft、clientWidth、clientHeight 等属性（挑几个背下来，答题的时候不要哑了）。

​	这些属性有一个共性，就是需要通过即时计算得到。因此浏览器为了获取这些值，也会进行回流。

​	除此之外，当我们调用了 getComputedStyle 方法，或者 IE 里的 currentStyle 时，也会触发回流。原理是一样的，都为求一个“即时性”和“准确性”。



## EventLoop

​	在浏览器的事件循环中，首先大家要认清楚 3 个角色：**函数调用栈**、**宏任务（macro-task)队列**和**微任务(micro-task)队列**。

​	常见的 macro-task 比如： setTimeout、setInterval、 setImmediate、 script（整体代码）、I/O 操作等。

​	常见的 micro-task 比如: process.nextTick、Promise、MutationObserver 等

​	注意：script（整体代码）它也是一个宏任务；此外，宏任务中的 setImmediate、微任务中的 process.nextTick 这些都是 Node 独有的

​	任务顺序是先一个宏任务，然后一队微任务

## Node中的EventLoop

​	浏览器的 Event-Loop 由各个浏览器自己实现；而 Node 的 Event-Loop 由 libuv 来实现。

28章











# ---