

伪元素：不会出现在dom树中，但是是真实存在的一个元素，可以去显示内容，可以设置样式

伪类：表示一种状态下的样式

# 页面布局

假设高度已知，请写出三栏布局，其中左栏右栏宽度各位300px，中间自适应

1.浮动	2.绝对定位 => 还没有达到及格分，三种及格 =>	3.flexbox	4.表格	5.网格布局



```js
// float
// 常见问题在于清除浮动，脱离文档流的，如果处理不好会带来很多问题；优点：兼容性好
.center {
  background: green;
}
.right {
  width: 300px;
  float: right;
  background: pink;
}
.left {
  width: 300px;
  float: left;
  background: pink;
}
// absolute
// 快，方便，缺点是已经脱离文档流了，意味着下面的所有子元素也要脱离文档流
.center{
  position: absolute;
  left: 300px;
  right: 300px;
  background: yellowgreen;
}
.left{
  position: absolute;
  width: 300px;
  background: pink;
  left: 0;
}
.right {
  position: absolute;
  background: pink;
  width: 300px;
  right: 0;
}
// flex
// 比较完美，兼容性差
.left{
  width: 300px;
}
.right{
  width: 300px;
}
.center{
  flex: 1;
}
section{
  display: flex;
}
// 表格布局
// 兼容性好，flex解决不了的时候可以尝试下表格布局，有时候三部分某个部分增高的时候其他部分也会跟着增高
section{
  display: table;
  width: 100%;
}
.left{
  width: 300px;
  display: table-cell;
}
.right{
  width: 300px;
  display: table-cell;
}
// 网格布局
// 新出的技术，栅格
section{
  display: grid;
  width: 100%;
  grid-template-rows: 100px; // 行高
  grid-template-columns: 300px auto 300px;// 三列
}

// 去掉高度已知，哪个不再适用了呢 => 浮动，绝对定位，网格
// 浮动为啥左右两边超出来了，这是因为浮动的基本原理，它的内容向左浮动的时候，被这个左侧的块挡住了，
// 所以文案在那边排的，当内容超出以后，发现左侧没有遮挡物，也就是没有float元素的话，会向左这块
// 这种情况让它不要影响到左右 => 创建BFC
```





```html
<section>
   <div class="right"></div>
   <div class="left"></div>
   <div class="center"></div>
</section>
// center必须放最后面(float时)
```







-------------------------------------

