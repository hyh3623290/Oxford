# entry

​	可以接收字符串，数组，对象

```js
entry: "./src/index.js"

entry: ["./src/index.js", "./src/other.js"]

// 多入口
entry: {
  main: "./src/index.js",
  other: "./src/index.js",
}
```

​	多入口对应输出文件 `[name].js`，会输出两个文件( bundle )。一个输入文件对应一个chunk，对应一个bundle

# output

占位符 - hash，chunkhash，name，id

```js
output: {
  path: path.resolve(__dirname, "./dist")
  filename: "[name]-[hash:6].js"
}
```

hash的作用就是缓存

# loader

​	模块解析，webpack默认只认识json和js ，其他的就需要loader呢













# ---

⚠️ mode会把process.env设置成对应的