# React Router

​	我们知道不同路由一般会跳转不同的页面，可以认为一个 URL 对应一个 UI。对于多页面应用而言，一个 URL 对应的就是一个 HTML 页面，而对于单页面应用，一个 URL 对应的其实是一个组件的展示。我们可以通过 URL 来控制 UI 或者 HTML 的展示，这种方式就是路由的方式

## BrowserRouter

​	它使用 HTML5 提供的 history API（pushState、replaceState 和 popstate 事件）来保持 UI 和 URL 的同步。

​	basename 所有路由的基准 URL。basename 的正确格式是前面有一个前导斜杠，但不能有尾部斜杠

```jsx
<BrowserRouter basename="/calendar">
  <Link to="/today" />
</BrowserRouter>
```

## HashRouter

​	使用 URL 的 hash 部分（即 window.location.hash）来保持 UI 和 URL 的同步。



## Route

### 语法

​	当 URL 和 Route 匹配时，Route 会创建一个 match 对象作为 props 中的一个属性传递给被渲染的组件。这个对象包含以下 4 个属性：

1. **params**: Route 的 path 可以包含参数，params 就是用于从匹配的 URL 中解析出 path 中的参数；
2. **isExact**：同 Route 的 exact；
3. **path**：同 Route 的 path；
4. **url：URL** 匹配的方式。

### Route 渲染组件的方式

​	route如果不写path，则一直匹配。Route渲染优先级：children>component>render。 三者能接收到同样的[route props]，包括match, location and history，但是当不匹配的时候，children的match为 null。

​	有时候，不管location是否匹配，你都需要渲染⼀些内容，这时候你可以⽤children。其它⼯作⽅法与 render完全⼀样。当你⽤render的时候，你调⽤的只是个函数。component传一个组件，只在当location匹配的时候渲染

​	**component**的话其实也可以用函数的方式写，但是这样不好`<Route component={()=><child />}>`，这样的话组件更新的时候会不停的挂载卸载。因为渲染component的时候会调用React.createElement，如果使用下面这种匿名函数的形式，每次都会生成一个新的匿名的函数，导致生成的组件的type总是不相同，这个时候会产生重复的卸载和挂载。为什么type值不同，因为它是匿名函数，两个匿名函数怎么能相同呢

​	当你⽤component的时候，Router会用你指定的组件和 React.createElement创建一个新的[React element]。这意 味着当你提供的是一个内联函数的时候，每次render都会创建一个新的组件。这会导致不再更新已经现有组件，而是直接卸载然后再去挂载一个新的组件。因此，当⽤用到内联函数的内联渲染时，请使⽤用render或者children。



## Switch & exact

​	Switch 通常被用来包裹 Route，用于渲染与路径匹配的第一个子 `<Route>` 或 `<Redirect>`，它里面不能放其他元素。

## Link & NavLink

​	NavLink 是特殊的 Link，可以为与当前 URL 匹配的 Link 元素增加样式属性。唉，其实就是加一些特殊样式的Link

## 嵌套路由

​	嵌套路由是指在 Route 渲染的组件内部定义新的 Route

# 虚拟DOM

​    操作真实 DOM 会引起重排和重绘，会影响到性能，因此要尽量减少 DOM 操作。其次DOM的数据结构也非常复杂。对于 DOM 的任何操作都映射为内存中对象的操作，在必要时才重新渲染页面。

⚠️15 创建虚拟 DOM：createElement - 虚拟 DOM 转换为真实 DOM ReactDOM.render - Fiber对象 - don't understand

# diff算法

​	给定任意两棵树，找到最少的转换步骤，是一个复杂且值得研究的问题。

React 结合 Web 界面的特点做出了三个简单的假设，使得 Diff 算法复杂度直接降低到 O(n)。

1. Web UI 中 DOM 节点跨层级的移动操作特别少，可以忽略不计；
2. 两个相同组件产生类似的 DOM 结构，不同的组件产生不同的 DOM 结构；
3. 对于同一层次的一组子节点，它们可以通过唯一的 id 进行区分。

基于以上三个假设，React 分别对 tree diff、component diff 以及 element diff 进行算法优化。

## tree diff

​	由于跨节点层级的移动操作特别少到可以 忽略不计，针对这一点，React 通过对两个树的同一层的节点进行比较，当发现节点已经不存在时，则该节点及其子节点会被完全删除掉，不会用于进一步的比较。这样只需要对树进行一次遍历，便能完成整个 DOM 树的比较。所以，官方建议不要进行DOM节点的跨层级操作，因为它会整个移除并重新插入。在实现自己的组件时，保持稳定的 DOM 结构会有助于性能的提升。例如，我们有时可以通过 CSS 隐藏或显示某些节点，而不是真的移除或添加 DOM 节点。

## component diff

React 是基于组件构建应用的，对于组件间的比较所采取的策略也是简洁高效。

1. 如果是同一类型的组件，按照原策略继续比较虚拟 DOM Tree ；
2. 如果不是，则将该组件判断为 dirty component，从而替换整个组件下的所有子节点；
3. 对于同一类型的组件，有可能其虚拟 DOM 没有任何变化，如果能够确切的知道这点那可以节省大量的 diff 运算时间，因此 React 允许用户通过 shouldComponentUpdate() 来判断该组件是否需要进行 diff 

## element diff

​	当节点处于同一层级（比如`ul`下面一组`li`）时，React Diff 提供了三种节点操作，分别为：INSERT_MARKUP（插入）、MOVE_EXISTING（移动）和 REMOVE_NODE（删除）

1. **INSERT_MARKUP**：新的 component 类型不在老集合里， 即是全新的节点，需要对新节点执行插入操作；
2. **MOVE_EXISTING**：在老集合有新 component 类型，且 element 是可更新的类型，generateComponentChildren 已调用 receiveComponent，这种情况下 prevChild=nextChild，就需要做移动操作，可以复用以前的 DOM 节点；
3. **REMOVE_NODE**：老 component 类型，在新集合里也有，但对应的 element 不同则不能直接复用和更新，需要执行删除操作，或者老 component 不在新集合里的，也需要执行删除操作

​    有种情况：老集合中包含节点：A、B、C、D，更新后的新集合中包含节点：B、A、D、C，两个集合的顺序发生了一些 改变。此时新集合和老集进行 diff 差异化对比时，发现 B != A，则创建并插入 B 至新集合，删除老集合 A；以此类推，创建并插入 A、D 和 C，删除 B、C 和 D。

​    针对这一现象，React 提出优化策略：允许开发者对同一层级的同组子节点，添加唯一 key 进行区分，虽然只是小小的改动，性能上却发生了翻天覆地的变化！

⚠️16 - 那么，如此高效的 diff 到底是如何运作的呢？让我们通过源码进行详细分析。

# Fiber

​	JS单线程 - 如果前面有一个任务长期霸占 CPU ，那后面的什么事情都干不了 - 浏览器会呈现卡死的状态 - 对于我们前端框架而言，解决这种问题通常有三个方向：

1. 优化每个任务，提高它的运行速度，挤压 CPU 运算量；
2. 快速响应用户，让用户觉得快，不阻塞用户的交互；
3. 尝试 Worker 多线程。

​    Vue 选择了第一种，而 React 选择了第二种

​	React V15 在渲染时，会递归比对 VirtualDOM 树，找出需要变动的节点，然后同步更新它们， 一气呵成。这个过程 React 称为 Reconcilation 。在 Reconcilation 期间， React 会占据浏览器资源，会导致用户触发的事件得不到响应，并且会导致掉帧，导致用户感觉到卡顿。

​	为了给用户制造一种应用很快的“假象”，我们不能让一个任务长期霸占着资源。 你可以将浏览器的渲染、布局、绘制、资源加载(例如 HTML 解析)、事件响应、脚本执行视作操作系统的“进程”，我们需要通过某些调度策略合理地分配 CPU 资源，从而提高浏览器的用户响应速率, 同时兼顾任务执行效率。

​	所以 React 通过Fiber 架构，让自己的 Reconcilation 过程变成可被中断。“适时”地让出 CPU 执行权，除了可以让浏览器及时地响应用户的交互，还有其他好处:

1. 与其一次性操作大量 DOM 节点相比, 分批延时对DOM进行操作，可以得到更好的用户体验；
2. 给浏览器一点喘息的机会，他会对代码进行编译优化（JIT）及进行热代码优化，或者对 reflow 进行修正。

​    渲染的过程可以被中断，可以将控制权交回浏览器，让位给高优先级的任务，浏览器空闲后再恢复渲染。

​	通常屏幕的帧率为60Hz，也就是每秒60张画面。所以每一张画面的时间为1000/60 = 16ms，所以我们的代码执行实现少于16ms，就能看到流畅的画面。

⚠️ 17

# 服务端渲染

​	它可以在请求时，在服务端直接生成一个完整结构的 HTML 文件，返回给浏览器展示。这样减少了首次加载白屏的时间，同时解决了 SEO 不友好的缺点。

```js
ReactDOMServer.renderToString(element)
ReactDOMServer.renderToStaticMarkup(element)
```

⚠️18

# React路由工作原理

⚠️19

# umi

```js
yarn global add umi
```

⚠️20

# dva

⚠️21

















# - -

⚠️**react性能**

- 官方建议不要进行DOM节点的跨层级操作，因为它会整个移除并重新插入。在实现自己的组件时，保持稳定的 DOM 结构会有助于性能的提升。例如，我们有时可以通过 CSS 隐藏或显示某些节点，而不是真的移除或添加 DOM 节点。
- 为了提高性能，请在循环渲染的组件中添加 key 值





【参考文献】

- React深度剖析