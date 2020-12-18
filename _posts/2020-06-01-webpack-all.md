##  webpack 的流程是怎样的

## 参考

- [webpack编译流程](https://juejin.im/post/5d70b186f265da03e1689864)
- [Webpack源码解读：理清编译主流程](https://juejin.im/post/5dc01199f265da4d12067ebe)

##  webpack 的 Loader 和 plugin 的区别是怎样

### Loader

webpack 开箱即用只支持 JS 和 JSON 两种文件类型，通过 Loaders 去支持其它文 件类型并且把它们转化成有效的模块，并且可以添加到依赖图中。

本身是一个函数，接受源文件作为参数，返回转换的结果。

常⻅见的 Loaders 有哪些?

如何写一个 loader 

### Plugin

插件用于 bundle 文件的优化，资源管理和环境变量注入 作用于整个构建过程

## 文件监听

文件监听是在发现源码发生变化时，⾃动重新构建出新的输出文件。

webpack 开启监听模式，有两种⽅式: ·启动 webpack 命令时，带上 --watch 参数 ·在配置 webpack.config.js 中设置 watch: true

- 唯⼀一缺陷:每次需要手动刷新浏览器

### 文件监听的原理分析

轮询判断文件的最后编辑时间是否变化 某个文件发生了变化，并不会立刻告诉监听者，⽽是先缓存起来，等 aggregateTimeout

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

- WDS 不刷新浏览器器

- WDS 不输出文件，而是放在内存中
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

WDM 将 webpack 输出的⽂件传输给服务器 适用于灵活的定制场景

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

### 热更新的原理分析

1. Webpack Compile: 将 JS 编译成 Bundle

2. HMR Server: 将热更新的文件输出给 HMR Rumtime

3. Bundle server: 提供⽂件在浏览器器的访问

4. HMR Rumtime: 会被注入到浏览器器， 更新⽂文件的变化

5. bundle.js: 构建输出的⽂文件

![image-20200601141734312](../assets/imgs/webpack-all/hot-module.png)

## webpack 打包库和组件

> 这个通常会问打包静态文件和打包npm包的区别



webpack 除了了可以用来打包应用，也可以⽤用来打包 js 库

实现一个大整数加法库的打包

- 需要打包压缩版和非压缩版本
- 支持 AMD/CJS/ESM 模块引⼊

### 如何将库暴露出去

library: 指定库的全局变量量

libraryTarget: ⽀持库引⼊的⽅式

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
    minimize: true, 
    minimizer: [
      new TerserPlugin({
        include: /\.min\.js$/, }),
    ]
  }
};
```

### 设置⼊口文件

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

浏览器的全局变量量 (Node.js 中没有 document, window) 

- 组件适配:将不兼容的组件根据打包环境进行行适配

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



## 整个打包的流程

1. 调用`webpack`函数接收`config`配置信息，并初始化`compiler`，在此期间会`apply`所有 webpack 内置的插件;

2. 调用`compiler.run`进入模块编译阶段；
3. 每一次新的编译都会实例化一个`compilation`对象，记录本次编译的基本信息；

4.  进入`make`阶段，即触发`compilation.hooks.make`钩子，从`entry`为入口： a. 调用合适的`loader`对模块源码预处理，转换为标准的JS模块； b. 调用第三方插件`acorn`对标准JS模块进行分析，收集模块依赖项。同时也会继续递归每个依赖项，收集依赖项的依赖项信息，不断递归下去；最终会得到一颗依赖树；

5. 最后调用`compilation.seal` render 模块，整合各个依赖项，最后输出一个或多个chunk；




##内部原理是什么

需要涉及到 tapable ，然后如何使用 loader 和 如何使用 plugin 需要说清楚

`tapable` 是一个流程管理的库，涉及到同步和异步

## 是否写过 loader 

这个肯定要说有，但是写了什么，如何写一个 `loader` 

参考 [inline-html-loader](https://github.com/cpselvis/inline-html-loader)

```js
const fs = require('fs');
const path = require('path');

const getContent = (matched, reg, resourcePath) => {
    const result = matched.match(reg);
    const relativePath = result && result[1];
    const absolutePath = path.join(path.dirname(resourcePath), relativePath);
    return fs.readFileSync(absolutePath, 'utf-8');
};

module.exports = function(content) {
    const htmlReg = /<link.*?href=".*?\__inline">/gmi;
    const jsReg = /<script.*?src=".*?\__inline".*?>.*?<\/script>/gmi;

    content = content.replace(jsReg, matched => {
        const jsContent = getContent(matched, /src="(.*)\?__inline/, this.resourcePath);
        return `<script type="text/javascript">${jsContent}</script>`;
    }).replace(htmlReg, matched => {
        const htmlContent = getContent(matched, /href="(.*)\?__inline/, this.resourcePath);
        return htmlContent;
    });

    return `module.exports = ${JSON.stringify(content)}`;
}
```

看上面的代码，每一个`loader` 都会有一个 `content` 参数传入进来，这时候在使用`loader` 的匹配规则 `test` 时，匹配到相应的文件，这时候通过正则匹配到`content` 中的内容，然后通过每个 `loader` 都会获取到的全局变量`this.resourcePath` 配合正则获取到的 `path` 这时候就可以通过 `fs.readFileSync` 来获取内容了，这样就可以实现把通过 `link` 和 `script` 引入的文件的内容放到当前文件中，就实现了 `inline` 的效果。

如何使用上面这个`loader` 呢

```js
{
  test: /.html$/,
  use: 'inline-html-loader'
}
```

## 相关细节问题

- 如何将多个CSS 打包

  > 以下是我的答案，不确定是否正确

  使用 extract-text-webpack-plugin 来获取到所有的 css ，然后使用`css-loader` 打包文件，如果只是需要两个文件打包，那加上`include` 好了，只命中需要打包的文件。

  ```js
  const ExtractTextPlugin = require("extract-text-webpack-plugin");
  
  module.exports = {
    module: {
      rules: [
        {
          test: /\.css$/,
          use: ExtractTextPlugin.extract({
            fallbackLoader: "style-loader",
            loader: "css-loader"
          })
        }
      ]
    },
    plugins: [
      new ExtractTextPlugin("styles.css"),
    ]
  }
  ```

- output 中的 path 和 publicPath 的区别是什么



## 参考

- [react-webpack](https://github.com/JinJieTan/react-webpack) 

  React移动端项目，手动配置的脚手架，完美性能优化的极致追求