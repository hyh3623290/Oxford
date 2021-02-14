[toc]

# Virtual DOM

​	babel会帮我们把jsx编译成`React.createElement`

# createElement

## 功能

​	帮我们返回虚拟dom对象

1. 返回虚拟dom格式
2. 刨除元素（`null, true, false`）
3. 处理文本元素格式

```js
function createElement (type, props, ...children) {
	// 由于 map 方法无法从数据中刨除元素, 所以此处将 map 方法更改为 reduce 方法
  const childElements = [].concat(...children).reduce((result, child) => {
    // 判断子元素类型 刨除 null true false
    if (child != null && child != false && child != true) {
      if (child instanceof Object) {
        result.push(child)
      } else {
        result.push(createElement("text", { textContent: child }))
      }
    }
    return result
  }, [])  
  return {
    type,
    props: Object.assign({ children: childElements }, props),
    children: childElements
  }
}
```

⚠️ 学到一个技巧

```js
function x(a, ...arr) { console.log(...arr) } 
x(1, [1,2,3]) // [1,2,3] 如果console没有...的话输出[[1,2,3]]
// ...参数适用于剩余多个参数的情况，没有...的话就只取剩余参数的第一个
```



```js
比如某个文件表示一个小功能，我们可以把它们都导入到index.js，再统一导出，比如
// index.js
import fn1 from './A.js'
import fn2 from './B.js'

export default {
  fn1,
  fn2
}
这样只需导入index.js即可，显得比较整齐
```



# render

## 功能

​	`ReactDOM.render(element, container, callback)	`

​	`react-dom.js`提供了dom渲染的方法

​	通过调用 render 方法可以将 Virtual DOM 对象更新为真实 DOM 对象。

```js
function render(virtualDOM, container, oldDOM = container.firstChild) {
  // 在 diff 方法内部判断是否需要对比 对比也好 不对比也好 都在 diff 方法中进行操作  
  diff(virtualDOM, container, oldDOM)
}
```



## diff 算法

```js
function diff(virtualDOM, container, oldDOM) {
  if (!oldDOM) {
    mountElement(virtualDOM, container)
  }
}
```



### mountElement

```js
function mountElement(virtualDOM, container) {
  if(isFunction(virtualDOM)) {
    mountComponent(virtualDOM, container)
  } else {
    mountNativeElement(virtualDOM, container)
  }
}

function isFunction(virtualDOM) {
  return typeof virtualDOM.type === 'function'
}
```



#### mountNativeElement

- 生成DOM并插入container

```js
function mountNativeElement(virtualDOM, container) {
  const newElement = createDOMElement(virtualDOM)
  container.appendChild(newElement)
}
```



##### createDOMElement

1. 判断vdom节点类型，文本和非文本处理方式不同
2. 非文本要更新属性
3. 记得遍历其children

```js
export default function createDOMElement(virtualDOM) {
  let newElement = null
  if (virtualDOM.type === "text") {
    newElement = document.createTextNode(virtualDOM.props.textContent)
  } else {
    newElement = document.createElement(virtualDOM.type)
    updateElementNode(newElement, virtualDOM)
  }
  // 递归渲染子节点
  virtualDOM.children.forEach(child => {
    // 因为不确定子元素是 NativeElement 还是 Component 所以调用 mountElement 方法进行确定
    mountElement(child, newElement)
  })
  return newElement
}
```



###### updateElementNode

1. 通过props获取属性
2. 根据不同的属性做不同的处理（`onClick, value,checked, className, children`）

```js
export default function updateElementNode(element, virtualDOM) {
  const newProps = virtualDOM.props
  Object.keys(newProps).forEach(propName => {
    const newPropsValue = newProps[propName]
    // onClick -> click
    if (propName.slice(0, 2) === "on") {
      const eventName = propName.toLowerCase().slice(2)
      element.addEventListener(eventName, newPropsValue)
      // 如果属性名称是 value 或者 checked 需要通过 [] 的形式添加
    } else if (propName === "value" || propName === "checked") {
      element[propName] = newPropsValue
      // 刨除 children 因为它是子元素 不是属性
    } else if (propName !== "children") {
      // className 属性单独处理 不直接在元素上添加 class 属性是因为 class 是 JavaScript 中的关键字
      if (propName === "className") {
        element.setAttribute("class", newPropsValue)
      } else {
        // 普通属性
        element.setAttribute(propName, newPropsValue)
      }
    }
  })
}
```



#### mountComponent

1. 判断是否是类组件（原型是否有render）
2. 函数 -> 执行，类 -> 实例化
3. 它们返回的也是jsx，所以继续处理jsx就是前面的内容

```js
// 不管是类组件还是函数组件，返回的依然是jsx
export default function mountComponent(virtualDOM, container) {
  let nextVirtualDOM = null
  if (isFunctionalComponent(virtualDOM)) {
    nextVirtualDOM = buildFunctionalComponent(virtualDOM)
  } else {
    nextVirtualDOM = buildClassComponent(virtualDOM)
  }
  if (isFunction(nextVirtualDOM)) {
    mountComponent(nextVirtualDOM, container)
  } else {
    mountNativeElement(nextVirtualDOM, container)
  }
}

// Virtual DOM 是否为函数型组件
// 只有类组件的原型对象中有render方法 
export function isFunctionalComponent(virtualDOM) {
  const type = virtualDOM && virtualDOM.type
  return (
    type && isFunction(virtualDOM) && !(type.prototype && type.prototype.render)
  )
}

function buildFunctionalComponent(virtualDOM) {
  // props 属性传递给组件函数, 这样在组件中就可以通过 props 属性获取数据了
  return virtualDOM && virtualDOM.type(virtualDOM.props || {})
}

function buildClassComponent(virtualDOM) {
  const component = new virtualDOM.type()
  const nextVirtualDOM = component.render()
  return nextVirtualDOM
}
```





# Fiber

## 问题

​	在现有React中，更新过程是同步的，这可能会导致性能问题。

​	当React决定要加载或者更新组件树时，会做很多事，比如

- 调用各个组件的生命周期函数
- 计算和比对Virtual DOM（采用循环加递归）
- 最后更新DOM树

​    这整个过程是同步进行的，也就是说只要一个加载或者更新过程开始，那React就以不破楼兰终不还的气概，一鼓作气运行到底，中途绝不停歇。当组件树比较庞大的时候，问题就来了。

​	假如更新一个组件需要1毫秒，如果有200个组件要更新，那就需要200毫秒，在这200毫秒的更新过程中，浏览器那个唯一的主线程都在专心运行更新操作，无暇去做任何其他的事情。想象一下，在这200毫秒内，用户往一个input元素中输入点什么，敲击键盘也不会获得响应，因为渲染输入按键结果也是浏览器主线程的工作，但是浏览器主线程被React占着呢，抽不出空，最后的结果就是用户敲了按键看不到反应，等React更新过程结束之后，咔咔咔那些按键一下子出现在input元素里了。

​	这就是所谓的界面卡顿，很不好的用户体验。

## React Fiber的方式

​	破解JavaScript中同步操作时间过长的方法其实很简单——分片。

​	把一个耗时长的任务分成很多小片，每一个小片的运行时间很短，虽然总时间依然很长，但是在每个小片执行完之后，都给其他任务一个执行的机会，这样唯一的线程就不会被独占，其他任务依然有运行的机会。

​	React Fiber把更新过程碎片化，每执行完一段更新过程，就把控制权交还给React负责任务协调的模块，看看有没有其他紧急任务要做，如果没有就继续去更新，如果有紧急任务，那就去做紧急任务。	

​	维护每一个分片的数据结构，就是Fiber。

## 为什么叫Fiber呢？

​	大家应该都清楚进程（Process）和线程（Thread）的概念，在计算机科学中还有一个概念叫做Fiber，英文含义就是“纤维”，意指比Thread更细的线，也就是比线程(Thread)控制得更精密的并发处理机制。

​	我想说的是，其实对大部分React使用者来说，也不用深究React Fiber是如何实现的，除非实现方式真的对我们的使用方式有影响，我们也不用要学会包子怎么做的才吃包子对不对？

​	但是，React Fiber的实现改变还真的让我们的代码方式要做一些调整。



## React Fiber对现有代码的影响

​	理想情况下，React Fiber应该完全不影响现有代码，但可惜并完全是这样，要吃这个包子还真要知道一点这个包子怎么做的，你如果不喜欢吃甜的就不要吃糖包子，对不对？

​	在React Fiber中，一次更新过程会分成多个分片完成，所以完全有可能一个更新任务还没有完成，就被另一个更高优先级的更新过程打断，这时候，优先级高的更新任务会优先处理完，而低优先级更新任务所做的工作则会完全作废，然后等待机会重头再来。

​	因为一个更新过程可能被打断，所以React Fiber一个更新过程被分为两个阶段(Phase)：第一个阶段Reconciliation Phase和第二阶段Commit Phase。

​	在第一阶段Reconciliation Phase，React Fiber会找出需要更新哪些DOM，这个阶段是可以被打断的；但是到了第二阶段Commit Phase，那就一鼓作气把DOM更新完，绝不会被打断。（也是因为第一阶段已经找到了最小化操作dom的方式）

​	这两个阶段大部分工作都是React Fiber做，和我们相关的也就是生命周期函数。

​	以render函数为界，第一阶段可能会调用下面这些生命周期函数，说是“可能会调用”是因为不同生命周期调用的函数不同。

- componentWillMount
- componentWillReceiveProps
- shouldComponentUpdate
- componentWillUpdate

​    下面这些生命周期函数则会在第二阶段调用。

- componentDidMount
- componentDidUpdate
- componentWillUnmount

​    因为第一阶段的过程会被打断而且“重头再来”，就会造成意想不到的情况。

​	比如说，一个低优先级的任务A正在执行，已经调用了某个组件的componentWillUpdate函数，接下来发现自己的时间分片已经用完了，于是冒出水面，看看有没有紧急任务，哎呀，真的有一个紧急任务B，接下来React Fiber就会去执行这个紧急任务B，任务A虽然进行了一半，但是没办法，只能完全放弃，等到任务B全搞定之后，任务A重头来一遍，注意，是重头来一遍，不是从刚才中段的部分开始，也就是说，componentWillUpdate函数会被再调用一次。

​	在现有的React中，每个生命周期函数在一个加载或者更新过程中绝对只会被调用一次；在React Fiber中，不再是这样了，第一阶段中的生命周期函数在一次加载和更新过程中可能会被多次调用

​	使用React Fiber之后，一定要检查一下第一阶段相关的这些生命周期函数，看看有没有逻辑是假设在一个更新过程中只调用一次，有的话就要改了。

​	我们挨个看一看这些可能被重复调用的函数。

1. componentWillReceiveProps，即使当前组件不更新，只要父组件更新也会引发这个函数被调用，所以多调用几次没啥，通过！
2. shouldComponentUpdate，这函数的作用就是返回一个true或者false，不应该有任何副作用，多调用几次也无妨，通过！
3. render，应该是纯函数，多调用几次无妨，通过！

​    只剩下componentWillMount和componentWillUpdate这两个函数往往包含副作用，所以当使用React Fiber的时候一定要重点看这两个函数的实现。

⚠️【总结】

​	如果一个组件本身更新它就很耗时，那也没有办法，用fiber打断它完全不会对总的耗时有什么大的影响。但是fiber还可以帮助你及时响应用户操作，以及不影响动画等优先级更高的东西。

​	此外，使用了fiber会对原来的一些操作产生影响，这也是学习fiber的原因，以及了解它原理的原因



## requestIdleCallback

​	当浏览器空闲的时候会执行这个函数。挂在window下面，我们可以通过获取空闲时间的大小来判断某任务是不是需要执行。

```js
requestIdleCallback(function(deadline) {
  // deadline.timeRemaining() 获取浏览器的空余时间
})
```

​	页面是一帧一帧绘制出来的，当每秒绘制的帧数达到 60 时，页面是流畅的，小于这个值时， 用户会感觉到卡顿

​	1s 60帧，每一帧分到的时间是 1000/60 ≈ 16 ms，如果每一帧执行的时间小于16ms，就说明浏览器有空余时间



## 解决方案

1. 利用浏览器空闲时间执行任务，拒绝长时间占用主线程
2. 放弃递归只采用循环，因为循环可以被中断（只要将循环的条件保存下来，下次任务就可以从中断的地方开始执行了）
3. 任务拆分，将任务拆分成一个个的小任务（以前是整个树的比对是一个任务，现在每个节点是一个任务）

​	Fiber就是比对更新虚拟dom的算法，虚拟dom比对不会长期占用主线程了

