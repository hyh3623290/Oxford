# 选择器

**元素选择符，通用选择符，类选择符，ID选择符**

**群组选择符**

​	就是一组，一起的意思

```less
h1, p {
  color: red; // h1和p都red
}
```

```less
p.warning.help { // 类名为warning且help且是p元素
  color: red;
}
```

**属性选择符**

```less
h1[class] { // 具有class属性的h1标签
  color: red;
}
```

**后代选择符**

```less
h1 p { // h1的后代p
  color: red;
}
```

**子元素选择符**

```less
h1 > p { // h1的子元素p
	color: red;
}
```

**紧邻兄弟选择符**

```less
h1 + p { // h1紧挨着的p元素
  color: red;
}
```

**兄弟选择符**

```less
h1 ~ p { // 跟h1在一级的p

}
```

**伪类选择符**

```less
a:link:hover {
  color: red;
}
```

​	伪类始终指代所依附的元素，比如我想选择我的第一个孩子

```less
#hyh > :first-child { }

// 大部分人会写成这样
#hyh:first-child
// 这个的意思是#hyh，且#hyh是父级的第一个子集，它选中的#hyh
```

```less
// 选择空元素
p:empty {
  display: none; // 禁止显示空段落
}

// 选择唯一的子代
img:only-child // 只有标签可以，试了一下类名不香
```













-----------------------

⚠️注：有些元素在有些旧的浏览器上无法识别，如ie8不识别`<main>`元素，我们可以在dom上创建元素，让浏览器知道元素的存在，`document.createElement('main')`