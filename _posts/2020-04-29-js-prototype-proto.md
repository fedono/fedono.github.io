---
layout: post 
title: "prototype 与 proto" 
author: "fedono"
---

`instanceof`运算符的一个用处，是判断值的类型。

```js
var x = [1, 2, 3];
var y = {};
x instanceof Array // true
y instanceof Object // true
```

上面代码中，`instanceof`运算符判断，变量`x`是数组，变量`y`是对象。

注意，`instanceof`运算符只能用于对象，不适用原始类型的值。

```js
var s = 'hello';
s instanceof String // false
```

上面代码中，字符串不是`String`对象的实例（因为字符串不是对象），所以返回`false`。

此外，对于`undefined`和`null`，`instanceof`运算符总是返回`false`。

```js
undefined instanceof Object // false
null instanceof Object // false
```

利用`instanceof`运算符，还可以巧妙地解决，调用构造函数时，忘了加`new`命令的问题。

```js
function Fubar (foo, bar) {
  if (this instanceof Fubar) {
    this._foo = foo;
    this._bar = bar;
  } else {
    return new Fubar(foo, bar);
  }
}
```

上面代码使用`instanceof`运算符，在函数体内部判断`this`关键字是否为构造函数`Fubar`的实例。如果不是，就表明忘了加`new`命令。



参考

- [详解prototype与__proto__](http://louiszhai.github.io/2015/12/17/prototype/) 

  > 感觉看这篇文章就足够了

  可以使用 `Object.prototype.isPrototypeOf()` 来判断当前对象是否为另一个对象的原型 

- [对象的继承](https://wangdoc.com/javascript/oop/prototype.html) 

- [](https://wangdoc.com/javascript/oop/object.html)