# 选择器

## 通用选择器



## ID选择器



## 类选择器



## 元素选择器



## 后代选择器

```scss
h1 p { // h1的后代p
  color: red;
}
```



## 儿子选择器

```scss
h1 > p { // h1的子元素p
	color: red;
}
```



## 紧邻兄弟选择器

```scss
h1 + p { // h1紧挨着的p元素
  color: red;
}
```



## 兄弟选择器

```scss
h1 ~ p { // 跟h1在一级的p

}
```



## 群组选择器

```scss
h1, p {
  color: red; // h1和p都red
}
```



## 属性选择器

```scss
h1[class] { // 具有class属性的h1标签
  color: red;
}
```



## 伪类选择器

```scss
a:link:hover {
  color: red;
}
```

​	伪类始终指代所依附的元素，比如我想选择我的第一个孩子

```scss
#hyh > :first-child { }

// 大部分人会写成这样
#hyh:first-child
// 这个的意思是#hyh，且#hyh是父级的第一个子集，它选中的#hyh
```







# ---