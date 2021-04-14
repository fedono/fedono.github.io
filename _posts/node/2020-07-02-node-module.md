

## 引入模块的三个步骤

Node 中引入模块，需要经历三个步骤

- 路径分析
- 文件定位
- 编译执行



奇怪，路径分析和文件定位难道不是一码事吗？



## 为什么可以不通过`require` 引入内部包就可以直接运行

> 核心模块部分在 Node 源代码的编译过程中，编译进了二进制执行文件。在 Node进程启动时，部分核心模块就被直接加载进内存中，所以这部分核心模块引入时，文件定位和编译执行这两步可以省略掉，并且在路径分支中优先判断，所以他的加载速度是最快的

文件模块则是在运行时动态加载，需要完整的路径分析、文件定位、编译执行过程，速度比核心模块慢

### JavaScript 模块的编译

每个模块文件中存在 `require`, `exports`, `module` 这三个变量，每个模块中还有`__filename` 和 `__dirname_ `，他们从何而来？

在编译的过程中，Node 对获取的JavaScript文件内容进行了头尾包装，在头部添加了 `(function(exports, require, module, __filename, __dirname) {\n` 在尾部添加了 `\n})` 。

一个正常的JavaScript 文件会被包装如下

```js
(function(exports, require, module, __filename, __dirname) {
  var math = require('math');
  exports.area = function(radius) {
    return Math.PI * radius * radius;
  };
});
```

 这样每个模块文件之间都进行了作用域隔离。包装之后的代码会通过vm 原生模块的 `runInThisContext()` 发放执行（类似eval，只是具有明确上下文，不污染全局），返回一个具体的function对象。最后，将当前模块对象的 exports 属性，require() 方法，module(模块对象自身)，以及在文件定位中得到的完整文件路径和文件目录作为参数传递给这个 funtion() 执行。



为何存在 exports 的情况下，还存在 module.exports。理想情况下，只要赋值给 exports 即可。

```js
exports = function() {
  // my code
};
```

但是通常会得到一个失败的结果，起原因在于，exports 对象是通过形参的方式传入的，直接赋值形参会改变形参的引用，但不能改变作用域外的值。测试代码如下

```js
var change = function(a) {
  a = 100;
  console.log(a); // 100
};

var a = 10;
change(a);
console.log(a); // 10
```

如果要达到 require 引入一个类的效果，请赋值给 module.exports 对象。这个迂回方案不改变形参的引用。