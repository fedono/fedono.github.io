##  webpack 的流程是怎样的

## 参考

- [webpack编译流程](https://juejin.im/post/5d70b186f265da03e1689864)
- [Webpack源码解读：理清编译主流程](https://juejin.im/post/5dc01199f265da4d12067ebe)

##  webpack 的 Loader 和 plugin 的区别是怎样

### Loader

webpack 开箱即用只支持 JS 和 JSON 两种文件类型，通过 Loaders 去支持其它文 件类型并且把它们转化成有效的模块，并且可以添加到依赖图中。

本身是一个函数，接受源文件作为参数，返回转换的结果。

常⻅见的 Loaders 有哪些?

### Plugin

插件⽤用于 bundle 文件的优化，资源管理理和环境变量量注⼊入 作⽤用于整个构建过程

## 文件监听

⽂文件监听是在发现源码发⽣生变化时，⾃自动重新构建出新的输出⽂文件。

webpack 开启监听模式，有两种⽅方式: ·启动 webpack 命令时，带上 --watch 参数 ·在配置 webpack.config.js 中设置 watch: true

- 唯⼀一缺陷:每次需要⼿手动刷新浏览器器

### 文件监听的原理分析

轮询判断⽂文件的最后编辑时间是否变化 某个⽂文件发⽣生了了变化，并不不会⽴立刻告诉监听者，⽽而是先缓存起来，等 aggregateTimeout

```js
module.export = {

//默认 false，也就是不不开启
 watch: true, //只有开启监听模式时，watchOptions才有意义 wathcOptions: {

//默认为空，不监听的文件或者文件夹，支持正则匹配
 ignored: /node_modules/,
 //监听到变化发生后会等300ms再去执行，默认300ms
 aggregateTimeout: 300, //判断文件是否发生变化是通过不停询问系统指定文件有没有变化实现的，默认每秒问1000次 poll: 1000

} }
```

## 热更新

### webpack-dev-server

- WDS 不不刷新浏览器器

- WDS 不不输出⽂文件，⽽而是放在内存中
- 使⽤用 HotModuleReplacementPlugin插件

```diff
{
"name": "hello-webpack", "version": "1.0.0", "description": "Hello webpack", "main": "index.js",
"scripts": {
"build": "webpack ",
+ ”dev": "webpack-dev-server --open"
},
"keywords": [], "author": "", "license": "ISC"
}
```

### 使用 webpack-dev-middleware

WDM 将 webpack 输出的⽂文件传输给服务器器 适⽤用于灵活的定制场景

```js
const express = require('express');
const webpack = require('webpack');
const webpackDevMiddleware = require('webpack-dev- middleware');
const app = express();
const config = require('./webpack.config.js'); 
const compiler = webpack(config);
app.use(webpackDevMiddleware(compiler, { publicPath: config.output.publicPath}));
app.listen(3000, function () {
	console.log('Example app listening on port 3000!\n');
});
```

### 热更更新的原理分析

1. Webpack Compile: 将 JS 编译成 Bundle

2. HMR Server: 将热更更新的⽂文件输出给 HMR Rumtime

3. Bundle server: 提供⽂文件在浏览器器的访问

4. HMR Rumtime: 会被注⼊入到浏览器器， 更更新⽂文件的变化

5. bundle.js: 构建输出的⽂文件

![image-20200601141734312](../assets/imgs/webpack-all/hot-module.png)

## webpack 打包库和组件

webpack 除了了可以用来打包应用，也可以⽤用来打包 js 库

实现一个大整数加法库的打包

- 需要打包压缩版和非压缩版本
- 支持 AMD/CJS/ESM 模块引⼊

### 如何将库暴露出去

library: 指定库的全局变量量

libraryTarget: ⽀支持库引⼊的⽅式

```js
module.exports = { 
  mode: "production", 
  entry: {
  "large-number": "./src/index.js",
  "large-number.min": "./src/index.js" },
  output: {
    filename: "[name].js", 
    library: "largeNumber", 
    libraryExport: "default", 
    libraryTarget: "umd"
	}
};
```

### 如何指对 .min 压缩

通过 include 设置只压缩 min.js 结尾的⽂文件

```js
module.exports = {
  mode: "none", 
  entry: {
    "large-number": "./src/index.js",
    "large-number.min": "./src/index.js" 
  },
  output: {
    filename: "[name].js", library: "largeNumber", libraryTarget: "umd"
  }, 
  optimization: {
    minimize: true, minimizer: [
      new TerserPlugin({
        include: /\.min\.js$/, }),
    ]
  }
};
```

### 设置⼊入⼝口⽂文件

```js
// package.json 的 main 字段为 index.js
if (process.env.NODE_ENV === "production") { 
  module.exports = require("./dist/large-number.min.js");
} else {
	module.exports = require("./dist/large-number.js"); 
}
```



## 如何优化命令⾏的构建日志

使⽤用 friendly-errors-webpack-plugin

- success: 构建成功的⽇志提示
- warning: 构建警告的日志提示
- error: 构建报错的日志提示

stats 设置成 errors-only

```diff
module.exports = { entry: {
app: './src/app.js',
search: './src/search.js' },
output: {
	filename: '[name][chunkhash:8].js', path: __dirname + '/dist'
},
plugins: [
+ new FriendlyErrorsWebpackPlugin()
],
+ stats: 'errors-only' 
};
```

## webpack ssr 打包存在的问题

浏览器器的全局变量量 (Node.js 中没有 document, window) 

- 组件适配:将不不兼容的组件根据打包环境进⾏行行适配

- 请求适配:将 fetch 或者 ajax 发送请求的写法改成 isomorphic-fetch 或者 axios

样式问题 (Node.js ⽆无法解析 css) 

- 方案一:服务端打包通过 ignore-loader 忽略略掉 CSS 的解析

- 方案二:将 style-loader 替换成 isomorphic-style-loader

## 如何主动捕获并处理构建错误

compiler 在每次构建结束后会触发 done 这 个 hook

process.exit 主动处理理构建报错

```js
plugins: [ function() {
  this.hooks.done.tap('done', (stats) => { if (stats.compilation.errors &&
    stats.compilation.errors.length && process.argv.indexOf('- -watch') == -1)
      {
        console.log('build error');
        process.exit(1); 
      }
        }
     ) 
  }
]
```

> Node.js 中的 process.exit 规范
>
> - 0 表示成功完成，回调函数中，err 为 null
>
> - 非 0 表示执⾏失败，回调函数中，err 不不为 null，err.code 就是传给 exit 的数字

## webpack 如何做优化

- 缓存，提升二次构建速度
  
- 使用 cache-loader 来缓存
  
- 多进程/多实例，使用 `thread-loader` 多线程 `loader` 来加快打包过程

  - 每次 webpack 解析一个模块，thread- loader 会将它及它的依赖分配给 worker 线程中

- 缩小构建目标

  - 使用 `exclude` 

    目的:尽可能的少构建模块
    比如 babel-loader 不解析 node_modules

- 减少文件搜索范围

  - 优化 resolve.modules 配置(减少模块搜索层级)
  - 优化 resolve.mainFields 配置
  - 优化 resolve.extensions 配置

- 图片压缩

  - 配置 image-webpack-loader

    识别 User Agent，下发不同的 Polyfill

- 动态 Polyfill

  - Polyfill Service原理

- 利用 SplitChunksPlugin 进⾏公共脚本分离

- tree shaking

- scope hoisting

- Dead code elimination

- 代码压缩

  - TerserPlugin 
    - 同时开启多线程 	`parallel: 4` 压缩

- 使用高版本

  - V8 带来的优化(for of 替代 forEach、Map 和 Set 替代 Object、includes 替代 indexOf) 
  - 默认使用更快的 md4 hash 算法
  -  webpacks AST 可以直接从 loader 传递给 AST，减少解析时间 
  - 使用字符串方法替代正则表达式

- 分包:设置 Externals

  将 react、react-dom 基础包通过 cdn 引入，不打入 bundle 中

  ```js
   
  ```

- 进一步分包:预编译资源模块

  **思路**:将 react、react-dom、redux、react-redux 基础包和业务基础包打包成一个文件

  **方法**:使用 DLLPlugin 进行分包，DllReferencePlugin 对 manifest.json 引用

## 参考

- [react-webpack](https://github.com/JinJieTan/react-webpack) 

  React移动端项目，手动配置的脚手架，完美性能优化的极致追求