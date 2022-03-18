developer.mozilla.org/en/differential_inheritance_in_javascript

```js
let paser_url = /^(?:([A-Za-z]+):)?(\/{0,3})([0-9.\-A-Za-z]+)(?::(\d+))?(?:\/([^?#]*))?(?:\?([^#]*))?(?:#(.*))?$/;
var names = ['url', 'schema', 'slash', 'host', 'port', 'path', 'query', 'hash'];


let tags = /[^<>]+|<(\/?)([A-Za-z]+)([^<>]*)>/g;
let text = '<html><body bgcolor=linen><p>' + 'This is <b>bold<\/b>!<\/body><\/html>';
while ((a = tags.exec(text))) {
  for (i = 0;i < a.length; i+=1) {
    console.log('[' + i + '] ' + a[i])
  }
}
a = text.match(tags); // 就能把所有的内容都提取出来了
```

- JavaScript 中的函数是根据词法来划分作用域的，而不是动态划分作用域的，具体内容参见《JavaScrip权威指南》中 8.8.1 词法作用域 

- 排序的稳定性是指排序后数组中的相等值的相对位置没有发生改变，而不稳定性排序则会改变相等值的相对位置。

- JavaScript 的 new 运算符创建一个继承于其运算数的原型的新对象，然后调用该运算数，把新创建的对象绑定给 this，这给运算数（它应该是一个构造器函数）一个机会在返回给请求者前自定义新创建的对象。

  如果你忘记了使用此 new 运算数，你得到的就是一个普通的函数调用，并且 this被绑定到全局对象，而不是新创建的对象。这意味着当你的函数尝试去初始化新成员属性时它将会污染全局变量。这是一件非常糟糕的事情。而且既没有编译时警告，也没有运行时警告。

- 在ECMAScript 的规范中，换行符被称为"行结束符(Line Terminators)"，它会影响自动插入分号机制的处理过程。

- JSON 特别易于用在 web 应用中，因为JSON 就是 JavaScript。使用 eval 函数可以把一段JSON文本转化成一个有用的数据结构

  ```js
  // 用圆括号吧JSON文本括起来是一种避免JavaScript语法歧义的变通方案
  var myData = eval('(' + myJSONText + ')');
  ```

  在JavaScript的语法中，表达式语句不允许以左花括号"{" 开始，因为那会与块语句（Block Statement）产生混淆。详细说明请参见 ECMAScript 规范相关章节-- "12.4 Expression Statement"。在使用 eval 解析JSON 文本时，为了解决此问题，可以将JSON文本套上一对圆括号。圆括号在此处作为表达式的分组运算符，能对包围在其中的表达式进行求值。它能正确的识别对象字面量。详细说明参见 ECMAScript 规范的相关章节--"11.1.6 The Grouping Operator"。

  感觉这个原因也是为什么`{} + []` 会等于0的原因，因为第一个`{}`会被解释为`Block Statement` 

- ![image-20201110095127401](/Users/zengdeping/Library/Application Support/typora-user-images/image-20201110095127401.png)